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
