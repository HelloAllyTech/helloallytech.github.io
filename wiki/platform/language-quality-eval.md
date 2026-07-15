---
title: Language-Quality Evaluation & RCA
tags: [platform, evaluation, language-quality, voice, llm-judge, rca]
summary: How we evaluate and fix the language quality of voice roleplay agents — categorized error annotation by LLM judge, two objective speech metrics, and single-variable root-cause analysis.
---

# Language-Quality Evaluation & RCA

This page describes the framework we use to evaluate and *fix* the **language quality** of Ally's voice roleplay agents (the AI plays a distressed client; the human trainee is the counselor). It covers what industry practice recommends, the layered framework, the measurement toolkit, and the controlled-experiment methodology for root-cause analysis.

The detailed engineering specs live with the code — see [ally-ai](../repos/ally-ai.md) (the offline eval/judge service) and [ally-be](../repos/ally-be.md) (orchestration, storage, dashboards). This page is the conceptual overview.

---

## 1. The problem: our current signals can't drive fixes

- **Per-session observations are anecdotes** — they can't distinguish a systemic problem from a one-off, and carry no denominator.
- **Scalar scores (1–5 / 0–100) and raw issue counts** blend comprehension, generation, and speech into one number that can't localize a cause. Industry experience calls this the **metric mirage**: healthy-looking dashboards coexisting with a failing product.

The standard alternative (from machine-translation and dialogue-evaluation practice) is **categorized error annotation** — every issue gets a *dimension, error category, and severity*, reported as a **weighted error rate per 100 turns** (severity weights minor = 1, major = 5, critical = 10). Structured, denominated, and comparable across runs. This is the MQM error-typology idea, executed entirely by an **LLM judge over session transcripts — no human text annotation, no hand-labeled calibration sets.**

## 2. The framework: four layers, gated bottom-up

Language capability decomposes into layers that map onto pipeline components — which is what makes failures *attributable*:

| Layer | Question | Dimensions | Measured by | Implicates |
|---|---|---|---|---|
| **Comprehension** | Did it understand the trainee? | understanding | LLM judge (conditioned on STT — §5) | LLM |
| **Content** | Is what it said correct, well-formed, consistent? | adequacy, fluency, coherence | LLM judge + **script fidelity** (objective) | LLM + prompt |
| **Appropriateness** | Is *how* it said it right? | register, dialect-lexicon, colloquialness, persona/social, code-switching | LLM judge | **Mostly prompt/persona design**; LLM as fallback |
| **Speech realization** | Does it *sound* right? | intelligibility, naturalness, prosody, affect, accent | **Round-trip WER** (objective) + manual listening | TTS / voice |

**Gate rule:** intelligibility (round-trip WER) gates the layers above it in audio — you can't judge the naturalness of unintelligible speech. A failed gate means those judgments are *unmeasured*, not "fine," and the dashboard shows them as masked.

**Priority tiers** (for the distressed-persona training use case): Tier 1 = intelligibility; Tier 2 = adequacy + register + dialect-lexicon + colloquialness (these break the training purpose); Tier 3 = naturalness / prosody / affect / accent; Tier 4 = fluency / coherence floor checks.

## 3. The measurement toolkit

**Objective, automatable (these carry their own attribution):**

- **Round-trip WER**: `WER(T, ASR(TTS(T)))` where `T` is the LLM's own output text — re-synthesize with the session's TTS voice, transcribe with a strong ASR, and compare. Isolates TTS pronunciation; no human transcript needed. Thresholds: ≤ 20 % good / 20–30 % warn / > 30 % critical. Use **CER** for scripts with unreliable word segmentation. *Per-language precondition: a reliable ASR must be declared for the language.*
- **Script fidelity**: percentage of turns rendered cleanly in the target script (a Unicode-block check) — catches encoding / glyph failures. Pure code, no LLM.

**LLM-as-judge (all text layers):** one call per session over the whole transcript → per-turn error annotations against a **frozen typology**:

| Dimension | Error categories |
|---|---|
| understanding | misinterpreted_intent, ignored_context |
| adequacy | off_topic, hallucination, omission |
| fluency | grammar, script_error, disfluency, truncation |
| coherence | contradiction, non_sequitur |
| register | too_formal_diglossia, too_casual |
| dialect_lexicon | wrong_regional_variety |
| colloquialness | literal_translation_stilt |
| persona_social | too_blunt, persona_break |
| codeswitch | foreign_token_leak, unnatural_switch |

Each annotation carries a severity, a short evidence quote, and an **isolation basis** — including `persona_specified` vs `persona_unspecified`, which encodes *"did the prompt even ask for it?"* directly into every error (see §6). Judge reliability comes from **pinning** (judge model + rubric version stamped on every row; comparisons valid only within one version), temperature-0 structured output, reading **deltas** rather than absolute levels, and spot-checks via the error-log drill-down.

Critical rubric **carve-outs** (learned from earlier conversation-drift work — these prevent false positives): in-character distress ≠ incoherence; natural code-switching with the configured partner language is *correct*; configured fillers ≠ disfluency; intentional withholding of secrets (disclosure design) ≠ omission; counselor-led topic changes ≠ off-topic.

