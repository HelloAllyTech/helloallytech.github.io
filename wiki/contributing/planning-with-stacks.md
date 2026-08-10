---
title: Working with the Stacks MCP
tags: [contributing, agents, planning, mcp, workflow]
summary: Pull Stacks context whenever a product judgement comes up — while planning and while coding — how an agent searches the library itself, how to cite what comes back, and how the server is wired in.
last_reconciled: 2026-08-10
---

# Working with the Stacks MCP

**Stacks** is a retrieval service (RAG) that holds product-management and engineering-practice
guidance, exposed to Claude Code and other agents over MCP. It is wired into every Ally repo, so
the guidance is available at the moment a decision is being made rather than in review, when
re-deciding is expensive.

## The rule

> [!IMPORTANT]
> **Whenever a decision about how the product should behave comes up, search Stacks, incorporate
> relevant returned guidance, and cite chunk titles. An agent does this for itself — you do not
> have to fetch context on its behalf. Planning is the most important moment, not the only one.**

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

## How you reach it

Four surfaces. Three are tools an agent calls for itself; one is a prompt only you can invoke:

| Surface | Type | Who invokes it |
|---|---|---|
| `search_chunks` | tool — `query`, optional `max_results` (1–10, default 4) and `tags` | the agent |
| `get_chunks` | tool — `ids` (1–20) from a search result or a returned block | the agent |
| `list_tags` | tool — no arguments | the agent |
| `/stacks:planning_context` | MCP **prompt**, one argument `task_description` | you, in chat |

The split is the thing to hold on to. **An agent reaches the library on its own initiative, at the
moment a judgement actually comes up** — which is usually mid-implementation, long after any
planning step. The prompt is the *human* entry point, for when you want the context in front of
you rather than in a session's working memory; it takes a whole task description and returns full
chunk bodies.

> [!NOTE]
> For a few hours on 2026-08-10 the server was narrowed to just `get_chunks` and the prompt, and
> this page said "there is no search tool". That was a mistake and was reversed the same day.
> `list_documents` was not restored — a book catalogue is browsing, whereas tags feed back into a
> search filter. If you find a page or a repo instruction claiming an agent cannot search, it is
> stale; this table is current.

One habit survived the reversal, because it was always true for a different reason:

- **Don't let a session claim Stacks does or doesn't cover something.** `list_tags` shows how the
  library is organised, not what it contains, and a search returning nothing is not evidence of a
  gap. The most a session can honestly say is what a particular result set did or didn't contain.
  A confident "Stacks has nothing on empty states" is not a checkable claim.

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

**Judge relevance yourself.** Both surfaces always return their top-ranked matches whether or not
they fit — there is no relevance floor and no "nothing found" response. The prompt returns eight
chunks; `search_chunks` returns four by default. A result set full of off-topic chunks is a normal
outcome, not a signal to force a fit.

**An agent's queries look different from yours.** `search_chunks` wants a specific noun phrase
(`"empty state design patterns"`) and works best called two to four times across a task's distinct
aspects; the prompt wants the whole task description above. Both are right — the prompt runs one
search over whatever you typed, while an agent can afford several sharp queries. Search results
come back compact (title, book, section, framing sentence, id) so that searching often is cheap.

`get_chunks` fetches the verbatim source excerpt behind a chunk, plus its section and book
summaries, when the exact wording matters. It takes ids from a search result or a returned block;
invented ids will not resolve.

> [!WARNING]
> **A rate-limit error means retry, not "no guidance found".** If a search fails that way, wait a
> few seconds and run it again before concluding anything about the corpus. The server now says
> which failure it hit, so an agent has no excuse for turning a throttled search into an invented
> answer.
>
> The "3 requests per minute" figure that used to sit here was wrong for retrieval. It is
> `EMBED_MAX_RPM`, a deliberately pessimistic guess at the embedder's free tier that applies only
> to *book ingestion*; the search path never passed through that limit at all. The real retrieval
> ceiling is unmeasured — the authoritative numbers are the rate-limit headers the server logs on
> a 429. Don't quote 3/min.

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

Every Ally repo commits three things at its root, so a session is wired up the moment it starts
and nobody has to add or invoke anything by hand:

| File | What it does |
|---|---|
| `.mcp.json` | Declares the `stacks` server. Committed; the credential is not — `${STACKS_API_KEY}` is read from the environment at connect time |
| `CLAUDE.md` / `AGENTS.md` | State the rule above, so it is in context from the first turn |
| `.claude/skills/stacks/SKILL.md` | A skill whose description names the trigger moments. Its one-line description sits in context every turn at negligible cost, so the rule keeps re-asserting itself deep into a long session, where a `CLAUDE.md` line read at startup has long since stopped competing for attention |

**The `UserPromptSubmit` hook was retired on 2026-08-10.** Every repo used to commit
`.claude/settings.json` and `.claude/hooks/stacks-search.sh`, a keyword-gated hook that pulled a
small context block when a prompt *looked* product-shaped. It existed only because an agent could
not search for itself; once `search_chunks` came back it was redundant, and keeping it would have
meant two paths spending the same quota on overlapping queries.

It is worth knowing why a phrase gate was never good enough, because the same trap applies to any
replacement: it fired on the *prompt text*, so it could not reach the library mid-implementation
when no new prompt had arrived — which is exactly where most product decisions get made. Its regex
also both over- and under-matched ("what happens when the list is empty" did not match; `empty
state` was needed verbatim). And it failed open and silent by design, so a session that got no
context looked identical to one where the library had nothing.

That last property is the one to carry forward: **silence is not evidence the library was
consulted.** Ask what was searched and what was cited.

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

To check the tools specifically, ask the session to list them — `search_chunks`, `get_chunks` and
`list_tags` should all be available, alongside `/stacks:planning_context` as a slash command. A
session that offers only `get_chunks` is talking to a stale deployment.

If a search returns an authorization error, it is the key; if it returns a rate-limit message,
retry. Neither means the corpus is empty.

Do not work around a broken Stacks connection by skipping the library silently. Fix it, or say in
the plan that Stacks was unavailable.

---

*See also: [Contributing Guide](guide.md), [Cross-Repo Agent Guide](../platform/agent-guide.md),
[Product Management Best Practices](../product/best-practices.md).*
