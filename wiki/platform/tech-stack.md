---
title: Tech Stack
tags: [platform, tech-stack, tooling]
summary: The polyglot technology stack used across Ally's frontend, mobile, backend, AI, and infrastructure layers.
---

# Tech Stack

Ally runs on a modern, polyglot stack focused on reliability, security, and speed of iteration: TypeScript-first frontends, Python AI services, a NestJS backend, all orchestrated through Docker Compose for consistent dev-to-prod environments.

## Frontend
- React 18, Next.js (landing) and Vite 5 (dashboards) power the web surfaces.
- TypeScript everywhere, with Tailwind CSS and Redux Toolkit for stateful experiences.
- Nx monorepo tooling; shared component library (`ui-shared`).
- See [ally-web](../repos/ally-web.md).

## Mobile
- React Native 0.79 with TypeScript for iOS and Android.
- LiveKit-native modules for secure audio streaming and transcription.
- Redux Toolkit + RTK Query, React Navigation, Firebase Crashlytics, Socket.IO.
- See [ally-mobile](../repos/ally-mobile.md).

## Backend & APIs
- NestJS (TypeScript, Node 24) for REST, WebSocket, and event-driven services.
- PostgreSQL (TypeORM), Redis, and Socket.IO for transactional and realtime workloads.
- AWS SQS, S3, and SES integrations orchestrated through domain-driven service modules.
- JWT + OTP + Google OAuth authentication; RBAC and multi-tenant isolation.
- See [ally-be](../repos/ally-be.md).

## AI Services
- Python 3.12 services on FastAPI.
- LangChain (and LangGraph in the training agent) across OpenAI and other models for counseling insights.
- Weaviate vector search and Deepgram speech intelligence underpin semantic analysis.
- Multi-provider TTS (Deepgram, ElevenLabs, Google, Hume, Sarvam) and STT (Deepgram, Google, Sarvam, OpenAI Whisper).
- See [ally-ai](../repos/ally-ai.md) and [ally-ai-learn](../repos/ally-ai-learn.md).

## Data & Realtime
- LiveKit for low-latency voice and presence channels.
- Redis streams and PostgreSQL analytics layers for longitudinal session data.
- Event pipelines translating transcripts into searchable, privacy-aware knowledge.

## Infrastructure & DevOps
- Ansible + Terraform-managed environments; Incus containers on Hetzner baremetal.
- AWS ECR (images), SQS (queues), S3 (assets/env), CloudWatch (HIPAA audit logs).
- Caddy reverse proxy with automatic HTTPS; Wireguard VPN mesh.
- Colima as a lightweight Docker runtime on macOS.
- Public GitHub Actions workflows covering lint, test, security, build, and image publish steps.
- See [infra](../repos/infra.md).

## Language & Runtime Versions

| Repo | Runtime |
|------|---------|
| ally-be | Node.js 24 |
| ally-web | Node.js 22 |
| ally-mobile | Node.js ≥ 18, React Native 0.79 |
| ally-ai | Python 3.12+ |
| ally-ai-learn | Python 3.12+ |
| infra | Ansible, Terraform, Bash |

## Code Quality Conventions

- **Python repos** (ally-ai, ally-ai-learn): Black (88 chars), isort, flake8, pytest + asyncio, pre-commit hooks.
- **TypeScript repos** (ally-be, ally-web, ally-mobile): Prettier, ESLint, Jest (ally-be) / Vitest (ally-web), Husky + lint-staged.

## Progressive Docker Compose Rollout

Each service family is being migrated onto curated Docker Compose bundles so local and production environments stay in lockstep. New repositories launch with Compose files that mirror production images and networking; legacy stacks are refactored in phases (infrastructure utilities and backend APIs first, then AI workloads and frontends). This keeps onboarding friction low (`docker compose up` per bundle), lets teams adopt Compose incrementally, and gives CI/CD pipelines the same topology developers run locally.

---

*See also: [Platform Overview](overview.md), [Architecture](architecture.md).*
