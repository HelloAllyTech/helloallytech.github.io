---
title: ally-ai-learn — Voice Training Agent
tags: [repo, ai, livekit, langgraph, voice, python]
summary: A LiveKit-based voice AI agent (FastAPI + LangGraph) that simulates mental-health client conversations, detecting counseling skills in real time, scoring them, and publishing events to ally-be via AWS SQS.
---

# ally-ai-learn — Voice Training Agent

## Purpose

ally-ai-learn (internal package name `ally-learn-core`) is the voice-based training agent for the Ally platform. It simulates realistic client conversations for mental-health counselor training using LiveKit voice agents, then:

- Simulates a configurable client persona in a live voice session.
- Detects counseling events in real time — both positive skills and areas for improvement.
- Scores utterances and delivers immediate UI feedback over LiveKit data channels.
- Publishes all events to AWS SQS (`sqs-learn-message-and-event-queue`) for downstream processing in ally-be.
- Supports multiple TTS/STT/LLM providers, selectable per scenario via room metadata.

It runs two LiveKit workers side by side: the v1 runtime (`app/worker.py`) and Roleplay Studio v2 (`app/worker_v2.py`, package `app/roleplay_v2/`), an Actor+Director-driven studio for authoring and running voice roleplays from a versioned Scenario Spec.

## Tech Stack

- **Python 3.12+** (`>=3.12,<3.14`), managed with **Poetry** (`package-mode = false`).
- **FastAPI** `^0.121` + **uvicorn** — health checks and REST API (scenario reports, voice preview).
- **LiveKit** — `livekit` 1.0.23, `livekit-api`, `livekit-agents` 1.3.12 (extras: deepgram, elevenlabs, openai, sarvam, google, turn-detector), plus noise-cancellation and Silero VAD plugins.
- **LangGraph** `^1.0.10` + `langgraph-checkpoint` — stateful conversation pipeline; integrated with LiveKit via `LLMAdapter` (`livekit-plugins-langchain`).
- **LLM**: OpenAI (default, `gpt-4o-mini`) via `langchain-openai`; also Google Gemini (`langchain-google-genai`), and OpenAI-compatible open-source backends (Ollama, vLLM). OpenAI is **always required** for embeddings and branching resolution regardless of the conversation LLM.
- **STT**: Deepgram (default), Google Cloud STT, Sarvam (Indian languages).
- **TTS**: ElevenLabs, Deepgram, Sarvam, Google Cloud TTS (Chirp 3 HD), Hume.
- **Embeddings**: OpenAI (`text-embedding-3-small`) for semantic-similarity event detection.
- **AWS**: `boto3` for SQS; LocalStack for local emulation.
- **Other**: `slack-sdk` (alerting), `pydub`, `numpy`, `stopwordsiso`/`advertools`.

## Architecture & Key Modules

The service is composed of a FastAPI app plus LiveKit agent workers. The core of a live session is a **LangGraph pipeline** driven by the worker.

**Entry points**
- `app/main.py` — FastAPI application: `GET /api/health`, `/api/v1` routers (scenario report, voice preview), CORS/middleware.
- `app/worker.py` — v1 `AgentWorker`; 3-phase session lifecycle (configure → initialize → start).
- `app/worker_v2.py` — Roleplay Studio v2 worker (gated by `ROLEPLAY_V2_ENABLED`, default on).

