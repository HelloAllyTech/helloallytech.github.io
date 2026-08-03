---
title: Activity Log
tags: [operations, log]
summary: A chronological log tracking all wiki updates and modifications.
---

# Activity Log

This is an append-only log of modifications, updates, and indexing runs performed on the wiki. All logs use the parseable prefix format: `## [YYYY-MM-DD] action | description`.

## [2026-07-31] add | New subsection: AI-Product Patterns.
- New `product/ai-product-patterns.md`, taken off the backlog because the Analytics → Suggestions feature (a model reads a window of platform analytics and proposes roadmap items) forced every question on it. Twelve principles for features where **a model proposes and a person decides**: the reviewed value is what gets stored, provenance is pinned per artefact rather than per run, and a classification that is not in the live set stores `null` rather than a default.
- Records the two rules that came out of building it. "Nothing to propose" is a first-class answer and lists are never padded to a target count — and an empty successful result must be distinguishable from a failed run, or a broken pipeline reads as a healthy product with no ideas. Also: inputs the model could not see are named on the surface, since a silently truncated payload produces a confident claim about data that was never sent.
- Carries [Prioritisation](product/prioritisation.md) principle 12 into generated work: a rejection is stored with its optional reason and fed into later runs as a standing decision, because a generator that never gets tired will otherwise re-propose the same idea every run.
- Registered in the hub Subsections table and under Product Management in `index.md`; the corresponding backlog entry is removed.

## [2026-07-29] update | Candidate analytics panels get a staging tab.
- New "Settled decisions" entry in `product/data-visualisation.md`: a panel that claims to change a decision (principle 9) is staged on the super-admin Analytics → **Testing** tab and judged against real production data before it reaches a surface anyone relies on. Nothing on Highlights is altered to make room, so a candidate that measures the same thing differently (median + IQR against a mean line; a rating-band mix against a mean rating) can be read beside the original — which is the argument for retiring it.
- Records the three rules that keep staging honest: staged panels honour every rule on the page from the first commit (nulls over zero denominators, sample floors travelling with the data, residuals in grey, the accruing period off the plot, same all-time window and per-chart grain), endpoints are named for what they measure and never for the tab, and the tab is a queue rather than a home.
- Lists the eleven panels currently under review, including weekly practising learners as a north-star candidate and score-by-Nth-session as the efficacy curve.

## [2026-07-29] update | Highlights goes all-time, with grouping per chart.
- Added principles 23–25 to `product/data-visualisation.md`: the window and the grain are separate questions and the grain belongs to the chart (server-side re-graining only — a mean of means and an unrecoverable percentile are why); captions must say so where re-graining changes the definition rather than the resolution (new vs returning); and a window with no predecessor gets no delta.
- Added the matching checklist gates, five anti-patterns, and a "Settled decisions" entry recording that the super-admin Highlights tab now has no range picker, resolves `range=all` from a measured data floor (rejecting the range where no floor exists rather than answering for the wrong period), fetches one response per grain on screen, and omits the still-accruing period from every plot while keeping it flagged in tables and exports.

## [2026-07-29] update | Scroll-don't-compress rule into Data Visualisation.
- From a super-admin Analytics report: "New users per period" at daily grain over an all-time window
  drew fifty dates rotated into one solid grey band under hairline bars. Carbon (like every charting
  library) does not overflow when given more categories than the container is wide — it **fits** them,
  silently, and the panel reads as a broken chart rather than as more data than fits.
- New principle 26 in `product/data-visualisation.md`: give each category a floor of horizontal room,
  let the plot exceed its card, and scroll the card sideways. Both costs go on the surface — the value
  axis and legend are in the same SVG as the plot so they scroll out of view with it, and part of the
  range starts off-screen, which a plot cut off at the card edge otherwise hides. The scroll region is
  keyboard-reachable or the range is mouse-only. Thinning ticks to every nth label is rejected: it
  fixes the labels and leaves the marks too thin to compare.
- Resolves the old open question about tick density on a narrow viewport, now a settled decision:
  **28px per x-axis category**, one number for every chart form because the tick label is the binding
  constraint either way. Overflow is measured, not inferred from the count, since whether a plot fits
  depends on the grid breakpoint its card is in — so the tab stop and the off-screen note appear only
  when something is genuinely out of view. Accepted cost recorded: Carbon positions its tooltip
  against the full plot width, so a hover near the scroll window's right edge is clipped.
