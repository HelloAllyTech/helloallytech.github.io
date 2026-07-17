---
title: AI Lab — Prompt Workbench & Human Evaluation
tags: [platform, ai-lab, prompts, evaluation, human-eval, admin]
summary: The admin AI Lab — author prompt Skills with variables, execute Runs against Anthropic/OpenAI, and collect structured human evaluations of the outputs.
---

# AI Lab — Prompt Workbench & Human Evaluation

**AI Lab** is an internal workbench in the admin dashboard for prompt engineering and
output evaluation. It lets the team author reusable prompt templates, run them against
production LLMs with different inputs, and gather structured judgements on the results —
both from human evaluators today and (on the roadmap) automated LLM-as-judge scoring.

The code lives in [ally-web](../repos/ally-web.md) (`apps/ally-admin-dashboard`, under
`src/pages/AILab/`) and [ally-be](../repos/ally-be.md) (the `src/lab/` NestJS module,
which owns the Postgres tables). This page is the conceptual overview; the code is the
source of truth.

---

## 1. Concepts

- **Skill** — a named prompt template. Its `content` holds `{{variable}}` placeholders and
  it may pin a specific model (from the platform LLM registry); otherwise the AI Lab default is used.
- **Variable** — a named slot referenced by skills (e.g. `{{transcript}}`).
- **Value** — a candidate value for a variable (a value can be a long block such as a whole
  transcript; the picker shows a one-line label).
- **Run** — one execution of one skill: its placeholders are resolved with chosen values, the
  prompt is sent to the skill's model, and the output (or error) is stored as a log row.
  Runs launched together share a `batchId`.
- **Evaluator** — a human reviewer with an email + generated password who signs in to the
  `/evaluate` micro-app to score published runs.

## 2. Flow

1. **Author** skills, variables, and values in their respective tabs.
2. **Run** — pick one or more skills, choose a value for each referenced variable, and execute.
   Each skill becomes its own log row with a `RUNNING → COMPLETED | FAILED` status.
3. **Publish** a completed run with an evaluation questionnaire. Question types: **RATING**
   (numeric scale), **YES_NO**, and **TEXT**. Questions are frozen at publish time.
4. **Assign** the published run to one or more evaluators.
5. Evaluators sign in to the portal and **submit** answers (all questions required; submissions
   are immutable).
6. **Results** aggregate per question — rating distributions & averages, yes/no counts, and
   free-text answers — plus a record-level normalized score (0–100) that pools ratings across
   differing scales.

## 3. Security & multi-tenancy notes

- AI Lab is gated by dedicated admin permissions (see the `AddAILabPermissions` migration).
- Evaluator accounts are **not** platform users: their JWTs carry a distinct `kind` claim so
  they can never authenticate against platform routes, and a per-account token version revokes
  outstanding tokens when a password is regenerated. Login runs a constant-cost password compare
  (even for unknown emails) to avoid user enumeration, and the portal login is rate-limited by IP.
- Skills/variables/values/runs are system-wide (not tenant-scoped) — this is an internal tool.

## 4. Roadmap

Known improvement areas being tracked: moving run execution off the HTTP request onto the
platform's SQS worker path (with live status), matrix runs over value sets, automated
LLM-as-judge evaluators, per-run token/cost tracking, side-by-side run comparison, richer
results analytics/export, and accessibility/pagination polish on the admin tabs.
