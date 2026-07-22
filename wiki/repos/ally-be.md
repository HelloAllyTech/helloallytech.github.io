---
title: ally-be — Core Backend
tags: [repo, backend, nestjs, api]
summary: ally-be is the multi-tenant NestJS backend that owns Ally's primary PostgreSQL database and orchestrates auth, real-time voice sessions, the audio/transcription pipeline, AI-driven feedback, and analytics for the counselor-training platform.
---

# ally-be — Core Backend

## Purpose

ally-be is the core API and event-processing service of the Ally platform — a multi-tenant, HIPAA-compliant backend for training mental health counselors. It owns the platform's primary PostgreSQL database and serves the web and mobile clients. Its responsibilities include:

- **Scenario-based learning** — simulated client conversations, learning pathways, and session management.
- **Real-time communication** — WebSocket gateways plus LiveKit voice/video rooms.
- **Audio & transcription pipeline** — async processing via AWS SQS and S3 with dead-letter-queue handling.
- **AI-driven insights** — call summaries, live nudges, and session reports through LLM integration.
- **Conversational guardrails** — real-time boundary-violation detection with multi-language support.
- **Multi-tenant access control** — role-based permissions, groups, and tenant isolation.
- **Gamification** — badges and community leaderboards.
- **AWS integration** — S3, SQS, SES, and CloudWatch for storage, messaging, email, and audit logging.
- **Versioned REST API** — documented via Swagger/OpenAPI.

It is architected as a modular NestJS monolith. Schema changes ship as TypeORM migrations (`synchronize: false`).

## Tech Stack

Sourced from `package.json` and `README.md`.

| Component | Technology |
|-----------|------------|
| Runtime | Node.js v24 |
| Language | TypeScript (~5.9) |
| Framework | NestJS 11 (`@nestjs/*`) |
| Database | PostgreSQL (`pg`) via TypeORM (`@nestjs/typeorm`, `typeorm` 0.3) |
| Caching / pub-sub | Redis (`ioredis`, `@nest-lab/throttler-storage-redis`) |
| Real-time | Socket.IO (`@nestjs/platform-socket.io`, `@nestjs/websockets`), LiveKit (`livekit-server-sdk`) |
| Auth | JWT (`@nestjs/jwt`), Passport (`passport-jwt`, `passport-local`), `google-auth-library`, `bcrypt`, `jose`, `jsonwebtoken` |
| AI / LLM | OpenAI (`openai`), Anthropic (`@anthropic-ai/sdk`) |
| STT / TTS | Deepgram (`@deepgram/sdk`), ElevenLabs (`@elevenlabs/elevenlabs-js`), Google Cloud (`@google-cloud/text-to-speech`, `@google-cloud/translate`), Sarvam (`sarvamai`), Hume (`hume`) |
| AWS | `@aws-sdk/client-s3`, `client-sqs`, `client-ses`, `client-cloudwatch-logs`, `s3-request-presigner` |
| API docs | Swagger/OpenAPI (`@nestjs/swagger`) |
| Scheduling | `@nestjs/schedule` |
| Rate limiting | `@nestjs/throttler` |
| Health checks | `@nestjs/terminus` |
| Validation | `class-validator`, `class-transformer`, `joi` |
| Logging | Winston (`winston`, `winston-cloudwatch`) |
| Security | `helmet`, `sanitize-html` |
| Media | `fluent-ffmpeg` |
| Testing | Jest (`jest`, `ts-jest`), Supertest |
| Lint / format | ESLint 9, Prettier, Husky |

## Architecture & Key Modules

Feature modules live under `src/<domain>/` (controllers, services, DTOs, and `entity/*.entity.ts`). Notable modules:

**AI & audio**
- `ai/`, `ai-chat/` — LLM integration (OpenAI streaming), session summarization, event analysis, provider abstraction.
- `audio/`, `audio-ingest/` — audio storage plus SQS-based async transcription pipeline with DLQ handling.
- `conversational-guardrails/` — real-time boundary detection (binary classification) with multi-language support and automatic agent redirect.
- `prompt/`, `prompts/` — versioned system prompts with dashboard sync and per-tenant runtime override resolution.
- `voice-preview/` — TTS voice preview across ElevenLabs, Deepgram, Sarvam, Google, and Hume.

