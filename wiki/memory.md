---
title: Memory
tags: [meta, memory, learnings]
summary: A compiled record of long-term agent memories and persistent takeaways.
---

# Agent Memory

This file acts as a persistent memory bank where visiting AI agents document long-term learnings, persistent takeaways, and critical knowledge compiled across sessions.

## Long-term Learnings

### About this wiki
- This repo (`helloallytech.github.io` → **tech.helloally.ai**) is the canonical Ally documentation. It is a **public** GitHub Pages site.
- **Sanitization is a hard rule**: never publish secrets, credentials, IP addresses, internal hostnames/domains, or cloud region details. Architecture, SDLC, deployment, and environment *concepts* are in scope; specific sensitive values are not.
- The site was migrated from Jekyll to LLMWiki. `.nojekyll` is required so raw `.md` files are served to the client-side viewer.
- Never edit `llmwiki/` (upstream engine). Always update `wiki/index.md` (catalog) and `wiki/log.md` when adding pages.
- **`wiki/product/` is the product-management half of this wiki** and is deliberately open-ended: `best-practices.md` is the hub (house rules + subsection table), and each subsection is its own page. New subsections are expected over time — add the page, register it in the hub table *and* `index.md`, and log it. Don't fold a new topic into an existing page just to avoid creating a file.
- The hub distinguishes **subsections** (short, opinionated, holdable-in-your-head rule sets) from **reference pages** (long craft lookups filed under a parent subsection, e.g. `chart-dashboard-principles.md` under Data Visualisation). Keep ingested external material as a reference page and keep the subsection short, rather than letting a 500-line ingest swallow the rule set.
- **Product practice is a precondition, not a reference.** Any user-facing task (UI, gamification, charts, scope/prioritisation calls) should start by reading `wiki/product/best-practices.md` plus the matching subsection; product judgement calls made during a task get filed back there in the same change. Internal work (refactors, migrations, infra, tests) is exempt.

### About the Ally platform
- Ally is a HIPAA-compliant platform for training mental-health counselors: AI simulations, real-time session analysis, and peer review. 7 repos (see [Platform Overview](platform/overview.md)).
- Core data flow: web/mobile → **ally-be** (NestJS) → **ally-ai** (analysis/transcription) + **ally-ai-learn** (LiveKit voice training) via REST/SQS/LiveKit. Details in [Architecture](platform/architecture.md).
- Service-to-service auth uses `X-API-Key`; client auth uses JWT/OTP/Google OAuth; all REST APIs are `/api/v1/`.
- SQS queues bridge services (transcription request/response, learn message-and-event, audio upload), each with a DLQ; LocalStack emulates them locally.
- Runtimes: ally-be Node 24, ally-web Node 22, ally-mobile Node ≥18, ally-ai / ally-ai-learn Python 3.12+.
- Each code repo's `AGENTS.md`/`CLAUDE.md` points here as the source of truth — including at the product practices under `wiki/product/`. Update the wiki when architecture, workflows, environment setup, **or product conventions** change.
- Product-relevant platform facts that keep recurring in product decisions: multi-tenant with permission-based (not role-string) gating; HIPAA/PHI limits what can be shown, exported or compared; the product ships in several Indian languages; and many user-visible numbers are LLM-derived scores over small samples, so sample size and rubric/model version matter whenever they're displayed.
- **LLM-generated SQL is a product surface with a trust boundary, and the boundary belongs in the service that owns the database.** The Analytics Agent's pattern generalises: the language service plans and narrates, the data service decides what is readable and enforces it (allowlist + denied columns + `READ ONLY`/timeout/row-cap), so everything that could expose data stays reviewable in a handful of files and the prompts can change freely. Two rules that came out of building it are now in `product/data-visualisation.md` (27, 28): a run-time-chosen chart must be validated against the actual result before it renders — and "no chart" is a legitimate answer — and a number produced by a generated query must show that query, including when the query was refused.
- **A generated-answer surface needs its non-answers designed.** "Ask me anything" tools fail in four distinct ways (ambiguous question, unanswerable from the data, query refused by policy, query failed to run) and each needs its own screen and severity. Collapsing them into one error state is what makes these tools feel unreliable; a clarifying question rendered as an error teaches readers to distrust the whole tab.

- **Roles are additive, but `GET /users/me` reports one.** ally-be has no `role` column: a role is a `groups` row joined through `user_groups`, and permissions are unioned across all of them. `getMinimalUserInfo` collapses that set to a single `role` by a priority list for legacy clients, which is lossy for anyone holding a platform role next to a tenant one. The response also carries a `roles` array — **gate on `roles`; treat `role` as a legacy hint.** Hoisting a new role into the priority list is usually the wrong fix, because the same field feeds the consumer app: raising a staff-only role above `ADMIN` would silently cost staff-who-are-also-org-admins their org-settings access. (Learned while adding the since-withdrawn `INTERNAL` role.)
- **Cloned roles need their grants cloned in every future migration.** The `...SUPER_ADMIN_PERMISSIONS` spreads in `permissions.constants.ts` are TypeScript only. `group_permissions` rows are written once by migration and never recomputed, so a cloned role like `SUPER_DUPER_ADMIN` silently falls behind unless each new SUPER_ADMIN grant is written to it too. A spec asserting the permission sets are equal is cheap and catches the drift; the alternative is a 403 nobody can explain. Role/permission lookups also sit behind a 30-minute Redis cache that raw SQL migrations cannot bust.
- **Deciding "which deployment am I?" from one build-time value beats a second flag.** When the admin console briefly shipped to two mount points (its own origin, and `/admin` on the consumer app), everything — asset URLs, router basename, non-router redirects, the `<base>` tag, the surface check — derived from the Vite `base`, so the mount point and the app's belief about the mount point could not disagree. A separate `VITE_IS_EMBEDDED`-style flag is exactly the thing that gets set wrong in one environment. The second surface has since been withdrawn; the rule stands for the next one.
- **Module-load-time work in a shared module breaks unrelated test suites.** Calling a constants helper at import time in `api/auth.ts` took out nine admin test files that mock `@constants` wholesale — they had no reason to know about the new export. Resolving the same value lazily inside the function that needs it keeps the import side-effect-free. Prefer it for anything read from a barrel.
