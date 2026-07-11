---
title: infra — Infrastructure & Dev Environment
tags: [repo, infra, ansible, terraform, docker, devops]
summary: The infra repo provisions Ally's Hetzner baremetal, Incus containers, and AWS services via Ansible + Terraform, and holds the shell scripts developers use to bootstrap repos and run the local stack.
---

# infra — Infrastructure & Dev Environment

## Purpose

`infra` is the central repository for Ally's infrastructure-as-code and local development tooling. It serves two distinct audiences:

1. **Developers** — shell scripts that clone every Ally repo, set up Docker/Colima on macOS, and spin up the full local stack (backend + frontends + LocalStack/Postgres/Redis).
2. **Operators** — Ansible playbooks and Terraform templates that provision Hetzner baremetal servers, carve them into Incus (LXC) containers, deploy each Ally service from AWS ECR, and wire up TLS, DNS, and a Wireguard mesh.

Ally is a HIPAA-compliant platform for training mental health counselors; infra is where the production topology and the reproducible dev environment are defined.

## Tech Stack

- **Ansible** — primary provisioning/deployment tool. Playbooks target baremetal hosts and per-service containers; secrets live in Ansible Vault.
- **Terraform** — used *from within* Ansible (the `terraform` and `incus_tf` roles). There is no standalone `terraform/` directory; the Terraform config is a Jinja-templated `main.tf.j2` that drives the Incus provider.
- **Docker / Docker Compose** — every Ally service runs as a Compose stack, both locally and inside production containers.
- **Incus** (LXC) — baremetal servers are partitioned into Incus containers via the `lxc/incus` Terraform provider.
- **Hetzner baremetal** — a fleet of physical hosts is the production/dev substrate.
- **AWS** — ECR (images), SQS (queues), S3 (env files & assets), CloudWatch (HIPAA audit logs); accessed via `aws`/`aws_login` roles.
- **Colima** — lightweight Docker VM for macOS (Apple Silicon), the recommended local Docker runtime.
- **Caddy** (via `xcaddy`) — reverse proxy with automatic HTTPS / ACME DNS challenges.
- **Wireguard** — VPN mesh connecting containers, configured through cloud-init in the Incus templates.
- **Python/uv** — repo carries a `pyproject.toml`, `uv.lock`, and `main.py`; pre-commit + ansible-lint enforce quality.

## Repository Structure

```
infra/
├── bootstrap.sh              # Clone/update all Ally repos
├── bootstrap-contributor.sh  # Fork-based setup (origin + upstream) for external contributors
├── ally-env.sh               # Source to get ally-* command aliases
├── dev_env.sh                # Start full local stack (ally-be + ally-web)
├── dev_cleanup.sh            # Stop & remove all Docker containers
├── colima.sh                 # Start Colima VM (macOS)
├── colima-cleanup.sh         # Prune Docker + delete Colima instance
├── docker-setup.sh           # Detect/switch/verify Docker Desktop vs Colima
├── _os.sh                    # Cross-platform OS detection helpers (sourced by other scripts)
├── README.md, CONTRIBUTING-QUICK-START.md, infra_upgrade.md
├── ansible/
│   ├── *.yml                 # Playbooks (see below)
│   ├── ansible.cfg, requirements.yml
│   ├── inventories/          # development / production / global
│   │   ├── development/  (group_vars, host_vars: app, api, ai, learn, admin, helpline, metabase, …)
│   │   ├── production/
│   │   └── global/       (baremetal, CI/teamcity, build, dm hosts)
│   ├── roles/                # ~32 roles (see Infrastructure section)
│   └── templates/            # docker-compose.j2 per service + Caddyfiles
└── (no top-level terraform/ — TF lives in ansible/roles/incus_tf & terraform)
```

### Ansible Playbooks

| Playbook | Purpose |
|----------|---------|
| `baremetal.yml` | Full baremetal setup: common, incus, terraform, xcaddy, caddy_dns_acme, incus_consul, incus_tf |
| `baremetal_user.yml` | Bootstrap a baremetal host (packages + `ally` user) |
| `ally_core.yml` | Deploy `ally-be` (Postgres/Redis/SQS/LiveKit env, ECR image, migrations) |
| `ally_ai.yml` | Deploy `ally-ai` |
| `ally_learn.yml` | Deploy `ally-ai-learn` |
| `ally_web.yml` / `web.yml` | Deploy `ally-web` frontends |
| `ally_admin.yml` / `ally_helpline.yml` | Deploy admin / helpline dashboards |
| `metabase.yml`, `posthog.yml` | Analytics services |
| `dm.yml`* / `build.yml` / `ci.yml` | Domain manager, build, and CI/TeamCity hosts |
| `locale.yml` | Locale/i18n service host |

