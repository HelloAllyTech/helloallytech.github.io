# CLAUDE.md — Ally Developer Wiki

This repository **is** the [Ally Developer Wiki](https://tech.helloally.ai) — the canonical, public documentation for the Ally platform, built on [LLMWiki](https://github.com/ajeygore/llmwiki).

## How to work here
- Read **`agents.md`** at the repo root — it is the authoritative guide for maintaining this wiki (Ingestion, Query, and Lint flows).
- All human-readable content lives in **`wiki/`**. Edit those Markdown files directly.
- The **`llmwiki/`** folder is the rendering engine — **never modify it** (it lives in a separate upstream repo).
- When you add or change pages: update the catalog in **`wiki/index.md`** and append an entry to **`wiki/log.md`**. Record durable learnings in `wiki/memory.md` and session state in `wiki/context.md`.
- **Product work:** before designing or building anything user-facing anywhere in the Ally platform, read **`wiki/product/best-practices.md`** (the hub — house rules + subsection index) and the matching subsection (`wiki/product/ui.md`, `gamification.md`, `data-visualisation.md`, `prioritisation.md`, `user-personas.md`, …). It is an intentionally growing section: file new product judgement calls back into the right subsection, or add a new one per §3 of the hub, then register it in the hub table and `wiki/index.md`.
- `.nojekyll` keeps GitHub Pages from processing the Markdown; pushing to `main` publishes to **tech.helloally.ai** (the `CNAME`).

## Content policy (public site)
This site is **public**. Document architecture, SDLC rules, deployment, and environment *concepts*, but **never** commit secrets, credentials, IP addresses, internal hostnames/domains, or cloud region details.

## Scope
This wiki documents the whole platform; each code repo (`ally-be`, `ally-ai`, `ally-ai-learn`, `ally-web`, `ally-mobile`, `infra`) points its own `AGENTS.md`/`CLAUDE.md` back here. Keep the per-repo pages under `wiki/repos/` in sync when those repos change.
