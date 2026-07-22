---
title: Activity Log
tags: [operations, log]
summary: A chronological log tracking all wiki updates and modifications.
---

# Activity Log

This is an append-only log of modifications, updates, and indexing runs performed on the wiki. All logs use the parseable prefix format: `## [YYYY-MM-DD] action | description`.

## [2026-07-21] update | Deprecate the Roleplay Studio v2 rehearsal + auto-improve loop.
- Removed the rehearsal harness (simulated trainees + QA judge) and the autonomous auto-improve loop (rehearse → critique → apply → re-rehearse) from Roleplay Studio v2 across ally-be, ally-ai-learn, and ally-web; the studio is now Build (Copilot) + Run (Actor + Director), with the trainer testing live / publishing directly.
- Updated `repos/ally-be.md` and `repos/ally-ai-learn.md` to drop the rehearsal-lifecycle / Trainee+Judge / auto-improve descriptions, and `platform/language-quality-eval.md` to stop citing the removed rehearsal harness as an experiment execution engine.

## [2026-07-17] ingest | Document the AI Lab feature.
- Added `platform/ai-lab.md` — concepts (skills/variables/values/runs/evaluators), the author→run→publish→assign→evaluate→results flow, security/multi-tenancy notes, and the roadmap.
- Linked it from the Platform section of `index.md`.

## [2026-07-11] init | Initialize Ally Developer Wiki repository.
- Created `agents.md`, `index.html`, `README.md`.
- Created skeleton documents: `welcome.md`, `getting-started.md`, `index.md`, `log.md`, `overview.md`, `memory.md`, `context.md`.

## [2026-07-11] migrate | Replace Jekyll site with LLMWiki engine.
- Vendored the LLMWiki engine into `llmwiki/`; removed Jekyll scaffolding (`_config.yml`, `_layouts/`, `Gemfile`, `Gemfile.lock`, `_site/`, `assets/`).
- Added `.nojekyll` so GitHub Pages serves raw Markdown to the client-side viewer; retained `CNAME` (tech.helloally.ai).
- Migrated former Jekyll pages (`index.md`, `get-started.md`, `tech-stack.md`, `CONTRIBUTING.md`) into wiki documentation pages.

## [2026-07-11] ingest | Document all platform repositories.
- Added `platform/overview.md`, `platform/architecture.md`, `platform/tech-stack.md`, `platform/agent-guide.md` from workspace `CLAUDE.md` / `AGENTS.md`.
- Added per-repo pages `repos/ally-be.md`, `repos/ally-ai.md`, `repos/ally-ai-learn.md`, `repos/ally-web.md`, `repos/ally-mobile.md`, `repos/infra.md`.
- Added `contributing/guide.md` (SDLC rules) and `contributing/dev-setup.md`.
- Rebuilt `index.md` catalog; updated `overview.md`, `memory.md`, `context.md`.

## [2026-07-11] sanitize | Scrub sensitive data for public hosting.
- Removed internal baremetal hostnames and private domains, literal local-dev credentials, and specific cost figures.
- Verified no IP addresses, cloud region codes, or secrets remain across `wiki/`.

## [2026-07-11] link | Point repos at the wiki.
- Updated/created `AGENTS.md` and `CLAUDE.md` in each code repo to reference this wiki as the canonical source of truth.

## [2026-07-11] reprocess | Refresh docs after repo updates.
- Updated `repos/infra.md`: cross-platform dev-script work (`_os.sh`) is now committed (was documented as uncommitted).
- Regenerated `repos/ally-mobile.md` after the repo was brought current (+27 commits: Scribe voice notes, review read/unread, simulation pause/resume, i18n parity, complete-profile gate).
- Updated `context.md`.

## [2026-07-13] update | Roleplay Studio v2 auto-improve loop.
- Updated `repos/ally-be.md`: documented the new `roleplay-studio/` module capabilities — evidence-rich rehearsal critique with persisted proposals, and the autonomous improve loop (rehearse → critique → apply → re-rehearse with trainer review before draft acceptance).

## [2026-07-13] update | Roleplay Studio v2 goes chat-first.
- Updated `repos/ally-be.md`: the copilot now drives the full loop from the chat (test-case selection, auto-improve start, in-chat progress narration, auto-apply on success, test-live/publish actions).

## [2026-07-15] ingest | Language-Quality Evaluation & RCA framework.
- Added `platform/language-quality-eval.md` from an internal team note: the four-layer language-capability framework, LLM-judge error typology, objective speech metrics (round-trip WER, script fidelity), and single-variable RCA methodology.
- Sanitized for public hosting: dropped author/draft header and companion-doc references to repo-internal engineering specs; removed the unfilled per-language findings table (kept the Kannada worked example). No file paths, credentials, or secrets. Detailed specs remain in the `ally-ai` / `ally-be` repos.
- Linked from `index.md` under Platform.

## [2026-07-22] update | Roleplay Studio v2 copilot gains an Iterate mode.
- Updated `repos/ally-be.md`: after building the spec, the trainer can switch the copilot session from Build to Iterate (`copilot_sessions.mode`), live-test the roleplay, and give plain-language feedback; the copilot reasons from the symptom to the spec field that drives it — grounded in the test sessions' Director telemetry (`get_test_session_insights`) — patches only that, and shows a structured "summary of updates made" (`iteration_summary` / `summarize_iteration`).
- Cross-repo `ROLEPLAY_STUDIO_V2.md` (kept in `ally-be` + `ally-ai-learn`) documents the new Flow 1b — ITERATE, the `iteration_system` prompt (now 8 prompts), and the two Iterate-only tools. No secrets, paths-only.
