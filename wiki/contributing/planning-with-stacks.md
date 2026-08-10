---
title: Working with the Stacks MCP
tags: [contributing, agents, planning, mcp, workflow]
summary: Pull Stacks context whenever a product judgement comes up — while planning and while coding — how to run the planning_context prompt, how to cite what comes back, and how the server is wired in.
last_reconciled: 2026-08-10
---

# Working with the Stacks MCP

**Stacks** is a retrieval service (RAG) that holds product-management and engineering-practice
guidance, exposed to Claude Code and other agents over MCP. It is wired into every Ally repo, so
the guidance is available at the moment a decision is being made rather than in review, when
re-deciding is expensive.

## The rule

> [!IMPORTANT]
> **Whenever a decision about how the product should behave comes up, run
> `/stacks:planning_context` with a description of the task, incorporate relevant returned
> guidance, and cite chunk titles. Planning is the most important moment, not the only one.**

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

## There is no search tool

Later on 2026-08-10 the server was narrowed to a single entry point, and it is not a tool:

| Surface | Type | Who invokes it |
|---|---|---|
| `/stacks:planning_context` | MCP **prompt**, one argument `task_description` | you, in chat |
| `get_chunks` | tool, `ids` (1–20) from a returned block | the agent |

`search_chunks`, `list_tags` and `list_documents` were **removed outright** — calls to them now
fail as unknown tools.

The structural consequence is the part worth holding on to. A prompt runs only when a human
invokes it, so **an agent can no longer reach the library on its own initiative, and can no longer
enumerate what the library contains.** Two habits follow:

- **Don't let a session claim Stacks does or doesn't cover something.** It has no way to check, and
  a confident "Stacks has nothing on empty states" is unfalsifiable from inside the session. The
  most it can honestly say is what a returned block did or didn't contain.
- **A missing context block usually means nobody ran the prompt** — not that the library came up
  empty. If an agent is about to invent an answer, run the prompt for it.

## Getting the context

Describe the task, not a search phrase. The argument is a task description and the server runs one
hybrid search over it:

```
/stacks:planning_context rolling out a database migration that changes voice config for live sessions
```

| Task | A description that works |
|---|---|
| A learner-facing streak widget | `adding a streak counter to the learner home screen — what cadence, what reward, what it shows on a broken streak` |
| A new analytics chart on the admin console | `a new chart on the admin analytics console showing per-cohort practice minutes, including small-sample cohorts` |
| Splitting a service in two | `splitting the session service in two, migrating incrementally without downtime` |

`"scenario_voices table migration"` retrieves nothing useful; the sentence above it retrieves the
guidance that matters. Minimum three characters.

**Read the results before you decide, not after.** The point is that returned guidance changes
the outcome — which slice ships first, which states the UI needs, what the non-goals are. Context
that arrives after the decision is a citation exercise.

**Judge relevance yourself.** The prompt always returns its eight top-ranked chunks whether or not
they fit — there is no relevance floor and no "nothing found" response. A block full of off-topic
chunks is a normal result, not a signal to force a fit.

`get_chunks` fetches the verbatim source excerpt behind a chunk, plus its section and book
summaries, when the exact wording matters. It takes ids from a block already in the conversation
and cannot search; invented ids will not resolve.

> [!WARNING]
> **The server's upstream embedder is rate limited to 3 requests per minute**, shared across every
> session, every engineer and the automatic hook. Over that ceiling the call returns an error —
> and because the hook fails silent, that looks exactly like "no guidance found". If a block
> doesn't appear when you expected one, wait a minute and run the prompt again before concluding
> anything about the corpus.

## Citing what you use

Name the **chunk title** inline where the guidance shapes a decision, so a reviewer can trace it:

```markdown
## Plan

3. Ship the streak counter read-only in v1; no notifications until we have retention data.
   (Stacks: *Gamification — Extrinsic Rewards and Motivation Crowding*)
```

Cite only chunks you actually applied. If a block came back with nothing relevant, say so in one
line — "Stacks context retrieved — nothing applicable" — and move on. A silent skip and a genuine
miss look identical to a reviewer. If no block was retrieved at all, say *that* instead; "nothing
applicable" and "never looked" are different claims and only one of them is checkable.

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
has no reason to contain. **If a block comes back with nothing for one of those, check the
deprecated page before re-deriving it.** Anything you find there that is still
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
| `.claude/settings.json` + `.claude/hooks/stacks-search.sh` | A `UserPromptSubmit` hook that pulls a small context block automatically when a prompt looks product-shaped |

**About the hook.** Since the server dropped its search tool, this hook is the *only* automatic
path into the library — an agent cannot invoke `planning_context` itself. It is still a floor
rather than the rule, and deliberately conservative:

- It fires only on product-judgement phrasing. "Fix the typo", "bump axios", "why is this test
  failing" do not match; "what should the empty state show", "add a streak counter", "the right
  wording for this error message" do.
- It trims the response to the 3 strongest chunks. `planning_context` has no `max_results` and
  always returns 8 (~3.2k tokens); the trim brings that back to ~1.4k, which is the difference
  between an occasional nudge and forcing compaction. Kept chunks keep their ids, so `get_chunks`
  still works on them.
- It stops after 4 injections per session, and waits at least 25 seconds between fires. The
  server no longer rate limits us, but its upstream embedder does — 3 requests/minute shared with
  whatever you run by hand — so an ungoverned hook would eat the quota your own
  `/stacks:planning_context` needs.
- It fails open and silent. No key, a Stacks outage, a slow response, a rate-limit error,
  malformed input — every path exits cleanly and your prompt goes through untouched. You will not
  be told it did nothing.

Because it fails silently by design, **never treat the hook as evidence the library was
consulted.** When guidance mattered, run the prompt deliberately and cite what you used.

**One-time setup per engineer:**

1. Get a Stacks API key from the platform team. Keys are per-person; never commit or paste one
   into a repo, a wiki page, or a ticket.
2. Export it from your shell profile (`~/.zshrc`):
   ```bash
   export STACKS_API_KEY="…"
   ```
3. Start a fresh agent session in any Ally repo and approve the `stacks` server when prompted.
   Project-scoped MCP servers require approval once per project.

**Checking it works** — in a session, confirm `/stacks:planning_context` is offered as a slash
command and returns a block of chunks. If it is missing or every call fails to authorize:

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
did not match the gate, the per-session cap was already spent, the 25-second cooldown had not
elapsed, or the upstream embedder rate-limited the call. The cooldown is machine-wide, so a
second run straight after the first is silent by design — wait, then retry.

Do not work around a broken Stacks connection by skipping the library silently. Fix it, or say in
the plan that Stacks was unavailable.

---

*See also: [Contributing Guide](guide.md), [Cross-Repo Agent Guide](../platform/agent-guide.md),
[Product Management Best Practices](../product/best-practices.md).*
