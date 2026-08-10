---
title: Working with the Stacks MCP
tags: [contributing, agents, planning, mcp, workflow]
summary: Search Stacks whenever a product judgement comes up — while planning and while coding — how to run the queries, how to cite what comes back, and how the server is wired in.
last_reconciled: 2026-08-10
---

# Working with the Stacks MCP

**Stacks** is a retrieval service (RAG) that holds product-management and engineering-practice
guidance, exposed to Claude Code and other agents over MCP. It is wired into every Ally repo, so
the guidance is available at the moment a decision is being made rather than in review, when
re-deciding is expensive.

## The rule

> [!IMPORTANT]
> **Whenever a decision about how the product should behave comes up, call the stacks MCP's
> `search_chunks` tool with 2–3 queries on the topic, incorporate relevant returned guidance, and
> cite chunk titles. Planning is the most important moment, not the only one.**

Concretely, that means:

| Moment | Why it matters |
|---|---|
| **Before writing an implementation plan** — a plan-mode plan, design doc, spec, or written breakdown before the first commit | The original rule. Guidance that arrives after the plan is a citation exercise |
| **While implementing**, at each point you would otherwise invent the answer: an empty, loading, edge or failure state; a user-facing label, button or error message; what a view shows and what it omits; a threshold, limit, cadence or reward rule | Where most product decisions are actually made, and where the corpus went unused before 2026-08-10 |
| **While reviewing**, for how a change behaves rather than how it reads | Cheaper than re-deciding after merge |

Trivial mechanical work (a rename, a dependency bump, a typo fix) does not need it.

> [!NOTE]
> Until 2026-08-10 this rule — and the MCP server's own tool descriptions — said *planning* only.
> That wording was load-bearing in a way that was easy to miss: a connected session reads the tool
> description to decide when the tool applies, concludes Stacks is a planning tool, and stops
> querying the moment implementation starts, no matter what a repo's `CLAUDE.md` says. Both the
> server copy and the repo instructions were widened together; changing one without the other
> puts it back.

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

**Read the results before you decide, not after.** The point is that returned guidance changes
the outcome — which slice ships first, which states the UI needs, what the non-goals are. A
search whose results arrive after the decision is a citation exercise.

`get_chunks` fetches the verbatim source excerpt behind a chunk when the exact wording matters;
`list_tags` and `list_documents` show what the corpus covers when you are not sure it holds
anything on your topic.

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

## Stacks replaced the in-wiki product practices

Until 2026-08-07 this wiki carried its own **Product Management Best Practices** section — a hub
plus seven pages of house rules, written from decisions this team had made. That section is
**deprecated**; Stacks is now where product guidance comes from, and judgement calls made during
a task go into the Stacks corpus rather than back into `wiki/product/`.

The old pages are still readable and still linked. Treat them as **history, not rules**:

- Nothing in them is a gate any more — including the Data Visualisation checklist, which used to
  be enforced as one.
- Where they contradict current Stacks guidance, Stacks wins.
- They are not deleted because several platform pages cite their principles by number
  ([Analytics Agent](../platform/analytics-agent.md) references *Data Visualisation* 13 and
  27–28 and *UI & Interaction* 4–5), and because they record why Ally works the way it does —
  which is context a general corpus does not have.

That last point is worth being deliberate about. The deprecated pages hold Ally-specific
findings — Carbon's chart-overflow behaviour, the `roles[]`-vs-collapsed-`role` gating trap, the
minimum-group-size rule for tenant-isolated metrics — that a general product-management corpus
has no reason to contain. **If a query returns nothing and you suspect the answer is one of
those, check the deprecated page before re-deriving it.** Anything you find there that is still
true and still load-bearing belongs in Stacks; putting it back into the wiki is not the path any
more.

## Configuration

Every Ally repo commits four things at its root, so a session is wired up the moment it starts
and nobody has to add or invoke anything by hand:

| File | What it does |
|---|---|
| `.mcp.json` | Declares the `stacks` server. Committed; the credential is not — `${STACKS_API_KEY}` is read from the environment at connect time |
| `CLAUDE.md` / `AGENTS.md` | State the rule above, so it is in context from the first turn |
| `.claude/skills/stacks/SKILL.md` | A skill whose description names the trigger moments. Its one-line description sits in context every turn at negligible cost, so the rule keeps re-asserting itself deep into a long session, where a `CLAUDE.md` line read at startup has long since stopped competing for attention |
| `.claude/settings.json` + `.claude/hooks/stacks-search.sh` | A `UserPromptSubmit` hook that runs a small search automatically when a prompt looks product-shaped |

**About the hook.** It is a floor, not the rule — it catches prompts you would otherwise not have
searched on, and it is deliberately conservative:

- It fires only on product-judgement phrasing. "Fix the typo", "bump axios", "why is this test
  failing" do not match; "what should the empty state show", "add a streak counter", "the right
  wording for this error message" do.
- It asks for 3 chunks, not the server default of 8 — roughly 1.3k tokens rather than 3.4k, which
  is the difference between an occasional nudge and forcing compaction.
- It stops after 4 injections per session.
- It fails open and silent. No key, a Stacks outage, a slow response, malformed input — every path
  exits cleanly and your prompt goes through untouched. You will not be told it did nothing.

Because it fails silently by design, **never treat the hook as evidence the search happened.**
When guidance mattered, query deliberately and cite what you used.

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

To check the hook specifically, run it by hand from a repo root — it should print a JSON block
with an `additionalContext` field:

```bash
echo '{"prompt":"what should the empty state show","session_id":"manual"}' | .claude/hooks/stacks-search.sh
```

Silence means one of its guards tripped: no `STACKS_API_KEY` in that shell, no `jq`, the prompt
did not match the gate, or the per-session cap was already spent.

Do not work around a broken Stacks connection by skipping the search silently. Fix it, or say in
the plan that Stacks was unavailable.

---

*See also: [Contributing Guide](guide.md), [Cross-Repo Agent Guide](../platform/agent-guide.md),
[Product Management Best Practices](../product/best-practices.md).*
