---
title: Activity Log
tags: [operations, log]
summary: A chronological log tracking all wiki updates and modifications.
---

# Activity Log

This is an append-only log of modifications, updates, and indexing runs performed on the wiki. All logs use the parseable prefix format: `## [YYYY-MM-DD] action | description`.

## [2026-08-10] update | Stacks MCP narrowed to a single prompt; every repo's hook, skill and router rewritten to match.
- **What changed upstream.** The server now exposes one prompt, `planning_context` (arg `task_description`, one hybrid search, returns 8 chunks with ids), and one tool, `get_chunks` (1–20 ids from such a block). `search_chunks`, `list_tags` and `list_documents` were **removed outright** — calls to them return "Tool not found".
- **The structural consequence, which is the part that matters.** A prompt only runs when a human invokes it. So an agent can no longer reach the library on its own initiative, and can no longer enumerate what the library contains. A session asserting "Stacks has nothing on X" is making a claim it has no way to check; the wiki now says so in `contributing/planning-with-stacks.md`, `platform/agent-guide.md` and `memory.md`.
- **Nothing failed loudly.** The `UserPromptSubmit` hook kept posting `tools/call` for `search_chunks`, got a JSON-RPC error, resolved its result path to empty and exited 0 — its documented fail-open behaviour. Every repo had been silently injecting no context for as long as the tool had been gone.
- **The hook now posts `prompts/get`.** Result text moved from `content[0].text` to `messages[0].content.text`. Three consequential differences: there is no `max_results`, so it trims the 8 returned chunks to the 3 strongest client-side (~1.4k tokens, holding the old budget); the curl timeout went 8s → 10s because returning 8 chunks measures 4.0–5.8s where a 3-chunk search was quicker; and a 25-second machine-wide cooldown was added.
- **Why the cooldown.** The server dropped its own rate limit, but its upstream embedder (Voyage, free tier) is capped at **3 requests/minute** shared across every session, every engineer and the hook. Over the ceiling the call errors and the hook silently injects nothing — indistinguishable from "no guidance found". Without spacing, an automatic fire eats the quota an engineer's own `/stacks:planning_context` needs. This ceiling is now documented as a warning on the Stacks page.
- **Scope.** 8 identical copies each of `.claude/hooks/stacks-search.sh` and `.claude/skills/stacks/SKILL.md` (workspace root + 7 repos), 13 `CLAUDE.md`/`AGENTS.md` routers, and 7 wiki pages. `.mcp.json` was unchanged — same URL, same `${STACKS_API_KEY}` expansion.

