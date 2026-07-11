---
title: Cross-Repo Agent Guide
tags: [platform, agents, conventions, reference]
summary: Quick-reference for AI assistants and developers working across Ally repos — per-repo entry points, cross-repo conventions, common tasks, and gotchas.
---

# Cross-Repo Agent Guide

This page helps AI agents (and developers) quickly orient in the Ally workspace and work effectively across repos. It is migrated from the workspace `AGENTS.md`. For per-repo depth, follow the links to each [repo page](../index.md#repositories).

## Repo Quick Reference

### [ally-be](../repos/ally-be.md) — Core Backend
- **Stack**: NestJS 11 / TypeScript 5.9 / Node 24 / PostgreSQL 14 / Redis 7 / TypeORM
- **Entry**: `src/main.ts` → `src/app.module.ts` (43 modules)
- **Key modules**: auth, learn, livekit, ai, chat, audio-ingest, conversational-guardrails, badge, community, prompt, tenant, user, authorization, scenario-report
- **DB migrations**: `src/database/migrations/` (211+ files, TypeORM); seeds in `src/database/seeds/`
- **WebSocket gateways**: `microphone-chat.gateway.ts`, `cloud-telephony.gateway.ts`, `scenario-report.gateway.ts`
- **API docs**: Swagger at `/api-docs`
- **Key patterns**: RTK Query-style API services, EventEmitter for async, Redis pub/sub, RBAC guards, tenant isolation decorators

### [ally-ai](../repos/ally-ai.md) — AI Service
- **Stack**: FastAPI / Python 3.12 / Poetry / Weaviate / LangChain / OpenAI
- **Entry**: `app/main.py`
- **Key services**: `app/core/conversations/`, `summaries/`, `text_generations/`, `reference_documents/`, `transcriptions/`, `queue/`
- **Prompts**: `app/prompts/` (file-based, dynamically loaded)
- **Vector DB**: Weaviate collections in `app/core/vector_db/constants.py`; schema migrations in `app/migrations/`
- **Background worker**: SQS transcription worker in `app/core/queue/transcription_request_sqs_worker.py`
- **Key patterns**: Abstract base classes for services, FastAPI dependency injection, Pydantic settings, HIPAA audit logging

### [ally-ai-learn](../repos/ally-ai-learn.md) — Training Agent
- **Stack**: FastAPI + LiveKit Agents / Python 3.12 / Poetry / LangGraph / multi-provider TTS/STT/LLM
- **Entry**: `app/main.py` (FastAPI), `app/worker.py` (LiveKit agent)
- **Graph pipeline**: `app/core/graph/` (LangGraph: process_events → resolve_branching → generate_response/detect_behaviors → apply_voice → send_feedback)
- **Event system**: `app/core/events/` (9 event types incl. SENTENCE_SIMILARITY, SEMANTIC_SIMILARITY, BINARY_CLASSIFIER, TIME, SCORE, COMBINATION, HELPER_*)
- **Provider factories**: `app/tts/factory.py`, `app/stt/factory.py`, `app/llms/factory.py`
- **Prompts**: `app/prompts/` (system, branching, prosody, events); extensive `docs/` (01–13 numbered guides)
- **Key patterns**: Factory pattern for providers, LiveKit agent lifecycle (configure/initialize/start), event orchestration with eligibility constraints

### [ally-web](../repos/ally-web.md) — Web Frontend Monorepo
- **Stack**: Nx 22 / React 18 / TypeScript / Vite (dashboards) / Next.js (landing)
- **Apps**: `apps/ally-web/` (landing :3000), `apps/ally-helpline-dashboard/` (:8080), `apps/ally-admin-dashboard/` (:8081)
- **Shared lib**: `libs/ui-shared/`
- **State**: Redux Toolkit + RTK Query + Redux Persist; real-time via Socket.IO + LiveKit components
- **Key patterns**: RTK Query for API calls, path aliases (`@api`, `@components`), permission-based routing, i18next

### [ally-mobile](../repos/ally-mobile.md) — Mobile App
- **Stack**: React Native 0.79 / TypeScript 5 / React 19 / Redux Toolkit + RTK Query
- **Entry**: `index.js` → `src/App.tsx`; navigation via React Navigation 7 (native-stack + bottom-tabs + drawer)
- **Services**: RTK Query in `src/services/` (30+ API files); 45+ custom hooks in `src/hooks/`
- **Real-time**: Socket.IO + LiveKit React Native; `react-native-live-audio-stream` for recording
- **Key patterns**: Dual listening modes (microphone + conference), AsyncStorage for tokens, Firebase Crashlytics

### [infra](../repos/infra.md) — Infrastructure
- **Stack**: Ansible 13 + Terraform + Docker Compose
- **Playbooks**: `ansible/` — `baremetal.yml`, `ally_core.yml`, `ally_ai.yml`, `ally_learn.yml`, `ally_web.yml`, `ci.yml`
- **Scripts**: `scripts/bootstrap.sh` (clone repos), `scripts/dev_env.sh` (start dev), `scripts/colima.sh`
- **Secrets**: Ansible Vault; **Key patterns**: Incus containers on Hetzner, Consul discovery, Caddy proxy, Wireguard VPN

## Cross-Repo Conventions

### Branch Naming
```
feature/<ticket-id>-short-description
fix/<ticket-id>-short-description
chore/<ticket-id>-short-description
```
(Individual repos' `CONTRIBUTING.md` may specify `feat/…`; check the target repo.)

### API Versioning
- All REST APIs use the `/api/v1/` prefix.
- WebSocket namespaces: `/microphone-chat`, `/cloud-telephony-chat`, `/scenario-report`.

### Environment Files
- Python repos: `env_sample` → copy to `.env`.
- TypeScript repos: `.env.example` or `docker.env.example`.
- Never commit `.env` files.

## Common Tasks

**Adding a new API endpoint** — ally-be: controller + service + DTOs in the relevant `src/` module; ally-web/mobile: add an RTK Query endpoint in `src/services/`.

**Adding a new AI capability** — ally-ai: prompt templates in `app/prompts/`, service logic in `app/core/`, endpoint in `app/api/v1/endpoints/`; ally-be: integration call in `src/ai/`.

**Adding a new event type (ally-ai-learn)** — create the event class in `app/core/events/event_types/`, register in `app/core/events/factory.py`, add prompts in `app/prompts/events/` (see `docs/04-event-types-reference.md`).

**Adding a TTS/STT/LLM provider (ally-ai-learn)** — create the provider class in `app/tts|stt|llms/`, extend the corresponding `factory.py` (see `docs/05-07`).

**Database changes (ally-be)** — `npm run migration:generate --name=DescriptiveName` then `npm run migration:run`.

**Weaviate schema changes (ally-ai)** — add a migration in `app/migrations/` following the `NNN-description.py` pattern.

## Gotchas & Important Notes

- **Lima/Colima VM**: local dev commonly runs Docker inside a Lima/Colima VM on macOS; commands execute within the VM context.
- **HIPAA**: never log PII/PHI outside designated audit loggers. Use `phi_logger` in ally-ai.
- **Multi-tenant**: all ally-be queries must respect tenant isolation; entities have `tenantId` columns.
- **Prompt sync**: prompts in ally-ai can be overridden at runtime via ally-be's prompt management; `scripts/sync_prompts.py` pushes defaults.
- **LiveKit rooms**: ally-ai-learn agents join rooms created by ally-be; room metadata carries scenario configuration.
- **SQS in dev**: use LocalStack (auto-started by docker-compose). Queue URLs differ between local and production.
- **Node versions**: ally-be requires Node 24; ally-web requires Node 22. **Python**: both AI repos require 3.12+.

---

*See also: [Platform Overview](overview.md), [Architecture](architecture.md), [Contributing Guide](../contributing/guide.md).*
