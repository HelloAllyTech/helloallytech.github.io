---
title: Developer Setup
tags: [contributing, setup, onboarding, environment]
summary: How to clone all Ally repositories and start the full local development environment.
---

# Developer Setup

This guide gets you from zero to a running Ally development environment. For repo-specific commands, see each [repo page](../index.md#repositories).

## Quick Start

1. **Clone the infrastructure repo** (the easiest way to get everything):
   ```bash
   git clone git@github.com:HelloAllyTech/infra.git
   cd infra
   ./bootstrap.sh
   ```
   This clones and sets up all Ally repositories. `bootstrap.sh` fetches every repo and only pulls when the working tree is clean (repos with uncommitted changes are skipped with a warning).

2. **Choose your area of interest** and navigate to that repository.

3. **Read the [Contributing Guide](guide.md)** for SDLC rules and per-repo standards.

## Full Local Stack (Web & Backend)

After cloning all repos, start the backend and frontends with a single command from the workspace root:

```bash
./infra/dev_env.sh
```

This will:
1. Start all backend services in `ally-be` via Docker Compose (Postgres, Redis, LocalStack/SQS), run migrations, and seed the database.
2. Build and start the `ally-web` frontends.
3. Print the URLs for the running applications.

Once finished, access:
- **Landing page (Web):** `http://localhost:3000`
- **Helpline Dashboard:** `http://localhost:8080`
- **Admin Dashboard:** `http://localhost:8081`
- **Backend API / Swagger:** `http://localhost:8001/api-docs`

To stop everything: `./infra/dev_cleanup.sh` (or `docker compose down` inside the relevant repo).

## Environment Files

- **Python repos** (ally-ai, ally-ai-learn): copy `env_sample` → `.env`.
- **TypeScript repos** (ally-be, ally-web, ally-mobile): copy `.env.example` / `docker.env.example`.
- **Never commit `.env` files.** Required keys per service are listed in each [repo page](../index.md#repositories) and in the repo's env sample.

## General Requirements

- Git and SSH access to the HelloAllyTech repositories.
- Docker and Docker Compose (recommended via **Colima** on macOS — see below).
- Your preferred editor (VS Code recommended).

### Repository-specific runtimes
- **ally-web** — Node.js 22, npm
- **ally-be** — Node.js 24, PostgreSQL, Redis
- **ally-mobile** — Node.js ≥ 18, React Native CLI, Xcode / Android Studio
- **ally-ai / ally-ai-learn** — Python 3.12, Poetry
- **infra** — Ansible, Terraform, Docker

## macOS: Colima (recommended Docker runtime)

**Colima** is a lightweight, free alternative to Docker Desktop for container virtualization — open-source, resource-efficient, fast, and fully Docker-CLI compatible.

The `infra` repo provides helpers:
```bash
./infra/colima.sh                 # start a tuned Colima VM
./infra/docker-setup.sh detect    # detect/switch between Colima and Docker Desktop
```

`docker-setup.sh` supports `switch-to-colima`, `switch-to-desktop`, `fix-compose`, and `test` subcommands, and prints OS-specific install hints on macOS/Linux/Windows.

## First Time Here?

- **Frontend:** [ally-web](../repos/ally-web.md) — React / Next.js
- **Mobile:** [ally-mobile](../repos/ally-mobile.md) — React Native
- **Backend:** [ally-be](../repos/ally-be.md) — NestJS API
- **AI/ML:** [ally-ai](../repos/ally-ai.md) or [ally-ai-learn](../repos/ally-ai-learn.md)
- **DevOps:** [infra](../repos/infra.md)

## How We Track Issues

We use GitHub Issues across all repositories. Labels include `bug`, `enhancement`, `good first issue`, and `help wanted`; work is organized into milestones. To find work: browse the repo you're interested in, look for `good first issue` / `help wanted`, check assignment, and comment to express interest before starting.

---

*See also: [Contributing Guide](guide.md), [Platform Overview](../platform/overview.md).*