## [2026-08-10] update | Widened the Stacks rule from planning-only to any product judgement, and wired it to fire without being invoked.
- **The diagnosis.** Stacks was reached only during planning, and the reason was not the wiki — it was the MCP server's own copy. `SERVER_INSTRUCTIONS` ("During planning or design work…") and the `search_chunks` tool description ("Call this during planning, architecture, or design discussions") are what a connected Claude Code session reads to decide when a tool applies. A session reads that, concludes Stacks is a planning tool, and stops querying the moment implementation starts — no matter what a repo's `CLAUDE.md` says. That copy had been treated as frozen §11 text; it was unfrozen and widened on purpose.
- **The rule now covers three moments**, not one: before an implementation plan (still the most important), mid-implementation at each point an answer would otherwise be invented (an empty, loading, edge or failure state; a user-facing label, button or error message; what a view shows and omits; a threshold, limit, cadence or reward rule), and review for behaviour rather than reading. Trivial mechanical work is still exempt.
- **Three delivery layers, all committed, none requiring a manual step.** Each Ally repo now carries `.claude/skills/stacks/SKILL.md` — a skill whose one-line description sits in context every turn, so the rule keeps re-asserting itself late in a long session where a startup-read `CLAUDE.md` line has stopped competing for attention — plus a `UserPromptSubmit` hook (`.claude/settings.json`, `.claude/hooks/stacks-search.sh`) that runs a small search when a prompt looks product-shaped. Both sit alongside the existing `.mcp.json` and the `CLAUDE.md`/`AGENTS.md` rule.
- **The hook is deliberately conservative and fails open.** It gates on product-judgement phrasing (verified silent on "fix the typo", "bump axios", "why is this test failing"; verified firing on "what should the empty state show", "add a streak counter", "the right wording for this error message"), asks for 3 chunks rather than the server default of 8 (~1.3k tokens, not ~3.4k — the difference between a nudge and forcing compaction), stops after 4 injections per session, and exits 0 silently on a missing key, a dead endpoint or malformed input. **It is a floor, not the rule** — because it fails silently, it is never evidence a search happened.
- **The two sides can drift apart silently.** Narrowing the server copy back to planning language re-breaks every repo's instructions with nothing failing anywhere; the coupling is recorded in the Stacks repo's `docs/retrieval.md` and in `contributing/planning-with-stacks.md`.
- Page retitled *Planning with the Stacks MCP* → *Working with the Stacks MCP*; the filename is unchanged, so every existing link and the URLs baked into the seven repos still resolve. Pointers widened in `contributing/dev-setup.md`, `contributing/guide.md` (workflow step 0), `platform/agent-guide.md`, `index.md`, `welcome.md` and `memory.md`.
## [2026-08-10] update | Worker connection lifecycle and what a health signal actually proves.
- `repos/ally-ai-learn.md`: new **Worker connection lifecycle and health signals** block under Architecture. Written after the voice worker fleet spent 19 hours registered with nobody while every signal available read green — orchestrator task status, container health, the service's own health endpoint, and the autoscaling metric.
- The mechanism is worth stating plainly because none of it is visible from the code you would naturally read: the agents SDK retries a lost LiveKit connection on a bounded budget (`max_retry`, default 16 ≈ 2.5 minutes), and on exhaustion it stops trying **without exiting the process**. `start.sh` tears the container down when either of its two processes exits, so the orchestrator would have self-healed — but that path only fires on exit, and a worker that has given up is still resident.
- The page now distinguishes the two HTTP surfaces: FastAPI's `/api/health` returns a static `ok` and proves only that the HTTP process is up, while the SDK's own health server 503s on a failed connection or a dead inference process. Health checks must target the second; targeting the first is indistinguishable from targeting nothing. Same caveat recorded for the active-session metric, which is emitted from the SDK's load hook and keeps publishing at full rate while disconnected — a demand signal, not a liveness one.
- Also notes that dispatch is by name, so a backend/worker name mismatch presents identically to an absent worker.
- Filed as a lesson in `memory.md`. Sanitized for public hosting: no hostnames, addresses, environment names or region details — the incident is described structurally.

