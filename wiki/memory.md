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

### About the Ally platform
- Ally is a HIPAA-compliant platform for training mental-health counselors: AI simulations, real-time session analysis, and peer review. 7 repos (see [Platform Overview](platform/overview.md)).
- Core data flow: web/mobile → **ally-be** (NestJS) → **ally-ai** (analysis/transcription) + **ally-ai-learn** (LiveKit voice training) via REST/SQS/LiveKit. Details in [Architecture](platform/architecture.md).
- Service-to-service auth uses `X-API-Key`; client auth uses JWT/OTP/Google OAuth; all REST APIs are `/api/v1/`.
- SQS queues bridge services (transcription request/response, learn message-and-event, audio upload), each with a DLQ; LocalStack emulates them locally.
- Runtimes: ally-be Node 24, ally-web Node 22, ally-mobile Node ≥18, ally-ai / ally-ai-learn Python 3.12+.
- Each code repo's `AGENTS.md`/`CLAUDE.md` points here as the source of truth. Update the wiki when architecture, workflows, or environment setup change.
