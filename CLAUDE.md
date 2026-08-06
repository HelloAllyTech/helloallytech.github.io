# CLAUDE.md — Ally Developer Wiki

This repository **is** the [Ally Developer Wiki](https://tech.helloally.ai) — the canonical, public documentation for the Ally platform, built on [LLMWiki](https://github.com/ajeygore/llmwiki).

## How to work here
- Read **`agents.md`** at the repo root — it is the authoritative guide for maintaining this wiki (Ingestion, Query, and Lint flows).
- All human-readable content lives in **`wiki/`**. Edit those Markdown files directly.
- The **`llmwiki/`** folder is the rendering engine — **never modify it** (it lives in a separate upstream repo).
- When you add or change pages: update the catalog in **`wiki/index.md`** and append an entry to **`wiki/log.md`**. Record durable learnings in `wiki/memory.md` and session state in `wiki/context.md`.

## Generated files — never hand-edit
`wiki/ROUTING.md`, `wiki/manifest.json` and `wiki/platform/stats.md` are produced by
`scripts/`. Edit the source instead: page frontmatter for the first two, the code itself
for the third. CI fails a PR whose generated files are stale — run
`python3 scripts/gen-routing.py` and commit.

Every page needs frontmatter with `title`, `tags`, `summary` **and `last_reconciled`** —
`ROUTING.md` is built from those keys, so a page without them is invisible to agents.

Before adding a number to a page, check whether `scripts/gen-stats.py` could count it.
Prose states invariants; `platform/stats.md` states counts.

## The docs machinery
`wiki/contributing/docs-system.md` is the spec for how all of this fits together —
routing, `.docs-map.yml`, the CI guard, `scripts/wiki-pr.sh`, PR lifecycle coupling and
the weekly health sweep. Read it before changing anything under `scripts/` or
`.github/workflows/`.
- **Product work:** before designing or building anything user-facing anywhere in the Ally platform, read **`wiki/product/best-practices.md`** (the hub — house rules + subsection index) and the matching subsection (`wiki/product/ui.md`, `gamification.md`, `data-visualisation.md`, `prioritisation.md`, `user-personas.md`, …). It is an intentionally growing section: file new product judgement calls back into the right subsection, or add a new one per §3 of the hub, then register it in the hub table and `wiki/index.md`.
- `.nojekyll` keeps GitHub Pages from processing the Markdown; pushing to `main` publishes to **tech.helloally.ai** (the `CNAME`).

## Content policy (public site)
This site is **public**. Document architecture, SDLC rules, deployment, and environment *concepts*, but **never** commit secrets, credentials, IP addresses, internal hostnames/domains, or cloud region details.

## Scope
This wiki documents the whole platform; each code repo (`ally-be`, `ally-ai`, `ally-ai-learn`, `ally-web`, `ally-mobile`, `infra`) points its own `AGENTS.md`/`CLAUDE.md` back here. Keep the per-repo pages under `wiki/repos/` in sync when those repos change.
