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

## 4. Recent improvements

A round of improvements landed across `ally-be`, `ally-web`, and `infra`:

- **Per-skill generation params** — temperature, max output tokens, and an optional system
  prompt, applied per provider (temperature only where the model supports it).
- **Token & cost tracking** — completed runs record prompt/completion/total tokens and an
  estimated USD cost (per-model pricing table); shown in the run detail drawer.
- **Asynchronous execution** — when a lab-run SQS queue is provisioned, runs are queued
  (PENDING) and executed by a worker with retry/DLQ handling; the runs log polls for live
  status. Falls back to synchronous execution when no queue is configured.
- **Matrix runs** — pick multiple values per variable to run a skill across the cartesian
  product of a value set (one row per combination, shared batch id), with a **side-by-side
  batch comparison** view.
- **Automated (LLM-as-judge) evaluation** — score a completed run's output against a rubric
  with a judge model, alongside human evaluation.
- **Results analytics** — CSV/JSON export and a per-question inter-rater agreement indicator.
- **UX & delivery** — per-skill failure surfacing when creating runs, evaluator invite emails,
  a focus-trapped shared side panel, and paginated runs; plus the first automated tests for
  the module (backend services + a frontend spec).

### Still open
- Migrate the remaining hand-rolled AI Lab drawers onto the shared (focus-trapped) `SidePanel`.
- Extend pagination to the Skills / Variables / Values / Evaluators tabs.
- Provision the lab-run SQS queue + DLQ in AWS and set `lab_run_queue` / `lab_run_dlq` so async
  execution goes live.
