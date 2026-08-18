---
title: ally-ai — AI Copilot Service
tags: [repo, ai, fastapi, weaviate, python]
summary: ally-ai (internally "Lifeline AI") is the FastAPI AI copilot service that provides conversation analysis, summaries, transcription, drift/language-quality scoring, and Weaviate-backed semantic search for the Ally counselor-training platform.
last_reconciled: 2026-07-30
---

# ally-ai — AI Copilot Service

## Purpose

ally-ai is the AI service that acts as a copilot for mental health counselors. It delivers intelligent insights over counseling conversations: conversation analysis and nudges, session summaries, audio transcription, response-drift and language-quality scoring, and semantic search over reference documents. The service is HIPAA-conscious (PHI logging, optional CloudWatch audit logging) and is built with FastAPI, managed with Poetry, and backed by the Weaviate vector database. The repository is internally named "Lifeline AI".

## Tech Stack

From `pyproject.toml` (package `lifeline-ai`, MIT license):

- **Python** `^3.12`
- **Web framework**: FastAPI `^0.115.8` + Uvicorn `^0.34.0`, `pydantic-settings` for config
- **LLM / orchestration**: `langchain-openai` `^1.1.12`, `langchain-google-genai` `^4.2.0`, `google-genai` `^1.0.0`
- **Vector DB**: `weaviate-client` `^4.11.0`
- **Speech-to-text**: `deepgram-sdk` `^4.7.0`, `sarvamai` `^0.1.0`, plus OpenAI transcription (providers: `openai`, `deepgram`, `sarvam`)
- **Audio**: `ffmpeg-python`
- **NLP / analysis**: `numpy`, `scikit-learn`, `vadersentiment` (sentiment), `rapidfuzz` (fuzzy matching)
- **AWS**: `boto3` (SQS + S3)
- **Alerts / HTTP**: `slack-sdk`, `httpx`
- **Observability**: LangSmith tracing (env-configured)
- **Dev tooling**: `black` (line-length 88, py312), `isort` (black profile), `flake8`, `pre-commit`, `pytest` + `pytest-asyncio` / `pytest-mock` / `pytest-cov` / `pytest-xdist`

## Architecture & Key Modules

The app is a FastAPI application (`app/main.py`) whose lifespan initializes the Weaviate client, the ally-core HTTP client, and OpenAI clients. Routers are mounted under an API prefix and a v1 prefix (`app/api/v1/api.py`).

**HTTP API endpoints** (`app/api/v1/endpoints/`):

- `/conversation` — conversation analysis
- `/summary` — session summaries
- `/reference-documents` — reference document semantic search / management
- `/drift` — response-drift scoring
- `/language-quality` — language quality scoring
- `/feedback-groundedness` — judges each post-session feedback claim against the transcript (`supported` / `unsupported` / `contradicted` / `misattributed`), plus whether a quoted span is actually in the transcript. Claims arrive already split by ally-be, one verdict per claim: a `contradicted` improvement is the harmful case — the learner marked down for work the transcript shows them doing — and separating it from an unearned compliment is only possible per claim.
- `/round-trip-wer` — round-trip word error rate for transcription quality
- `/analytics-agent` — two stateless transforms behind the admin Analytics Agent tab: `/plan` (question + schema catalogue → one read-only SELECT, or a clarifying question) and `/answer` (result rows → prose, caveats and a chart specification). No database access; ally-be runs the query. See [Analytics Agent](../platform/analytics-agent.md).
- `/knowledge-chunks` — write and read side of the `KnowledgeChunk` collection for the WhatsApp Q&A bot: `bulk-upsert` (per-object success/failure so ally-be can retry only what failed), `search`, `document/{id}` (delete by document), `ids` (paged, for reconciliation), and `{chunk_id}`.
- `/knowledge-agent` — the bot's answering loop. `/answer` returns one of three intents — `answer` (grounded, with citations), `decline` (the corpus does not cover it) or `clarify` (too vague to retrieve against) — **all as HTTP 200**, because a decline is a correct result and only a genuine failure should push ally-be onto its fallback path. `/crisis-check` is separate rather than a stage inside `/answer`, so ally-be can run the two concurrently and the safety net costs no latency on an ordinary question.

**Core business logic** (`app/core/`):