**Audio beyond intelligibility:** no formal listening-test tooling for now. Human listening happens via the session-logs audio player; the dashboard marks naturalness / prosody / affect / accent as "manual listening only."

## 4. This is not a greenfield build

The framework generalizes an existing **conversation-drift evaluation system** rather than rebuilding from scratch — that system already provides roughly 60 % of the architecture:

- **A working LLM-judge seam**: a stateless judge in the AI service (structured output, versioned rubric in prompt management) → orchestration and persistence in the backend → a Carbon dashboard tab in the web admin. The language judge is a sibling of this, on the same seam.
- **Experiment-config capture on every session**: prompt versions + scenario version on the session; LLM provider / model + generation parameters on per-turn metrics. "Did config X reduce errors vs config Y?" is already answerable by a grouped query.
- **Versioning**: scenario configs are fully versioned (diffable against their parent — this is how we auto-derive *which element changed* between experiments); prompt templates are versioned per code; roleplay specs are versioned separately.
- **Session-logs dashboard**: per-session drill-down with transcript, audio playback, models used, latency, and cost — the natural home for per-turn error badges.
- **Session audio in object storage** plus batch STT with multi-provider fallback in the AI service — the ingredients for round-trip WER.
- **Test harnesses**: a rehearsal harness (simulated skilled / poor / adversarial trainees) and a voice-to-voice tester become the *execution engine* for single-variable experiments, without waiting for live traffic.

**What's genuinely new:** the language-quality error typology + judge, the two objective metrics, per-language eval config, pinned-reference / delta readouts with changed-element derivation, the Language dashboard tab, and per-turn annotations in session logs.

## 5. Corrections established against the codebase

So the framework doesn't fight reality:

1. **"Backstory" and "role instructions" are one field** — the scenario's persona prompt is delivered to the agent verbatim as role instructions. They cannot be varied independently; the inventory treats them as a single element.
2. **"Prompt metadata" spans two surfaces** — scenario config (persona, per-language style fields) and prompt templates (the system-prompt skeletons). Both are versioned and captured per session; attribution must name `surface:element`.
3. **STT conditioning** — the judge flags garbled counselor input per turn, and those turns are excluded from the understanding / adequacy denominators, so STT failures aren't mis-billed to the LLM. (STT attribution itself stays with the conversation-drift system.)
4. **Per-turn TTS audio isn't stored** (one mixed room recording) — round-trip WER re-synthesizes from the LLM's own text with the session's voice config, which is exactly what the metric's definition specifies anyway.
5. **A versioning gap for per-language opening statements** was found and fixed so changes there become attributable.
6. **Existing 0–100 report metrics stay** as trainee-facing product features but never appear on the eval dashboard — no scalar quality scores anywhere in this system.

## 6. The fixing methodology: RCA by controlled experiment

1. **One variable at a time.** A "prompt" is a bundle of independently versionable elements; "we rewrote the prompt" is not one variable. The per-language levers are concrete scenario fields: **language characteristics** (a free-text style directive per language), **linguistic style samples** (few-shot exemplars per language), **allowed filler words**, and **per-language opening statements** — plus the persona prompt, the main-prompt template, and the model / voice axes.
2. **Prompt-before-model.** For register / dialect / colloquialness failures the decision rule is mechanical: *is the colloquial directive configured for this language? are style exemplars present?* If either is missing → populate it (a cheap metadata fix, one variable). If both are present and the error persists → a model-capability limit → escalate to the LLM axis. The judge's `persona_specified` / `persona_unspecified` label pre-sorts every error onto one side of this rule.
3. **Read deltas vs a pinned reference**, with the judge version held constant. Slice per language and dialect — never a global score.
4. **Isolation check**: a one-variable change should move only its layer; cross-layer movement is a leak to investigate.
5. **Failures → test cases** → re-run through the rehearsal / voice-to-voice harnesses before deployment.

## 7. Applying it per language (worked example: Kannada)

The framework is per-language by design. Kannada illustrates why the appropriateness layer matters:

- **Diglossia applies** — literary vs spoken Kannada differ sharply; "textbook-ish" output is the *expected default failure* of an LLM prompted without an explicit spoken-register directive and exemplars. The first experiment is mechanical: are the Kannada language-characteristics + colloquial style samples configured on the scenario? If not → populate them (one variable) and re-measure the `register` error rate.
- **Regional variety must be chosen explicitly** — it affects lexicon (text, judge-checkable) and accent (audio, TTS-voice-dependent); different failure modes, different fixes.
- **Kannada–English code-mixing** is normal spoken behavior; the judge treats natural mixing as correct and flags only leakage or unnatural switches.
- **Round-trip WER feasibility** depends on a reliable Kannada ASR, declared per language in config with its confidence tier.

## 8. What success looks like

- A failure narrows to a layer / component + error category well enough to design a **single-variable experiment** — and the experiment confirms it.
- One-variable changes move only their layer; leaks get surfaced.
- Tier-1 / Tier-2 weighted error rates fall across successive experiments, per language and variety.
- The team reads improvement / regression / staleness off the dashboard without re-reading transcripts.