- The rule is scoped to the axis the categories are actually on. A horizontal bar chart over an
  open-ended set (orgs, models) is crowded **vertically**, so it is deliberately left alone and the
  question of how far to grow such a card's height before the list becomes a table is now an open one.
- Craft detail in `product/chart-dashboard-principles.md` §11.2 as an ordered preference — coarsen the
  bucket first (it keeps the whole range in one view), then scroll, then move to a table — plus two
  anti-patterns.
- Implemented as `MIN_CATEGORY_WIDTH` + the `ScrollableChart` wrapper in `ally-web`
  (`apps/ally-admin-dashboard/src/pages/Analytics/chartKit.tsx`, `.analytics-chart-scroll` in
  `analytics-carbon.scss`), wired into every time-bucketed and open-ended-categorical plot across the
  Highlights, Scribe, Latency, Language-quality and AI-cost tabs and their expanded views.

## [2026-07-29] update | Count-distribution rules into Data Visualisation.
- From adding the **roleplay-volume** chart to super-admin Analytics -> Highlights: learners banded
  by how many roleplays they have completed over their lifetime (0 / 1 / 2 / 3–5 / 6–10 / 11–25 /
  26–50 / 51+). Two new principles in `product/data-visualisation.md`. **21** — a distribution over
  people is plotted as **counts**, with the shares in the takeaway and the drill-down table: counts
  compare honestly at both ends of a skewed axis and leak nothing, so the minimum-group-size floor
  suppresses only the percentages instead of blanking the panel. **22** — band the quantity the way
  it reads: a *discrete count* gets bands inclusive at both ends ("3–5" means 3, 4 or 5), a
  *continuous* quantity keeps lower-inclusive/upper-exclusive and says so; band fine where the
  decisions are (1 vs 2 is the activation question) and coarse where nothing changes; and band a
  **lifetime** count over all time, because inside a 30-day window everyone lands in the lowest
  bands whatever their real depth and the chart would report the length of the window.
- Principle 17 (residual zero band, greyed as context) already covered the "never completed one"
  bar and is reused rather than restated.
- Implemented as `GET /v1/analytics/roleplay-volume` in `ally-be` (`analytics/`, all-time, `tenantId`
  only) and `RoleplayVolumeCard` in `ally-web`'s admin Analytics -> Highlights tab.

## [2026-07-29] update | Stat-tile definition rule into Data Visualisation.
- From giving the super-admin Analytics -> Highlights **KPI strip** a one-line definition per tile.
  A tile of label + number + arrow is not self-describing: "Active orgs 0 ↓ −1" needs the reader to
  already know that *active* means a completed simulation and that the count is bounded by the
  range picker. New principle 20 in `product/data-visualisation.md`: the definition goes **below**
  the value (the number keeps the salience), renders in **every state** including loading and
  thin-sample (it describes the metric, not the value), and is never a tooltip — a KPI strip is the
  part of a dashboard most often screenshotted on its own. An unattributable tile (AI cost per sim)
  carries the principle-14 unscoped label itself, not only on the chart it summarises.
- Implemented as an optional `description` on `KpiTile` in `ally-web`
  (`apps/ally-admin-dashboard/src/pages/Analytics/chartKit.tsx`), with the wording of each
  definition taken from the derivation the corresponding chart already cites. Matching checklist
  gate and an "undefined KPI strip" anti-pattern added.

## [2026-07-29] update | Population-share rules into Data Visualisation.
- From building the super-admin Analytics -> Highlights **usage-levels** panel (each month's
  learners split by practice minutes — 0 / <10 / 10-25 / 25-50 / 50-100 / 100-500 / 500-1000 /
  1000+ — as a 100%-stacked bar over 12 complete months). Three new principles in
  `product/data-visualisation.md`: a **"did none of it" band is a residual, not a measured
  category** (derive it from the chosen denominator, grey it out of the ordered ramp, clamp it at
  zero), and **a 100%-stacked chart hides its own denominator** so the population has to travel
  with it. The existing competing-definitions rule earned its second implementation: "percentage
  of users" ships with an on-panel switcher between every registered learner and only those who
  ever practised, both computed from one pass over one set of band counts.
- The n = 5 per-person floor now names its third implementation, `MIN_USAGE_POPULATION`, which
  re-exports the cohort constant instead of restating the number. Where the floor applies to a
  denominator the client chooses, the API returns the floor rather than a per-row flag.
