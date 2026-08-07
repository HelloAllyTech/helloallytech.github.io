---
title: Planning with the Stacks MCP
tags: [contributing, agents, planning, mcp, workflow]
summary: Every implementation plan starts with a Stacks MCP search — how to run the queries, how to cite what comes back, and how to configure the server.
---

# Planning with the Stacks MCP

**Stacks** is a retrieval service (RAG) that holds product-management and engineering-practice
guidance, exposed to Claude Code and other agents over MCP. It is wired into every Ally repo, so
the guidance is available at the moment a plan is being written rather than in review, when
re-planning is expensive.

## The rule

> [!IMPORTANT]
> **Before writing an implementation plan, call the stacks MCP's `search_chunks` tool with 2–3
> queries covering the task's main topics, and incorporate relevant returned guidance, citing
> chunk titles.**

This applies to any planning session that produces an implementation plan — a plan-mode plan, a
design doc, a spec, or a written breakdown before the first commit. Trivial mechanical work
(a rename, a dependency bump, a typo fix) does not need it.

## Running the search

**Use 2–3 queries, not one.** A single query retrieves one neighbourhood of the corpus. Split the
task along its actual topics so the queries land in different places:

| Task | Queries that cover it |
|---|---|
| A learner-facing streak widget | `gamification streaks`, `progress feedback loops`, `notification frequency` |
| A new analytics chart on the admin console | `dashboard chart selection`, `sample size honesty`, `admin permission gating` |
| Splitting a service in two | `service boundaries`, `incremental migration strategy`, `rollout and rollback` |

Write queries as topics, not as the ticket title. `"scenario_voices table migration"` retrieves
nothing; `"database migration rollout strategy"` retrieves the guidance that matters.

**Read the results before writing the plan, not after.** The point is that returned guidance
changes the plan — which slice ships first, which states the UI needs, what the non-goals are. A
search whose results arrive after the plan is written is a citation exercise.

## Citing what you use

Name the **chunk title** inline where the guidance shapes a decision, so a reviewer can trace it:

```markdown
## Plan

3. Ship the streak counter read-only in v1; no notifications until we have retention data.
   (Stacks: *Gamification — Extrinsic Rewards and Motivation Crowding*)
```

Cite only chunks you actually applied. If 2–3 queries return nothing relevant, say so in one line
— "Stacks searched (`x`, `y`, `z`) — nothing applicable" — and move on. A silent skip and a
genuine miss look identical to a reviewer.

## Stacks vs. the wiki's own practices

They are different things and both apply:

- **[Product Management Best Practices](../product/best-practices.md)** is *our* house rules,
  written from decisions this team has already made. It is binding, and it lives in this wiki.
- **Stacks** is a broader corpus of general guidance. It is advisory — it informs the plan, it
  does not overrule a practice we have already settled.

Where Stacks contradicts a settled Ally practice, the wiki wins. Where Stacks surfaces something
genuinely better than what the wiki says, that is a product judgement call: file it back into the
matching subsection in the same change, per §3 of the hub. That is how the corpus improves our
house rules instead of quietly competing with them.

## Configuration

Every Ally repo has a `.mcp.json` at its root declaring the `stacks` server, so the tool is
available as soon as an agent session starts in that repo. The file is committed; the credential
is not — it is read from the environment at connect time.

**One-time setup per engineer:**

1. Get a Stacks API key from the platform team. Keys are per-person; never commit or paste one
   into a repo, a wiki page, or a ticket.
2. Export it from your shell profile (`~/.zshrc`):
   ```bash
   export STACKS_API_KEY="…"
   ```
3. Start a fresh agent session in any Ally repo and approve the `stacks` server when prompted.
   Project-scoped MCP servers require approval once per project.

**Checking it works** — in a session, confirm a `search_chunks` tool is available and returns
results. If it is missing or every call fails to authorize:

- `echo $STACKS_API_KEY` in the *same* shell the agent was launched from — a variable exported in
  a different terminal, or set after the session started, will not be picked up.
- Confirm you are running in a repo that has `.mcp.json` at its root, and that you approved the
  server for that project.
- Restart the session after changing either — `.mcp.json` and the environment are both read at
  startup.

Do not work around a broken Stacks connection by skipping the search silently. Fix it, or say in
the plan that Stacks was unavailable.

---

*See also: [Contributing Guide](guide.md), [Cross-Repo Agent Guide](../platform/agent-guide.md),
[Product Management Best Practices](../product/best-practices.md).*