\*`dm.yml` is referenced in `ansible/README.md`. Roles are invoked with e.g. `ansible-playbook baremetal_user.yml -i inventories/global`.

## Developer Onboarding Scripts

The recommended entry point is to `source ally-env.sh`, which registers `ally-*` aliases (`ally-checkout`, `ally-colima`, `ally-dev`, `ally-dev-cleanup`, `ally-docker-setup`, `ally-changes`, `ally-help`) so the scripts run from anywhere.

- **`bootstrap.sh`** (alias `ally-checkout`) — Clones or updates every HelloAllyTech repo: `ally-ai`, `ally-ai-learn`, `ally-web`, `ally-be`, `ally-mobile`, `infra`, and `helloallytech.github.io`. Detects whether it runs from inside `infra` (targets the parent dir) or the parent. It fetches, and only pulls when the working tree is clean — repos with uncommitted changes are skipped with a warning. Requires SSH access to `git@github.com:HelloAllyTech/*`. Run with `./bootstrap.sh`.

- **`bootstrap-contributor.sh`** — Fork-based variant for external contributors; sets `origin` = your fork and `upstream` = the official repo (see `CONTRIBUTING-QUICK-START.md`).

- **`dev_env.sh`** (alias `ally-dev`) — Starts the full local stack. It sources `_os.sh`, then in `ally-be`: copies `.env`/`docker.env` from examples if missing, seeds `TEST_ACCOUNTS` (admin/counselor/learner/org-admin/user-cla, all with a default local password), runs `docker compose up -d`, `make health-check`, `npm run migration:run`, and `npm run seed`. Then in `ally-web`: builds `Dockerfile.deps` and runs `docker compose up -d --build`. Exposes Admin `:8081`, Helpline `:8080`, Web `:3000`.

- **`dev_cleanup.sh`** (alias `ally-dev-cleanup`) — Stops and removes all Docker containers.

- **`colima.sh`** (alias `ally-colima`) — Starts a Colima VM tuned for Apple Silicon dev: `--arch aarch64 --cpu 10 --memory 24 --disk 150 --vm-type vz --vz-rosetta --mount $HOME:w --mount-inotify --ssh-agent`. Guards with `require_os mac` (no-op on Linux/Windows). Sets `DOCKER_HOST` to the Colima socket.

- **`colima-cleanup.sh`** (alias `ally-colima-cleanup`) — macOS-only; `docker system/builder prune -a --volumes`, then stops and deletes the Colima instance (with confirmation).

- **`docker-setup.sh`** (alias `ally-docker-setup`) — Detects and manages the Docker backend. Subcommands: `detect` (default), `switch-to-colima`, `switch-to-desktop`, `fix-compose`, `test`, `help`. Verifies Docker CLI, active context, Colima status, socket location, and the Compose plugin.

- **`_os.sh`** — Shared helper sourced by the scripts above. Sets `ALLY_OS` (`mac`/`linux`/`wsl`/`windows`/`unknown`), exposes `is_mac`/`is_linux`/`is_windows`, `sed_inplace` (BSD vs GNU sed), and `require_os`.

### Cross-platform dev scripts (`_os.sh`)

The developer scripts are OS-aware via a shared **`_os.sh`** helper (sourced by the scripts above):

- **`_os.sh`** — sets `ALLY_OS` (`mac`/`linux`/`wsl`/`windows`/`unknown`), exposes `is_mac`/`is_linux`/`is_windows`, a BSD-vs-GNU `sed_inplace`, and `require_os` (gracefully skips + exits on the wrong platform).
- **`docker-setup.sh`** — prints OS-specific install hints for Docker/Compose (Homebrew on macOS, `apt`/`dnf` or `get.docker.com` on Linux, Docker Desktop on Windows) and runs Colima checks only on macOS.
- **`dev_env.sh`** — preserves an existing local `TEST_ACCOUNTS` value instead of overwriting it.
- **`colima.sh` / `colima-cleanup.sh`** — guard with `require_os mac`, so they no-op with a helpful message on Linux/Windows.

