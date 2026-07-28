---
title: Activity Log
tags: [operations, log]
summary: A chronological log tracking all wiki updates and modifications.
---

# Activity Log

This is an append-only log of modifications, updates, and indexing runs performed on the wiki. All logs use the parseable prefix format: `## [YYYY-MM-DD] action | description`.

## [2026-07-28] update | Settle the Data Visualisation open questions; promote to Adopted.
- Answered all three open questions in `product/data-visualisation.md` from the super-admin
  Analytics remediation: the minimum sample size before a score is stated (**n = 20**, with the
  "not enough data" treatment), the **approved palette** (semantic scales, grey reserved for
  context, same-hue ramps for ordered categories, name-hashed colours for open value sets, an
  8-hue ceiling) and its greyscale / dark-mode / print answers, and the state of **metric parity**
  between web and mobile (not reconciled, now scoped as a follow-up rather than an open rule).
- Added two principles the audit earned: **never fabricate a measurement for a period that had
  none** (gaps, not zeros; any carried-forward value must be visibly marked) and **a panel that
  cannot honour an active filter must say so**. Both have new checklist gates.
- Added five anti-patterns observed in the codebase: a zero that means "nothing happened",
  colour assigned by result-set position, a part and a whole in one stack, two magnitudes on one
  axis, and comparing across judge versions.
- Promoted the subsection from **Draft** to **Adopted** (hub table updated), so its checklist is
  a gate on future chart work rather than guidance.
- Cross-linked the implementation from `repos/ally-web.md`.

## [2026-07-28] ingest | Chart & Dashboard Design Principles reference.
- Added `product/chart-dashboard-principles.md` — a constraints-file reference for building charts and dashboards (perception rules, chart anatomy, clarity/simplicity/colour, persuasion-vs-deception boundaries, chart-type lookup tables, dashboard composition, acceptance checklist, anti-pattern list). `[EXTRAPOLATION]` markers flag the sections that extend the per-chart rules to multi-chart surfaces.
- Introduced a **Reference pages** concept in the hub: long craft lookups filed under a parent subsection, distinct from subsections themselves. Registered the new page there (parent: Data Visualisation) and in `index.md`; §3 of the hub now covers both cases.
- Cross-linked from `product/data-visualisation.md` (callout, checklist, dashboard-composition note) so the short Ally rule set and the long craft reference stay paired.
- Editorial fixes on ingest: corrected three off-by-one internal section references in the source (§0's section map, and two pointers in §9 that named section 11 instead of 10).

## [2026-07-28] ingest | Add the Product Management Best Practices section.
- Created `product/best-practices.md` — the hub: house rules for every product decision, the subsection table with maturity markers, the procedure for adding a subsection, and a backlog of candidate topics.
- Created five seed subsections (all Draft): `product/ui.md`, `product/gamification.md`, `product/data-visualisation.md`, `product/prioritisation.md`, `product/user-personas.md`.
- Wired the section for discovery by future agent sessions: new **Product Management** group in `index.md`; pointers in `welcome.md` (nav + the pasteable AI-agent setup prompt), `getting-started.md`, `platform/agent-guide.md` (callout + Common Tasks + See also), `platform/overview.md`, `contributing/guide.md` (workflow step 0 + PR description), `overview.md`, `memory.md`, `repos/ally-web.md`, `repos/ally-mobile.md`, and the repo-root `CLAUDE.md`.
- Content is sanitized: conventions and composite personas only — no customer names, real user quotes, PHI, or commercial terms.

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

## [2026-07-28] ingest | Scribe summary write contract & deploy ordering.
- Added `platform/scribe-summary-writes.md`: the destructive whole-document replace that caused months of "scribe notes won't save" reports, the merge-based write contract that replaced it (key present / null / absent semantics), the four client rules (send only edited keys, diff against the seed baseline, no LLM on a save path), form-seeding rules, and the autosave pattern.
- Documented the **backend-first deploy rule** and why the reverse order is silent data loss rather than an error; noted that older clients in the field stay compatible by design.
- Sanitized for public hosting: no environment names, hosting providers, buckets, regions, hostnames, or credentials — the page describes the code contract and ordering rule only.
- Linked from `index.md` under Platform, and cross-referenced from the ally-be / ally-web / ally-mobile repo pages.