- `conversations/` — `conversation_service.py`, conversation analysis
- `summaries/` — `summary_service.py`, session summarization
- `transcriptions/` — audio transcription services with pluggable providers (`DeepgramTranscriptionService`, `OpenAITranscriptionService`, `SarvamTranscriptionService`)
- `embeddings/` — OpenAI embedding client/service for vectorization
- `vector_db/` — Weaviate client (`weaviate_client.py`) and collection helpers
- `reference_documents/` — reference document retrieval (distance-threshold based)
- `knowledge_base/` — `KnowledgeChunk` read/write service for the WhatsApp bot's corpus
- `knowledge_agent/` — the bot's agent: detect language → translate to English → embed → retrieve → answer, decline or clarify, plus the crisis classifier. Citations are returned as integers indexing the numbered passages and validated in code, out-of-range values dropped: a model asked to echo a UUID will eventually invent a plausible one, and a fabricated id cannot be detected whereas an out-of-range integer can.
- `llm/` — `dispatch.py`, one `generate_structured` call across providers. Gemini uses `response_schema`; Anthropic has no equivalent, so structured output goes through a single forced tool call.
- `drift/`, `language_quality/`, `round_trip/`, `feedback_groundedness/` — LLM-judge modules (each with `judge.py`, `prompt.py`, `schemas.py`; `round_trip` also has `wer.py`). Every one of them emits ONLY labels, booleans and counts — never a score, rate or rating. Rates, severity weights and correlations are computed by ally-be in SQL at read time, so re-weighting a metric never means re-judging the corpus.
- `analytics_agent/` — the analytics agent's planner and narrator (`agent.py`, `prompt.py`, `schemas.py`). The schema catalogue arrives on the request from ally-be rather than being defined here, so the tables the model is shown and the tables the guard permits cannot drift apart.
- `text_generations/` — OpenAI text-generation client/service and structured-output models
- `storage/` — `s3_service.py` for S3 access
- `queue/` — SQS clients, message models, processors, and the transcription request worker (see Integration Points)
- `ally_core/` — HTTP client/service for calling the ally-be backend
- `llm_usage/`, `execution_manager.py`, `drift`, `round_trip` — supporting services
- `phi_events.py`, `phi_logger.py` — PHI-aware event logging for HIPAA handling

**Prompts** (`app/prompts/`): file-based templates organized by concern — `analysis`, `audio`, `notes`, `nudge`, `scenario`, `simulation`, `summary`, `tags`, `user`, `shared` — with a `manager.py` and `resolver.py`. Prompts are synced to the backend (see Makefile / Local Setup).

**Schemas** (`app/schemas/`): Pydantic models — `common.py` (e.g. `ChatMessage`), `conversation.py`, `summary.py`, `reference_document.py`, `health.py`.

## Integration Points

**ally-be (ally-core) via REST** — `app/core/ally_core/` wraps an `httpx.AsyncClient` pointed at `ALLY_CORE__ENDPOINT` (default `http://localhost:8001`), authenticated with `x-api-key`. Observed calls include:
- `GET /api/v1/prompts/by-codes` — fetch current prompt text by code (used by the drift judge to source its rubric)
- `process_transcript` — pushes processed transcript results back to the backend

**SQS queues (AWS, LocalStack for local)** — configured via `QUEUE__*` env vars:
- `QUEUE__TRANSCRIPTION_RESULTS_QUEUE_URL` (`TRANSCRIPTION_RESULTS_QUEUE`)
- `QUEUE__TRANSCRIBE_AND_SUMMARIZE_RESPONSE_QUEUE_URL` (`TRANSCRIBE_AND_SUMMARIZE_RESPONSE_QUEUE`)
- `QUEUE__TRANSCRIBE_AND_SUMMARIZE_RESULTS_QUEUE_URL`

A dedicated worker `app/core/queue/transcription_request_sqs_worker.py` consumes transcription requests and runs the transcribe/summarize pipeline (`transcription_request_handler.py`, message models in `message_models.py`). This aligns with the platform's `sqs-ai-transcription-request-queue` / `sqs-ai-transcription-response-queue` flow between ally-be and ally-ai.

**S3** — `app/core/storage/s3_service.py`; results bucket set via `QUEUE__TRANSCRIBE_AND_SUMMARIZE_RESULTS_BUCKET`.

**Weaviate** — vector storage/search for embeddings and reference documents; schema managed by migrations (`app/migrations/`, `MigrationHistory` collection). Reference document matching uses `REFERENCE_DOCS__DISTANCE_THRESHOLD` (default `0.35`).