This makes the previously macOS-only scripts usable on Linux/WSL/Windows as well. (`colima.sh` allocates `--disk 150`, while `README.md` may still document a smaller figure.)

## Infrastructure & Deployment

Production/dev runs on a fleet of **Hetzner baremetal** hosts, each provisioned by `baremetal.yml` and subdivided into **Incus containers**, one per Ally service (`app`, `api`, `ai`, `learn`, `admin`, `helpline`, `metabase`, etc.), addressed on an internal private network.

- **Incus provisioning (`incus_tf` role)** — Templates `main.tf.j2` for the `lxc/incus` Terraform provider (`>=1.5.7`). Defines an `ally` project, a NAT'd `allybr0` network, a `dir` storage pool with per-container `home`/`data`/`postgres`/`docker` volumes plus a `shared` volume, and privileged/nested containers running `ubuntu/24.04/cloud`. Cloud-init `user-data`/`vendor-data`/`network-config` templates seed the host, the `ally` user, and **Wireguard** peering (endpoint, allowed IPs, per-container keys). Container proxy devices forward ports from host to container.
- **`terraform` role** — Installs the Terraform binary on the host when absent.
- **AWS integration** — `aws` role installs the CLI; `aws_login` writes `~/.aws/config`+`credentials`; `ecr_scripts` provides `ecr_login.sh`/`ecr_push.sh`. The `ally_docker_service` role pulls each service's env file from **S3**, logs into **ECR**, runs the Compose stack from the ECR image, and (for core) runs DB migrations. `ally_core.yml` wires **SQS** queue URLs, **CloudWatch** HIPAA log group, LiveKit, Postgres, and Redis into the container env.
- **Caddy / TLS** — `xcaddy` builds Caddy with plugins; `caddy_web` and `caddy_dns_acme` roles configure reverse proxies and ACME DNS-01 certificate issuance (per-host Caddyfiles under `ansible/templates/`).
- **Supporting roles** — `incus_consul` (service discovery), `hashistack`, `postgres`, `minio`, `rabbitmq`, `posthog`, `metabase`, `nodejs`/`nodejs_ci`, `poetry`, `golang`, `openjdk21`, `ruby_334`/`rails_app`, `teamcity`/`teamcity_agent` (CI), `vite_static_site`.
- **Secrets** — Ansible Vault; set `ANSIBLE_VAULT_PASSWORD_FILE` and place the deploy key at `~/.ssh/id_ansible_ally` (see `ansible/README.md`).

The `infra_upgrade.md` doc records a cost/reliability audit and a proposed AWS consolidation (collapsing services onto fewer nodes behind a bastion, keeping managed Postgres) to reduce monthly cost and operational surface.

## SQS Queues & LocalStack

Locally, SQS is emulated by **LocalStack**, brought up as part of the `ally-be` Docker Compose stack via `dev_env.sh`. Queue names/URLs referenced in the deployment inventories and playbooks include:

- `sqs-ai-transcription-requests-queue` — transcription requests (be → ai)
- `sqs-learn-message-and-event-queue` — learning events (ai-learn → be)
- Transcription request **DLQ** (`SQS_TRANSCRIPTION_REQUEST_DLQ_URL`)

`ally_core.yml` injects these as `SQS_TRANSCRIPTION_REQUEST_QUEUE_URL`, `SQS_TRANSCRIPTION_REQUEST_DLQ_URL`, and `SQS_LEARN_MESSAGE_AND_EVENT_QUEUE_URL`.

## Key Documentation

- **`README.md`** — Primary dev guide: bootstrap, Docker/Colima setup, `docker-setup.sh` command reference, and the "new developer" quick-start flow.
- **`ansible/README.md`** — Ansible Vault setup, deploy private key, and example playbook invocations.
- **`CONTRIBUTING-QUICK-START.md`** — Fork/upstream workflow for external contributors (`bootstrap-contributor.sh`, PR flow, SSH troubleshooting).
- **`infra_upgrade.md`** — May 2026 AWS cost-optimization audit and the target "2 Nodes + 1 Bastion" architecture.

---

*Part of the [Ally Platform](../platform/overview.md). See also: [Architecture](../platform/architecture.md), [Developer Setup](../contributing/dev-setup.md).*