**Learning & sessions**
- `learn/`, `scenario-path/` — scenario engine, learning pathways, session management.
- `roleplay-studio/` — Roleplay Studio v2: versioned Scenario Spec + copilot authoring (the copilot interviews the trainer and builds the spec via tools), session dispatch to the v2 runtime, and Director telemetry persistence. The copilot verifies the draft with `compile_spec`; the trainer then tests live or publishes. See the repo's `ROLEPLAY_STUDIO_V2.md` for the cross-repo reference.
- `scenario-character/` — client-persona (NPC) definitions.
- `scenario-report/`, `scenario-session-review/`, `scribe-session-review/` — reporting and threaded review/feedback.
- `session-event/`, `case/`, `reference-document/` — event tracking, case management, supporting materials.

**Platform & infrastructure**
- `auth/`, `authorization/` — authentication plus RBAC guards and permission decorators.
- `tenant/`, `user/`, `settings/` — multi-tenant management, users, config.
- `livekit/` — room lifecycle, participant tracking, agent dispatch, webhook handling.
- `aws/`, `queue/`, `message-broker/`, `redis/` — AWS wrappers, SQS setup, Redis pub/sub, Redis client.
- `audit/` — HIPAA-compliant CloudWatch audit logging.
- `analytics/` — Metabase integration and tenant-specific dashboards.
- `badge/`, `community/` — gamification and leaderboards.
- `language/`, `dynamic-i18n/` — translation service and internationalization.
- `scheduler/`, `health/`, `rate-limit/`, `notification/`, `database/` — background jobs, health checks, throttling, SES notifications, TypeORM data source/migrations/seeds.

**WebSocket gateways**
- `/microphone-chat` — live audio chat with message streaming.
- `/audio-ingest` — cloud-telephony integration.
- `/scenario-report` — real-time scenario reporting.

**Auth model** — the REST API is versioned (`/api/v1/...`). Supported methods: JWT (access/refresh flow), email OTP (v2), Google OAuth, Magic Link, and `X-API-Key` for service-to-service calls. Multi-tenancy is enforced at the entity level, with fine-grained RBAC via groups and permission guards. See `DATA_SCHEMA.md` for the full store map (105 TypeORM tables plus Weaviate, Redis, SQS, S3, LiveKit).

## Integration Points

- **ally-ai** — reached at `AI_SERVICE_API_URL`; outbound calls authenticate with `AI_SERVICE_OUTBOUND_API_KEY`. ally-ai owns the Weaviate vector DB (`Conversation`, `ReferenceDocument` collections). SQS and LiveKit bridge the two services.
- **ally-ai-learn** — reached at `AI_LEARN_SERVICE_API_URL` with `AI_LEARN_SERVICE_OUTBOUND_API_KEY`.
- **LiveKit** — via `livekit-server-sdk` using `LIVEKIT_URL` / `LIVEKIT_API_KEY` / `LIVEKIT_API_SECRET`; handles room creation, participant lifecycle, agent dispatch (`LIVEKIT_AGENT_NAME`), and webhook events.
- **AWS SQS queues** (from `.env.example`):
  - `sqs-ai-transcription-request-queue` (+ `-dlq`) — transcription requests.
  - `sqs-audio-file-retry-queue` (+ `-dlq`) — audio file retries.
  - `sqs-learn-message-and-event-queue` — learning messages and events from ally-ai-learn.
- **ally-web / ally-mobile** — consume the versioned REST API plus the Socket.IO gateways above. CORS origins are configured via `ALLOWED_ORIGINS` (defaults include the landing page :3000, helpline dashboard :8080, admin dashboard :8081).
- **Redis message broker** — pub/sub for inter-service event delivery (`message-broker/`).
- **Metabase** — analytics dashboards via `METABASE_URL` / `METABASE_API_KEY`.
- **AWS SES** — transactional email.

## Local Setup

**Prerequisites:** Node.js v24, npm, Docker + Docker Compose (v2). Accounts/keys for LiveKit, OpenAI, and Deepgram; AWS is optional (LocalStack covers S3/SQS/SES/CloudWatch locally).