**LangGraph conversation flow** (`app/core/graph/`)
- `graph_builder.py` — `build_simulation_graph()` (live voice) and `build_report_graph()` (text-only N-turn simulation for reports).
- `state.py` (`SimulationState`), `nodes.py`, `checkpointer.py`, `knowledge_retrieval.py`, `prompt.py`.
- The simulation graph runs event detection, behavior detection, and knowledge retrieval in parallel from start, converges at a sync point, resolves branching instructions, then generates the client response and sends UI feedback in parallel. Client text → TTS → audio.
- `resolve_branching_instruction` and `retrieve_knowledge` run as subgraphs so their internal LLM calls are not streamed to the user (workaround for LiveKit agents issue #2836).

**Real-time event / skill detection** (`app/core/events/`)
- `event_orchestrator/` — registry, scoring (`ScoreEvaluatorMixin`), delivery (`EventSender`).
- `factory.py` — `build_events_from_metadata()` builds `BaseEvent` instances by detection type.
- `event_types/` — 9 detection types: `SENTENCE_SIMILARITY`, `SEMANTIC_SIMILARITY`, `BINARY_CLASSIFIER`, `TIME`, `SCORE`, `COMBINATION`, `HELPER_UTTERANCE_LENGTH`, `HELPER_INTERRUPTED`, `HELPER_PARAPHRASED` (plus a guardrail type for boundary-violation detection).
- Events have **ACTIVE** visibility (UI feedback: emoji, message, score) or **PASSIVE** (analytics-only). Both are sent to SQS; only ACTIVE generate UI feedback.

**Scenario / persona handling** (`app/core/scenario/`)
- `base.py` — `Scenario.create()` parses backend room metadata into typed `Scenario`, `PromptData` (client persona), `ScenarioVoice`/`VoiceConfig`, `LanguageConfig`, `TerminationEvent`.
- `simulation_instructions.py` — `SimulationBehaviorInstruction` (SHOULD_DO / SHOULD_NOT_DO counselor behaviors) and `SimulationStateInstruction` (ordered agent states with score windows).
- `app/core/agent/factory.py` — `parse_room_metadata()`, provider client configuration, agent creation.

**Provider layer** (factory pattern) — `app/tts/`, `app/stt/`, `app/llms/`, each with `base.py`, `factory.py`, and per-provider implementations. Selection is driven by scenario metadata with env-based fallbacks.

**Scoring & reports**
- `app/core/scoring/score_keeper.py` — `ScoreKeeper` tracks behavior-instruction scores.
- `app/core/scenario_report/` — N-turn text simulation and report generation with webhook delivery back to ally-be.

**Prompts** (`app/prompts/`) — `manager.py` (template loading), `resolver.py` (backend override resolution via `promptData.prompts`); role subdirectories for director, counselor, trainee, judge, actor, events, etc.

## Integration Points

- **ally-be → ally-ai-learn (inbound)**: ally-be dispatches LiveKit sessions and calls this service's REST API with an `x-api-key` header. `X_API_KEY` must match ally-be's `AI_LEARN_SERVICE_OUTBOUND_API_KEY`. Scenario configuration (persona, events, voice, STT/LLM) is passed as **LiveKit room metadata**.
- **ally-ai-learn → ally-be (outbound REST)**: uses `CORE_SERVICE_BASE_URL` + `CORE_SERVICE_API_KEY` (must match ally-be's `AI_SERVICE_API_KEY`), e.g. prompt sync (`scripts/sync_prompts.py`, run at deploy) and scenario-report webhook delivery.
- **AWS SQS**: all triggered events are published to `sqs-learn-message-and-event-queue` (`SQS_MESSAGE_EVENT_QUEUE_NAME`) via `app/core/queue/sqs.py` (`SQSQueue`, async through `asyncio.to_thread`). Message schema `CoreEventMessageType` (`message_type: event`, `room_id`, serialized `event_data`, timestamp).
- **Deferred termination**: auto-termination events are held (`EventSender.deferred_termination_event`) until the agent finishes speaking the termination message, then sent to SQS with `autoTerminationStatus=true` in the shutdown callback — so ally-be doesn't close the room prematurely.
- **LiveKit**: WebRTC audio to/from LiveKit rooms; UI event feedback is sent over LiveKit **data channels** (not SQS), schema `CustomUIEventMessageType`.
- **Roleplay Studio v2**: shares the Scenario Spec model with ally-be (owner of the spec, Copilot authoring, rehearsal lifecycle) and ally-web (`ally-admin-dashboard` studio UI); this repo owns the v2 runtime and Actor/Director/Trainee/Judge prompts.

## Local Setup

Prerequisites: Python 3.12+, Poetry, LiveKit credentials, OpenAI API key (always required), AWS creds or LocalStack, and at least one TTS key (Deepgram is the default).

```bash
git clone <repository-url>
cd ally-learn-core
poetry install
cp env_sample .env      # then fill in credentials
```

Required env: `LIVEKIT_API_KEY`, `LIVEKIT_API_SECRET`, `LIVEKIT_URL`, `OPENAI_API_KEY`, `DEEPGRAM_API_KEY`, `AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, `AWS_REGION`, `SQS_MESSAGE_EVENT_QUEUE_NAME`. Also `CORE_SERVICE_BASE_URL`/`CORE_SERVICE_API_KEY` and `X_API_KEY` for ally-be integration.

**Run — two terminals:**
```bash
# Terminal 1 — LiveKit worker
poetry run python -m app.worker dev

# Terminal 2 — FastAPI server
poetry run uvicorn app.main:app --reload --port 8000
```

**Run — startup script** (starts uvicorn + v1 worker + v2 worker in parallel, with supervisor restart for v2):
```bash
./start.sh
```

**Docker:**
```bash
docker build -t ally-learn-core .
docker run -p 8000:8000 --env-file .env ally-learn-core   # runs FastAPI + worker together
```

**LocalStack (emulated SQS):**
```bash
docker-compose up -d       # LocalStack on :4566 (SES, S3, Lambda, CloudWatch Logs, SQS)
```
Set `AWS_ACCESS_KEY_ID=test`, `AWS_SECRET_ACCESS_KEY=test`, `AWS_ENDPOINT_URL=http://localhost:4566`.

**Full dev stack** (app + LocalStack, app exposed on host port `8002` to avoid clashing with other Ally services):
```bash
docker compose -f docker-compose.full.yml up --build
```

**Alongside ally-be / ally-web** (reuses ally-be's LocalStack):
```bash
docker compose -f docker-compose.local.yml up -d
```

**Health check:** `curl http://localhost:8000/api/health`

## Testing & Code Quality

- **pytest** (`^8.0`) with `pytest-asyncio`, `pytest-cov`, `pytest-mock`, `pytest-xdist`. Config in `pytest.ini`; tests under `tests/unit/` and `tests/integration/`. Markers: `unit`, `integration`, `slow`, `external`, `livekit`, `auth`, `database`.
- **`run_tests.sh`** wraps common flows: `./run_tests.sh unit | integration | all | coverage | parallel | smoke | marker <name> | specific <file>`. Coverage target is `--cov-fail-under=80`.
- Direct commands: `poetry run pytest`, `poetry run pytest tests/unit/`, `poetry run pytest --cov=app`.
- **Formatting/linting**: Black (line length 88, `py312`), isort (black profile), flake8 (`.flake8`). Enforced via `.pre-commit-config.yaml`.

## Key Documentation

- `README.md` — full overview, provider configuration, scenario metadata schema, troubleshooting.
- `docs/readme.md` — documentation index and reading paths.
- `docs/architecture.md` — system diagram, component inventory, factory pattern, design decisions.
- `docs/getting-started.md` — local setup, Docker, LocalStack, tests.
- `docs/LOCAL_DEV.md` — running alongside ally-be and ally-web.
- `docs/01-worker-lifecycle.md` — `AgentWorker` 3-phase lifecycle and termination.
- `docs/02-graph-pipeline.md` — `SimulationState`, the 6-node graph, parallel flow, report graph.
- `docs/03-event-system.md` — orchestrator, registry, scoring, eligibility.
- `docs/04-event-types-reference.md` — reference card for all 9 detection types.
- `docs/05-tts-providers.md`, `docs/06-stt-providers.md`, `docs/07-llm-providers.md` — provider factories and per-provider config.
- `docs/09-scenario-engine.md` — Pydantic scenario models and metadata parsing pipeline.
- `docs/10-branching-instructions.md` — conditional conversation flow, variable substitution.
- `docs/11-queue-messaging.md` — SQS event delivery, message schemas, deferred termination, LocalStack.
- `docs/12-prompts-system.md` — template loading and backend override resolution.
- `docs/13-event-termination.md` — auto-termination configuration and flow.
- `docs/events-off-gate-design.md` — event gating design notes.
- `ROLEPLAY_STUDIO_V2.md` — cross-repo reference for the Actor/Director roleplay studio and Scenario Spec.
- `CONTRIBUTING.md`, `tests/README.md`, `.github/RELEASE_GUIDE.md` — contribution, test, and release guides.

---

*Part of the [Ally Platform](../platform/overview.md). See also: [Architecture](../platform/architecture.md), [ally-be](ally-be.md), [ally-ai](ally-ai.md).*
