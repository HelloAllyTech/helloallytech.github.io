---
title: Context
tags: [meta, context, state]
summary: Active session meta-context, handoffs, and current operational states.
---

# Meta Context

This file tracks active session state, progress, and agent handoffs between task iterations.

## Current State (2026-07-11)

**Bootstrap + full migration complete.** The wiki has been initialized and populated from the Ally workspace:

- LLMWiki engine vendored into `llmwiki/`; Jekyll scaffolding removed; `.nojekyll` added; `CNAME` (tech.helloally.ai) retained.
- Existing developer-hub content (former `index.md`, `get-started.md`, `tech-stack.md`, `CONTRIBUTING.md`) migrated into `platform/` and `contributing/` pages.
- Six per-repo documentation pages generated under `repos/` and cross-linked.
- Workspace `CLAUDE.md` and `AGENTS.md` knowledge folded into `platform/overview.md`, `platform/architecture.md`, and `platform/agent-guide.md`.
- Whole wiki sanitized: no secrets, IPs, internal hostnames, or cloud regions.
- Each code repo's `AGENTS.md`/`CLAUDE.md` updated to point at this wiki.

## Reprocess (2026-07-11, later)
- `infra` landed its cross-platform dev-script work (`_os.sh` + OS-aware `docker-setup.sh`/`dev_env.sh`/`colima*.sh`); `repos/infra.md` updated from "uncommitted" to committed.
- `ally-mobile` was brought up to date (+27 commits: Scribe voice-dictation notes, review read/unread management, simulation pause/resume, i18n parity, first-login complete-profile gate); `repos/ally-mobile.md` regenerated.
- Generated drift discarded in `ally-mobile` (lockfiles) and `ally-web` (`next-env.d.ts`).

## Open Follow-ups
- No custom project skills existed to migrate (only `settings.local.json` files); `skills/` holds just the example skill.
- `raw/` is empty — drop source documents there for future ingestion.
- Re-run a lint pass when repos change significantly (see `agents.md` Lint Flow).