```bash
# 1. Configure environment
cp docker.env.example docker.env
cp .env.example .env
# edit both with your credentials

# 2. Start infrastructure (Postgres, Redis, LocalStack, SQS)
docker-compose up

# 3. Install dependencies
npm install

# 4. Run migrations
npm run migration:run

# 5. Seed the database (idempotent)
npm run seed

# 6. Start the app in dev mode
npm run start:dev
```

Once running:
- Swagger UI — `http://localhost:8001/api-docs`
- Health check — `http://localhost:8001/api/health`

Postgres is exposed on host port **5477** (mapped from container 5432). Production build/run: `npm run build` then `npm run start:prod`.

**Database & seeding notes**
- Migration scripts: `migration:generate`, `migration:create`, `migration:run`, `migration:revert`, `migration:show` (all wrap `typeorm` against `src/database/data-source.ts`). Permission-migration generator: `npm run migration:permissions`.
- Seeds live in `src/database/seeds/` (fixtures in `fixtures.ts`). Seeding inserts 1 tenant, 4 test users (SUPER_ADMIN, ADMIN, LEARNER+COUNSELOR, MULTI_TENANT_ADMIN), scenario voices, sample scenarios/pathway/case/badges, and scenario sessions. Seeded test users share a default local password and a fixed local OTP (see the repo's seed fixtures — local development only).
- `npm run seed:reset -- --confirm && npm run seed` truncates seeded tables and re-seeds; refuses to run without `--confirm` and refuses entirely under `NODE_ENV=production`.
- Set `AUTO_SEED=true` to seed automatically after migrations on container start.

## Testing & Code Quality

**Testing** — Jest for unit tests (`*.spec.ts` co-located with source) and e2e tests (`test/**/*.e2e-spec.ts`).

```bash
npm run test            # unit tests
npm run test:watch      # watch mode
npm run test:cov        # with coverage
npm run test:e2e        # end-to-end
npm run test:docker     # run tests inside Docker (test-docker.sh)
```

Docker-based tests use `compose.test.yaml` (spins up postgres-test on PG 14, redis-test, localstack-test, sqs-setup-test, and the test/coverage/e2e runners) under `NODE_ENV=test`, isolated on a `test-network` with tmpfs storage. See `TESTING.md` and `DOCKER_TESTING_SETUP.md`.

**Lint & format**

```bash
npm run lint       # ESLint
npm run lint:fix   # ESLint autofix
npm run format     # Prettier
```

Standards: ESLint + Prettier enforced, Husky git hooks (`prepare: husky`), `.pre-commit-config.yaml`, and `.gitleaks.toml` for secret scanning. Contribution conventions (branch naming `<type>/<desc>`, conventional commits, PR format) are in `CONTRIBUTING.md`.

## Key Documentation

- `README.md` — full service overview, architecture, directory map, setup, API endpoints, features, observability, troubleshooting.
- `CLAUDE.md` — repo guidance; points to the data-schema reference and describes the module layout.
- `DATA_SCHEMA.md` — cross-store data map: 105 PostgreSQL/TypeORM tables by domain, Weaviate collections, Redis/SQS/S3/LiveKit contents, and a "where do I find…?" index. Read before building features touching stored data.
- `CONTRIBUTING.md` — Git branch/commit/PR conventions and code-review process.
- `TESTING.md` — Docker-based testing guide (services, env, writing unit/e2e tests, CI example).
- `DOCKER_TESTING_SETUP.md` — Docker testing infrastructure setup details.
- `docs/README.md` — index of the `docs/` folder.
- `docs/prompts-folder.md` — `src/prompts/` prompt folder conventions: naming, structure, optional `.meta.json`, adding prompts.
- `docs/prompts-api.md` — Prompts API: dashboard exposure, sync endpoint, auth, runtime resolution.
- `docs/dynamic-i18n.md` — dynamic internationalization documentation.
- `src/prompts/README.md`, `src/prompt/README.md` — module-level prompt documentation.

---

*Part of the [Ally Platform](../platform/overview.md). See also: [Architecture](../platform/architecture.md), [ally-ai](ally-ai.md), [ally-ai-learn](ally-ai-learn.md).*