The `KnowledgeChunk` collection (migration `004`) backs the WhatsApp Q&A bot. Three things about it differ from the older collections and are deliberate:

- **The object UUID is `kb_document_chunks.id` in ally-be.** Postgres is the system of record; this index is derived.
- **It stores the chunk `text`**, unlike `RoadmapOpportunity`. The retrieve→generate loop runs inside ally-ai in one call, so without the text every question would need a back-call to ally-be. Staleness is closed structurally instead: chunk text is immutable for a given `(document_id, chunk_version, chunk_index)`, so an edit writes new objects and deletes the old ones rather than mutating in place.
- **There is no `tenant_id`.** The corpus is global. A future private per-tenant corpus is a NEW collection, not a filter bolted onto a shared one.

Retrieval uses two thresholds, not one: `MIN_SIMILARITY` (0.35) is a permissive floor and `DECLINE_SIMILARITY` (0.42) is the actual decision. A relevant passage matched against a short paraphrased question scores roughly 0.40–0.60 with `text-embedding-3-small`, so a single hard floor at the decision value would decline constantly on legitimate rephrasings.

**Slack** — optional alerting (`SLACK_ALERTS__*`).

## Local Setup

```bash
# Install dependencies (dev included)
poetry install            # or: make install

# Start Weaviate locally
docker-compose up -d

# Configure environment
cp env_sample .env        # edit values

# Run Weaviate schema migrations
poetry run python scripts/migrate.py all    # or: make migrate

# Start the API (http://localhost:8000, docs at /docs)
poetry run python app/main.py
```

Migration helpers: `scripts/migrate.py status | history | up | down | all | generate "<name>"`; `scripts/test_weaviate_connection.py`; `scripts/list_reference_documents.py`. Sync prompts to the backend with `poetry run python scripts/sync_prompts.py` (or `make sync-prompts`).

**Full local stack** (API + Weaviate + LocalStack SQS/S3) via `docker-compose.full.yml`:

```bash
docker compose -f docker-compose.full.yml up --build
```

- API exposed on `http://localhost:8001`; Weaviate on `8080`/`50051` (image `semitechnologies/weaviate:1.28.3`); LocalStack on `4566`.
- The app container runs `bootstrap_localstack.sh`, then migrations, then prompt sync, then starts both the API (`app.main`) and the transcription SQS worker (`app.core.queue.transcription_request_sqs_worker`).
- `scripts/bootstrap_localstack.sh` waits for LocalStack, creates the required queues and the results S3 bucket. Tear down with `docker compose -f docker-compose.full.yml down` (volumes persist).

## Testing & Code Quality

```bash
poetry run pytest                                   # or: make test  (pytest tests/ -v)
poetry run pytest --cov=app --cov-report=term-missing --cov-report=html
poetry run pytest tests/utils/ -v                   # utility unit tests
```

- Pytest config in `tests/pytest.ini` with `asyncio_mode = auto`; shared fixtures in `tests/conftest.py`. Unit tests focus on utility calculators (affirmation counter, positivity-lift, reflective-listening, interruption, silence, WER, etc.).
- Formatting/linting: `poetry run black app/`, `poetry run isort app/`, `poetry run flake8 app/` (config in `.flake8`, `pyproject.toml`).
- Pre-commit hooks: `pre-commit install` / `pre-commit run --all-files` (config in `.pre-commit-config.yaml`; secret scanning via `.gitleaks.toml`).

Makefile targets: `install`, `test`, `migrate`, `sync-prompts`.

## Key Documentation

- `README.md` — primary guide: overview, prerequisites, quick start, dev setup, Weaviate migration system, Docker/compose, full local stack, API docs endpoints (`/docs`, `/redoc`, `/openapi.json`), and project structure.
- `CONTRIBUTING.md` — Git conventions: branch naming, conventional commit/PR format, code review process.
- `tests/TESTING_SETUP.md` — test suite layout, coverage of utility calculators, running/coverage commands, fixtures, troubleshooting.
- `app/core/transcriptions/readme.md` — notes the transcription SQS worker (stub heading only).
- `.github/RELEASE_GUIDE.md` — release process guide.
- `.github/pull_request_template.md` — PR template.

---

*Part of the [Ally Platform](../platform/overview.md). See also: [Architecture](../platform/architecture.md), [ally-be](ally-be.md), [ally-ai-learn](ally-ai-learn.md).*
