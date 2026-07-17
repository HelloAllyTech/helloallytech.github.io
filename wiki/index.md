# Wiki Directory

Welcome to the **Ally Developer Wiki** catalog. This catalog is parsed by the LLMWiki engine to generate the navigation sidebar. It is the canonical, sanitized knowledge base for the Ally mental health counselor–training platform.

## Start Here
- [Welcome](welcome.md) — Welcome to the Ally Developer Wiki
- [Getting Started](getting-started.md) — How to operate this wiki

## Platform
- [Platform Overview](platform/overview.md) — What Ally is, its mission, and the ecosystem
- [Architecture & Data Flow](platform/architecture.md) — Services, integrations, queues, storage, HIPAA
- [Tech Stack](platform/tech-stack.md) — The polyglot stack across all layers
- [Cross-Repo Agent Guide](platform/agent-guide.md) — Conventions, common tasks, and gotchas for agents
- [Language-Quality Evaluation & RCA](platform/language-quality-eval.md) — Evaluating and fixing voice-agent language quality via LLM-judge error annotation + single-variable RCA
- [AI Lab — Prompt Workbench & Human Evaluation](platform/ai-lab.md) — Author prompt skills, run them against LLMs, and collect structured human evaluations

## Repositories
- [ally-be](repos/ally-be.md) — Core backend (NestJS)
- [ally-ai](repos/ally-ai.md) — AI copilot service (FastAPI + Weaviate)
- [ally-ai-learn](repos/ally-ai-learn.md) — Voice training agent (LiveKit + LangGraph)
- [ally-web](repos/ally-web.md) — Web apps (Nx: Next.js + Vite/React)
- [ally-mobile](repos/ally-mobile.md) — Mobile app (React Native)
- [infra](repos/infra.md) — Infrastructure & dev environment (Ansible + Terraform)

## Contributing
- [Developer Setup](contributing/dev-setup.md) — Clone repos and run the full local stack
- [Contributing Guide](contributing/guide.md) — SDLC rules, code standards, PR process

## Meta Context
- [Overview](overview.md) — Directory overview and structure
- [Memory](memory.md) — Long-term compiled agent memories
- [Context](context.md) — Operational context and session state

## Operations
- [Activity Log](log.md) — Chronological history of modifications

## Custom Skills
- [Example Skill](skills/example-skill/SKILL.md) — Demonstration of agent capabilities and loop policies