- Two anti-patterns: a share chart whose denominator is a silent choice; the dormant band coloured
  as the bottom of the usage ramp.

## [2026-07-29] update | Cohort-retention rules into Data Visualisation.
- From building the super-admin Analytics -> Highlights **learner cohort retention** panel
  (monthly cohorts x months-since-signup, with a 10/50/100 practice-minutes "active user"
  switcher). Four new principles in `product/data-visualisation.md`: a **definitional point is
  not a measurement** (a cohort's month 0 is 100% by construction and must say so), an
  **in-progress period belongs in a table before a line** (flag it in a grid, omit it from a
  chart), **let the reader switch between competing metric definitions** and compute them all
  against one denominator in a single pass, plus the matching checklist gates and two
  anti-patterns (a cohort triangle whose future is drawn as 0%; a retention rate over three
  people).
- New settled decision: **minimum group size for a per-person breakdown = 5** — show the size,
  suppress the rate, never drop the row. Distinct from the existing n = 20, which is about
  whether a derived score is trustworthy rather than about re-identification.
- One craft rule into `product/chart-dashboard-principles.md` §8.4: **a colour ramp under printed
  values must stop short of full saturation**, or the light/dark text switch lands in a band
  where neither colour clears 4.5:1.

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

## [2026-07-30] ingest | Analytics Agent (natural-language questions over platform data).
- Added `platform/analytics-agent.md`: the new super-duper-admin **Analytics Agent** sub-tab of the admin Analytics page — plan → guard → execute → narrate, why planning and narration are separate LLM calls, the planner's clarify/refuse intents, the three-layer trust boundary (table allowlist, denied columns, READ ONLY + statement_timeout + row cap), why the schema catalogue is introspected rather than hand-written, chart validation and the missing-measurement rules, browser-held conversation state, cost labels, and the explicit non-goals.
- Recorded the product judgement calls this feature forced against the wiki's existing rules: charts dropped when the rows cannot honestly carry them, nulls rendered as gaps rather than zeros, truncated results labelled as lower bounds, provenance on every answer, and the four non-answers as four distinct screens.
- Sanitized for public hosting: describes the pipeline, policy and reasoning only — no table/column inventories, model ids, environment names, hostnames, or credentials. The allowlist and denied-column policy live in the ally-be code.
- Linked from `index.md` under Platform, and cross-referenced from the ally-be / ally-ai / ally-web repo pages.


## [2026-08-03] update | INTERNAL role and the admin console at /admin on the consumer app.
- Updated `repos/ally-be.md` with a **Roles** section under the auth model: roles are `groups` rows joined via `user_groups` (no `role` column on `users`), the ten role names, and the three platform-level ones collected in `SUPER_ADMIN_ROLES`. Documents the new `INTERNAL` role — a permission-for-permission clone of `SUPER_ADMIN` for Ally staff, who reach the console without being listed or managed as super admins.
- Recorded the two things that catch people out: group grants do not inherit (the TypeScript spreads in `permissions.constants.ts` do not back-fill `group_permissions`, so a new SUPER_ADMIN permission must be granted to SUPER_DUPER_ADMIN and INTERNAL in the same migration), and `GET /users/me` collapses roles to one by a priority list — INTERNAL is deliberately not in that list, so clients must gate on the `roles` array instead. Also noted the 30-minute Redis TTL on role/permission lookups and the post-migration flush.
- Updated `repos/ally-web.md`: the admin dashboard now ships to two surfaces from one codebase — standalone at `/`, and embedded at `/admin` on the consumer origin for INTERNAL holders. Documented the single knob that decides it (Vite `base` via `VITE_ADMIN_BASE_PATH`, echoed back as `import.meta.env.BASE_URL`), consumer-session adoption on the shared origin, and the whole-console access gate that follows from it.
- Noted the honest caveat rather than overstating the split: `allowedRoles` is client-supplied, so "INTERNAL only signs in via /admin" is a routing convention, not a server-side boundary — and not a privilege question, since the two roles carry identical permissions either way.
- Sanitized for public hosting: describes roles, build configuration and routing requirements only — no environment names, hostnames, distributions, buckets, regions, or credentials. The deployment-specific routing lives in the ally-web repo's `docs/embedded-admin-console.md`.
