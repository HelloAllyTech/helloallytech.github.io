---
title: Product Management Best Practices
tags: [product, best-practices, hub, ux, design, prioritisation, agents]
summary: The hub for Ally's product-management conventions — the house rules that apply to every product decision, plus a growing set of subsections (UI, Gamification, Data Visualisation, Prioritisation, User Personas, …).
---

# Product Management Best Practices

This section is the **product-side counterpart to the engineering docs**. The rest of this
wiki explains *how the system is built*; this section explains *how we decide what to build
and how it should behave for the people using it*.

> [!IMPORTANT]
> **Agents: read this before you write product-facing code.** If a task changes what a user
> sees, does, earns, is shown as data, or is asked to prioritise — open the relevant
> subsection below **before** designing the change, and say in your plan which practice you
> applied. If the task is purely internal (refactor, migration, infra, test), skip it.

Everything here is a **team convention**, not a law of nature. Conventions are meant to be
argued with and edited — but changed deliberately, in this wiki, not silently in a PR.

---

## 1. Subsections

The section grows over time. This table is the canonical list — every subsection page must
appear here **and** under **Product Management** in [`wiki/index.md`](../index.md).

| Subsection | Covers | Maturity |
|---|---|---|
| [UI & Interaction](ui.md) | Screen and component decisions, states, hierarchy, accessibility, i18n, cross-surface consistency | Draft |
| [Gamification](gamification.md) | Badges, streaks, leaderboards, progress — and when *not* to gamify | Draft |
| [Data Visualisation](data-visualisation.md) | Charts, dashboards, metrics displays, and the honesty rules for showing numbers | Adopted |
| [Prioritisation](prioritisation.md) | Deciding what gets built, in what order, and what gets explicitly dropped | Draft |
| [User Personas](user-personas.md) | Who we build for — counsellor, learner, trainer/admin, super-admin — and what each one actually needs | Draft |

**Maturity** is one of `Draft` (written, not yet battle-tested), `Adopted` (the team follows
it), or `Deprecated` (kept for history; says what replaced it). Not-yet-written topics are
tracked in the **Backlog of subsections** below — this is an expanding section, so an idea listed
there is an invitation, not a gap.

### Reference pages

Longer craft references that sit *under* a subsection rather than beside it. A subsection is
the short, opinionated rule set you hold in your head; a reference page is the detailed
lookup you open while building. Each one must name its parent subsection.

| Reference | Parent subsection | What it is |
|---|---|---|
| [Chart & Dashboard Design Principles](chart-dashboard-principles.md) | Data Visualisation | A constraints file for building charts and dashboards: how people read a chart, required anatomy, clarity/simplicity/colour rules, the persuasion-vs-deception boundary, chart-type lookup tables, dashboard composition, an acceptance checklist and an anti-pattern list. |

---

## 2. House rules (apply to every subsection)

These hold regardless of which subsection you are in. A subsection may add rules; it should
not contradict these.

1. **Name the persona before the feature.** "Users want X" is not a requirement. Which of the
   [personas](user-personas.md) — and what were they doing 30 seconds before this screen? A
   counsellor mid-session and a trainer reviewing last week's cohort are different products.
2. **Start from the job, not the surface.** Decide what the person is trying to accomplish
   before deciding whether it's a tab, a drawer, a chart, or a notification.
3. **Write the empty, loading, error, and "too much data" states in the spec.** They are not
   polish — they are most of the real-world experience of any Ally screen backed by a live
   voice session, an LLM call, or an async queue. See [UI & Interaction](ui.md).
4. **Design for the slowest path you actually ship on.** Ally's product surfaces sit on top
   of real-time voice, LLM generation, and queue-driven pipelines
   ([Architecture](../platform/architecture.md)); latency and partial results are product
   problems, not just engineering ones.
5. **Multi-tenant by default.** Ally is multi-tenant with permission-gated features
   ([ally-be](../repos/ally-be.md)). Every product decision needs an answer to: which roles
   see this, which tenants get it, and what does the screen look like for someone who has the
   page but not the permission?
6. **Clinical-adjacent means restraint.** Ally supports mental-health counsellors and their
   training. Tone, celebration, scoring, and comparison all land differently here than in a
   consumer app — be deliberate, especially in [Gamification](gamification.md).
7. **Privacy is a product constraint, not just a compliance one.** HIPAA and PHI rules shape
   what can be shown, shared, exported, or put in a leaderboard. When in doubt, show less.
8. **Localisation is not a phase-two.** The product ships in multiple Indian languages
   (English, Hindi, Kannada, Marathi, Tamil at time of writing —
   [ally-mobile](../repos/ally-mobile.md)). Copy length, number/date formats, and
   right-sizing of labels are design inputs, not translation chores.
9. **Decide what you are *not* doing, in writing.** A scope without explicit non-goals gets
   re-litigated in review. See [Prioritisation](prioritisation.md).
10. **Instrument the decision.** If a feature ships without a way to tell whether it worked,
    the next prioritisation conversation about it will be pure opinion.
11. **File it back.** If you make a product judgement call that isn't covered here, add it to
    the relevant subsection in the same change. This section only stays useful if it
    compounds.

---

## 3. How to add or update a subsection

Follow the standard wiki [Ingestion Flow](../getting-started.md) plus these specifics:

1. **Create** `wiki/product/<kebab-case-topic>.md` with the frontmatter block
   (`title`, `tags` including `product`, `summary`).
2. **Structure it** like the existing pages so they stay scannable and diffable:
   - *Why this matters for Ally* — one short paragraph, no filler.
   - *Principles* — numbered, each one a decision rule you could actually apply.
   - *Checklist* — what to verify before calling the work done.
   - *Anti-patterns* — the mistakes we have actually made or nearly made.
   - *Ally-specific notes* — links into the platform/repo pages.
   - *Open questions* — what is unresolved, so the next session picks it up instead of
     re-deriving it.
3. **Register it** in the **Subsections** table above — or, if it is a long craft reference
   rather than a peer subsection, in the **Reference pages** table with its parent named —
   **and** under **Product Management** in [`wiki/index.md`](../index.md).
4. **Log it** in [`wiki/log.md`](../log.md) using `## [YYYY-MM-DD] action | Description`.
5. **Cross-link it** from at least one engineering page it touches, so the practice is
   reachable from the code side (e.g. a UI convention should be linked from
   [ally-web](../repos/ally-web.md)).
6. **Apply the public-content policy** — no secrets, credentials, customer names, real user
   quotes with identifying detail, PHI, internal hostnames, or unreleased commercial terms.

Keep pages **opinionated and short**. A practice nobody can hold in their head is a practice
nobody applies.

---

## 4. Backlog of subsections

Candidate topics, unwritten. Pick one up when a real decision forces the question — writing
these speculatively produces filler.

- Onboarding & first-run experience
- Notifications, nudges & interruption budget
- Copy, tone of voice & terminology (incl. clinical language)
- Accessibility (deepening the [UI](ui.md) section into its own page)
- Pricing, packaging & tenant tiering
- Experimentation & A/B testing (what is even testable at our traffic)
- Feedback loops: support tickets, session reviews, trainer interviews
- Release comms & change management for existing tenants
- AI-product patterns: disclosure, confidence, correction and human-in-the-loop
- Sunsetting and deprecating features

---

*See also: [Cross-Repo Agent Guide](../platform/agent-guide.md) ·
[Platform Overview](../platform/overview.md) ·
[Contributing Guide](../contributing/guide.md)*