## [2026-08-09] update-engine | Fixed the update.md §4 bug upstream and re-vendored the engine.
- The bug logged in the entry below is **fixed at the source**, not worked around here: [ajeygore/llmwiki#9](https://github.com/ajeygore/llmwiki/pull/9), merged. `setup.py` gains a `--root <dir>` argument, so the upgrade manual's "generate throwaway templates and diff them" step can finally be expressed without writing into the live wiki.
- **Why a flag rather than a docs fix.** `setup.py` resolves its target from the engine's own location — the current working directory never had any effect and was never meant to. That is correct for bootstrapping (`python3 path/to/llmwiki/setup.py` should set up *that* wiki from anywhere), so the resolution rule stayed and the missing capability was added. Default behaviour is unchanged, so every existing invocation still works.
- Upstream also gained `tests/test_setup.py` covering all three cases (`--root` targets the named directory and leaves the engine's workspace alone; the default target is the engine's parent whatever the cwd; a missing `--root` directory is created), wired into CI. The test was verified to fail against the pre-fix script — a regression test that never fails against the bug is not a regression test.
- Re-vendored the fixed engine here and re-ran the reconciliation with `--root`: the repo is untouched by template generation, `index.html` is identical to the freshly generated template, and the only remaining differences in `agents.md` are this wiki's deliberate customizations — the Stacks MCP house rule, the two-linter step, the public-site secrets clause.
- **The workaround recorded below is now obsolete.** Copying `llmwiki/` into the scratch directory still works and is kept in `update.md` for anyone on an older engine, but `--root` is the supported path.

## [2026-08-09] update-engine | Upgraded LLMWiki engine to latest (vendored), reconciled index.html and agents.md.
- Engine replaced wholesale per `llmwiki/update.md` §3B — the copy dated from the July migration commit and had never been locally edited, so the "never modify `/llmwiki/`" rule held and a clean swap was safe. New in the engine: a `lint` pass, a `tests/` suite, `update.md` itself, and `.gitattributes`/`.gitignore`.
- **Template reconciliation (§4).** Two upstream changes reached the root templates and were merged in by hand: the **Print button** in `index.html` (now byte-identical to the freshly generated template), and a new **"Reprocess After Every Major Task"** section in `agents.md`. Local customizations were preserved — the Stacks MCP house rule and the `scripts/wiki_lint.py` automation note both survive untouched.
- Step 6 of the merged section was adapted rather than copied: this wiki has **two** linters now, so it runs `scripts/wiki_lint.py` (Ally-specific, the CI gate) *and* `./llmwiki/lint` (engine generic), and regenerates the routing indexes first. The secrets rule gained the public-site clause — no hostnames, addresses or region details either.
- **`update.md` §4 has a bug worth knowing:** it tells you to generate reference templates by `cd`-ing to a scratch dir and running `python3 <workspace>/llmwiki/setup.py`. `setup.py` resolves paths relative to *the script*, not the cwd, so it writes into the real workspace instead. Here it created a `.github/workflows/pages.yml` and a `.llmwiki-port`. The working method is to copy `llmwiki/` into the scratch dir and run it from there.
- Consequences of that were undone deliberately: **`pages.yml` was removed** — this site already publishes through branch-based Pages with `.nojekyll` + CNAME, and the generated workflow publishes the entire workspace including `raw/`, which is a footgun rather than an upgrade. **`.llmwiki-port` was kept** (it is a real new engine feature) but repinned from 8001 to **8899**, because 8001 is `ally-be`'s dev port in this workspace.
- Verified: both linters pass (one warning, `ROUTING.md` has no frontmatter — it is generated), and the viewer serves the index, wiki pages, engine assets and the new Print button.

## [2026-08-09] add | Operator access to the fleet — SSH and VPN — is documented for the first time.
- **The gap:** `repos/infra.md` described Wireguard only as container-to-container peering seeded by cloud-init, which is accurate and answers a question nobody asks. It never said how *a person* gets onto the private network. New **Operator Access** section splits the two meanings of "Wireguard" in this repo — machine peering versus human access through a self-service portal — and states the rules that follow.
- **Provision operator peers through the portal, never by hand.** The portal owns the peer registry in its own database and reconciles the hub's live interface from it, so a hand-added peer works until the next reconcile and then silently disappears.
- **A Wireguard identity cannot be kept warm on a second host.** A standby built by copying the hub's config inherits the hub's private key rather than backing it up; two hosts then answer as the same peer while their registries drift. Found in a fleet survey — identical public keys on the same tunnel address, registries ten entries apart, one host taking no handshakes at all. Standbys get their own keypair; failover moves where clients point.
- **Host access is not uniform and the inventory does not say so.** Root login works on most hosts and not all; group names describe intent rather than hardware, and a group blanket-setting the connecting user breaks on hosts that only permit the unprivileged one. Documented as a per-host fact to verify, not a default to assume.
- Six operational learnings added to `memory.md` under a new *About operating the fleet* heading, including two that are cheap to repeat: a cleanly stopped unit appears in no failed-units list and `enabled` is never exercised on a host that does not reboot; and a runtime probe that *runs* the tool rather than using `command -v` will install LXD on a production host via Ubuntu's snap stub.
- Prompted by a live incident — the access portal had been cleanly stopped for two months and was found by hand rather than by an alert, because nothing monitors it and the fleet's only metrics exporter is itself failed.
- Sanitization: the survey behind this entry is full of hostnames, addresses and keys; none of it is here. Topology is stated in concepts only, per the public-site content policy. The identifier-bearing inventory stays out of this repo.

## [2026-08-07] restructure | Documentation system: routing, deduplication, CI enforcement, and automated reconciliation.
- **New spec:** `contributing/docs-system.md` documents the whole machinery — the layer model, the `.docs-map.yml` contract, the CI guard, the one-command wiki-PR path, PR lifecycle coupling, and the weekly sweep. Read it before changing anything under `scripts/` or `.github/workflows/`.
- **Routing over redirects.** Each code repo's `AGENTS.md` was an identical 21-line pointer to this site; it is now a per-repo router (≤120 lines) keyed by *what the agent is about to do*, with the gotchas that change behaviour inline. `CLAUDE.md` is a byte-identical copy, asserted by CI.
- **Generated indexes.** `scripts/gen-routing.py` builds `wiki/ROUTING.md` (~970 tokens, one line per page) and `wiki/manifest.json` from page frontmatter; `ROUTING.md` is synced into every code repo so an agent can pick a page before spending a fetch. Every page now carries `last_reconciled`.
- **Deduplication.** Branch naming was stated three times and contradicted itself (`feat/` vs `feature/<ticket-id>`); the Contributing Guide is now the single canonical statement and the ticket-id variant is gone. Four identical `CONTRIBUTING.md` copies became pointers, and the shared ~75% of the four `RELEASE_GUIDE.md` files moved to `contributing/release-process.md` — infrastructure identifiers deliberately stayed in the private repos.
- **Enforcement.** A shared `docs-guard` workflow in `helloallytech/.github` reads each repo's `.docs-map.yml` and fails a PR that changes covered code without moving its doc, with a `docs:skip` label and a `Wiki-PR: none — <reason>` trailer as the two visible escape hatches.
- **The cross-repo friction is gone.** `scripts/wiki-pr.sh` turns a wiki edit into one command from any code repo; `wiki-pr-lifecycle.yml` then merges the wiki PR when its source merges, closes it when the source is abandoned, and nudges after 7 days. That friction — not forgetfulness — is why `architecture.md`, `tech-stack.md`, `dev-setup.md` and `repos/infra.md` sat unchanged from 2026-07-11 while three code repos committed daily.
- **Reconciliation.** `wiki-health.yml` runs `scripts/wiki_lint.py` weekly for the drift no PR triggers, and regenerates `platform/stats.md` from the actual code. The first run showed how far prose had drifted: "43 modules" is 59, "211+ migrations" is 388, "9 event types" is 10. Counts now live in the generated page; prose states invariants.
- Merged with the Stacks MCP change that landed the same day: the routers' planning gate is the `search_chunks` call, and the deprecated `wiki/product/` pages are routed to as history for Ally-specific traps rather than as a gate.

## [2026-08-07] deprecate | Product Management Best Practices superseded by the Stacks MCP.
- The whole `wiki/product/` section is **Deprecated** — hub, all six subsections (UI, Gamification, Data Visualisation, Prioritisation, User Personas, AI-Product Patterns) and the Chart & Dashboard craft reference. Product guidance now comes from the external Stacks corpus at planning time; judgement calls are filed into Stacks, not back into this wiki. §3 "How to add or update a subsection" is closed and the backlog of unwritten subsections is abandoned.
- **Tombstoned, not deleted**, using the hub's own `Deprecated` maturity ("kept for history; says what replaced it"). Two reasons the pages stay: `platform/analytics-agent.md` cites them by principle number in five places (Data Visualisation 13 and 27–28, UI & Interaction 4–5), so removing or renumbering breaks those citations silently; and the pages record Ally-specific findings a general corpus has no reason to hold — Carbon's chart-overflow behaviour, the `roles[]`-vs-collapsed-`role` gating trap, minimum group size for tenant-isolated metrics. Every page carries a deprecation banner, a `Deprecated (2026-08-07)` maturity line and a `DEPRECATED …` prefix on its frontmatter summary so search results say so.
- Nothing in the section is a gate any more — including the Data Visualisation checklist, which `repos/ally-web.md` had described as "a gate, not guidance".
- Pointers redirected across the wiki: `platform/agent-guide.md` (the "what to build" callout now names Stacks, with the deprecation as a NOTE), `contributing/guide.md` (workflow step 0 and the user-facing PR checklist), `welcome.md`, `platform/overview.md`, `overview.md`, `index.md` (section header marked deprecated, every entry tagged), `memory.md`, and the three repo pages that linked in. `contributing/planning-with-stacks.md` gains a section recording the reversal — the precedence written one commit earlier (wiki binding, Stacks advisory) is now inverted — and tells engineers to check the deprecated pages when a Stacks query comes back empty on something Ally-specific.
- Also updated the same rule in every repo's `CLAUDE.md`/`AGENTS.md` and the workspace `CLAUDE.md`.

## [2026-08-07] add | Stacks MCP is now a required step before any implementation plan.
- New `contributing/planning-with-stacks.md`: before writing an implementation plan, call the stacks MCP's `search_chunks` tool with 2–3 queries covering the task's main topics, incorporate relevant returned guidance, and cite chunk titles. Covers query technique (topics, not ticket titles; 2–3 queries because one retrieves a single neighbourhood of the corpus), the citation format, and the rule that a search returning nothing relevant is stated in one line rather than skipped silently — a silent skip and a genuine miss are indistinguishable to a reviewer.
- Records the precedence rule: the wiki's own [Product Management Best Practices](product/best-practices.md) are house rules and binding; Stacks is a broader advisory corpus. Where they conflict the wiki wins, and anything from Stacks that genuinely beats a settled practice is filed back into the matching subsection in the same change rather than left to compete with it.
- Distribution: each repo (`ally-be`, `ally-ai`, `ally-ai-learn`, `ally-web`, `ally-mobile`, `infra`, and this one) now commits a `.mcp.json` at its root declaring the `stacks` server with the credential expanded from `${STACKS_API_KEY}`, so nothing secret is committed. Setup steps added to `contributing/dev-setup.md`; the rule is called out at the top of `platform/agent-guide.md`, as step 0 of the workflow in `contributing/guide.md`, and inline in every repo's `CLAUDE.md`/`AGENTS.md`.
- The server URL is deliberately **not** printed on this public wiki — it lives only in the committed `.mcp.json` inside the private repos.

## [2026-08-06] update | UI & Interaction: a truncated list needs a control, not just a count.
- Principle 15, a checklist item and the "count-only footer" anti-pattern, filed from the admin Product Roadmap board: it rendered the first 50 of 507 opportunities and reported "50 of 507" with no pager, so 457 rows and every action attached to them lived only in the API. The rule pairs the count with the control — visible range, position in the set, prev/next — and records the two consequences of an offset only being meaningful against the result set it was taken from: every search/filter/sort change returns to the first page in the same state update (or the first request goes out at the stale offset), and page-scoped selections are dropped with it. Also requires the control to stay reachable when a shrinking list leaves the current page empty, so the escape isn't a reload.

## [2026-08-06] update | UI & Interaction: cross-surface links and full-role-set gating.
- Two principles (13, 14) plus a checklist item, from building the consumer app's "Ally Admin" link into the admin console. **13:** a link into another surface is gated on a copy of *that* surface's own entry condition (mirroring the admin console's login `allowedRoles`), hidden when the destination URL isn't configured for the environment, and shaped like it leaves — anchor, external affordance, separated from in-app nav. **14:** derive role checks from `roles[]`, never the collapsed `role`, which the backend picks by a priority list that omits some roles and so misreports exactly the multi-role accounts a cross-surface feature targets.

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

## [2026-08-04] update | UI rules: hidden rows are unmanageable rows, and partial editors must name what they don't own.
- Added two principles to `product/ui.md`. **(11)** An exclusion filter on a management list removes every action attached to those rows, so it has to be paired with a way back in — an opt-in filter for the role allowed to see them, or a surface that owns those records outright. **(12)** When a form owns only part of a record, it must show the rest as read-only, name the surface that owns it, and re-send it on save — a replace-the-whole-set endpoint behind a partial editor silently deletes what the editor never showed.
- Filed from the admin Users tab: platform-role accounts (super admin / super duper admin / internal) are excluded from the list by design, which also made their roles uneditable anywhere in the product — so Ally staff could not be granted consumer-app roles without a hand-rolled API call. Fixed with a super-duper-admin-only "Include Ally staff & super admins" filter, plus the two tier roles held read-only in the role picker so granting LEARNER can never demote a super admin.
- Added the matching checklist items and the "invisible record" anti-pattern. No new pages, so `index.md` is unchanged.

## [2026-08-04] update | UI anti-pattern: the shrink-to-fit dialog.
- Added an anti-pattern to `product/ui.md`: containers (modals, panels) must carry a definite width rather than letting content size them. A `w-auto` dialog grows horizontally as tags/chips are added, so `flex-wrap` never engages and the dialog reflows on every selection. Filed from the admin "Change User Role" modal fix, where the panel was `min-w-[400px] w-auto` and reached ~1030px at seven roles.

## [2026-08-06] update | INTERNAL role and the /admin console surface withdrawn.
- Reversed the [2026-08-03] entry below. The `INTERNAL` role and the copy of the admin console path-mounted at `/admin` on the consumer app have both been removed: Ally staff hold `SUPER_ADMIN` and use the standalone admin dashboard, so the role had no surface left to distinguish it. The role was never assigned in production; ally-be migration `1885000000000` drops the group and its grants.
- `repos/ally-be.md`: the **Roles** section stays — it is the only page the role system has — but is back to nine roles, with `SUPER_ADMIN_ROLES` covering `SUPER_ADMIN` and `SUPER_DUPER_ADMIN`, and a short note that `INTERNAL` existed between the two migrations. The "group grants do not inherit" caveat is unchanged and still applies to `SUPER_DUPER_ADMIN`. The `GET /users/me` caveat keeps its point but now uses `MULTI_TENANT_ADMIN` as the example, which is the live case: it is absent from the priority list, so a user who also holds `ADMIN` collapses to `ADMIN` and is refused by the console's role gate. **Gate on `roles`, not `role`** stands.
- `repos/ally-web.md`: removed **Two surfaces, one build** and the `docs/embedded-admin-console.md` entry from the docs list; the repo's doc is deleted along with the Vite `base` knob, the `build:admin:embedded` script, consumer-session adoption, and the whole-console gate that only existed because of it. The `roles`-array handling (`resolveAdminRole`) is kept — it is what stops a multi-role `MULTI_TENANT_ADMIN` being locked out of the portal.
- `memory.md`: the four lessons are kept and re-tensed rather than deleted. They were about role modelling, grant cloning, one-build-time-value configuration, and module-load-time work in shared modules — all still true, none dependent on the feature surviving.

## [2026-08-06] create | Login `allowedRoles` contract and the role-retirement test case.
- Added `platform/login-allowed-roles.md` after retiring a role took consumer-app login down in production. The auth DTOs validated the client-supplied `allowedRoles` list strictly against ally-be's `UserRole` enum, so the moment the backend rolled out ahead of the web build, every request from the stale bundle failed validation before the handler read the account — a 400 for **every** user of that surface, not just holders of the removed role, which had never been assigned to anyone.
- Documents the mechanism that makes the leniency safe: `allowedRoles` is a filter intersected with the account's real group memberships, never a claim, so an unrecognised name matches nobody and is inert. `IsAllowedRoles()` now drops unknown names and keeps the recognised ones; a list with nothing recognisable left is still a 400. Who gets in is unchanged.
- The page is built around the **test case**, in two layers: the decorator spec's table (known roles / known + retired / non-strings / only-unknown / empty / not-an-array), where the "known + retired" row reproduces the stale bundle's exact payload; and a post-deploy `curl` probe run against a `@example.invalid` address so it exercises validation without sending an OTP to a real person — pass is a 404 from the account lookup, fail is the old 400. A second probe asserts the guard did not become a rubber stamp.
- Closes a caveat the [2026-08-03] entry had already flagged — that `allowedRoles` is client-supplied and therefore a convention, not a server-side boundary. The five-step retirement checklist (clients first, released mobile builds, replacement grants before dropping the group, Redis flush, probe) is the operational form of that.
- Registered in `index.md` under Platform, cross-linked from `repos/ally-be.md`'s Roles section, and filed as a lesson in `memory.md`. Sanitized for public hosting: no hostnames (probes use `<api-host>`), no real addresses, no environment names.
