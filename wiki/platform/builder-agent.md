---
title: Builder Agent — From a Sentence to a Merged Pull Request
tags: [platform, builder, agent, automation, ci, release, admin, llm]
summary: The Builder agent takes a described change, interviews you into a PRD, writes the code in a GitHub Actions runner, and keeps the resulting pull requests moving — reviewing, approving, prompting for merge and releasing. How the loop works, what bounds it, and why so much of its machinery is about not lying to the reader.
last_reconciled: 2026-09-17
---

# Builder Agent

Builder turns a described change into pull requests, then keeps them moving until
somebody merges them. You describe what you want in a sentence, it interviews you into a
PRD, dispatches a coding run into a GitHub Actions runner, and opens pull requests across
whichever repos the work touched. After that a reconcile loop watches them: reading CI,
bringing stale branches up to date, sending a reviewer at the diff, approving what comes
back clean, offering a merge button, and dispatching the production release once merged.

It is not a chat tool that writes code into your editor. Every run happens on a runner,
against real branches, and the artefacts are real pull requests that a person reviews.

---

## The shape of a session

A **session** is one piece of work. It moves through statuses:

| Status | Meaning |
|---|---|
| `INTERVIEWING` | The PRD interview is in progress |
| `PRD_READY` | The interview converged; there is something to build from |
| `BUILDING` | A run is dispatched or running |
| `WAITING_FOR_INPUT` | A run paused mid-build and needs an answer |
| `COMPLETED` | Pull requests opened, or the build finished with nothing to open |
| `FAILED` | The build gave up — retryable |
| `CANCELLED` | Stopped by a person |

A session owns **runs**. A run is one dispatched invocation on a runner, in one of four
modes:

- **build** — the main pass that writes the change
- **resume** — continues a run that paused on a question, once answered
- **fix** — sent at an open pull request with failing CI or unanswered review comments
- **review** — reads an already-open pull request with fresh context and reports findings

Within a run the agent announces a **stage** as it goes — setup, planning, coding,
testing, the machine test gate, verification, finalising, opening pull requests,
reporting, done. The stage is what the progress rail in the admin UI displays.

---

## The contract a run must keep

The runner cannot tell "finished quietly" from "died": a coding CLI exits 0 whenever the
agent produces a final response, including when it ends its turn mid-protocol. So the
protocol is explicit, and an outcome gate enforces it.

- **Announce stages**, so the UI reflects where the work is.
- **Run the machine test gate.** A run claiming `done` without a recorded passing gate is
  refused and recorded failed — testing used to be prompt-instructed with the agent's own
  summary as the only evidence, which is not checkable. A run that edited no files is
  exempt, because it is making no claim for a gate to verify.
- **Report an outcome exactly once, last.** A run that stops without doing so is recorded
  as a failure even when its work pushed cleanly.
- **Never wait for CI.** CI runs *after* the run ends, on the commits just pushed. There
  is nothing to wait for and nothing will notify the agent. If CI goes red, the reconcile
  loop dispatches a fresh fix run that can actually read the failure.

That last rule exists because a run once fixed a real defect, pushed it, went green — and
then spent two thirds of its turns trying to wait for confirmation that could never
arrive, before ending its turn without reporting. The work was merge-ready; the platform
recorded a failure, advised retrying it, and counted it against the circuit breaker.

### Asked for, or made true

The contract above is what the *agent* must do. Everything it can be spared, it is: the
runner does the work itself rather than instructing a model to.

This distinction was learned from the first run on a second engine. Two invariants had
only ever been requested in the prompt, and had only ever held because one particular
agent happened to comply:

- **The working branch.** The prompt said "create `builder/<slug>`". The test gate asks
  `git diff master...HEAD`, which is empty when HEAD *is* master — so an agent that
  committed straight to master produced a run where the gate saw nothing to gate on a
  repo holding the entire change, failed closed, and sent it to remediate work already
  done. Four rounds of that, seventeen minutes, a correct fix, no pull request, and a
  recorded verdict of "did not pass the test gate". The runner now checks the branch out
  before any agent starts, and refuses the run if it cannot.
- **The reporting helpers.** `stage`, `note`, `ask`, `complete-run` and the rest were
  shell *function definitions* embedded in the prompt. A coding agent's shell tool spawns
  a fresh shell per call, so a function defined in one does not exist in the next — they
  worked only because one agent pasted the whole body every time. The other read the
  documentation and ran `stage REMEDIATING`, getting `command not found`, silently,
  because telemetry is not allowed to fail a build. That run coded, tested and committed
  behind a progress rail frozen an hour earlier. They are executables on `PATH` now.

The general rule, and the more useful half of what that run taught: **an invariant
something else depends on should not rest on a model's willingness to read step four.**
Where the runner can establish a precondition itself, asking for it instead is a bug that
will find you the first time you change engines.

One corollary, worth knowing before writing another helper: the command is
`complete-run`, not `complete`, because `complete` is a bash builtin and builtins outrank
`PATH`. The old function form worked precisely because functions outrank builtins — so the
name was survivable for exactly as long as it was not a file. A test asserts no helper
name is shadowed this way.

---

## The reconcile loop

Every five minutes, for each open pull request Builder opened:

1. Refresh state from GitHub — merge status, head commit, check rollup.
2. Ingest feedback — failing checks and human review comments become actionable items.
3. Bring the branch up to date with master if it has fallen behind.
4. Dispatch a **review** run if the diff has not been read at this commit.
5. Otherwise dispatch a **fix** run if anything actionable is outstanding.
6. Approve, if a review came back clean and every required check is green.
7. Offer the merge button once GitHub itself says nothing is standing in the way.

A separate pass watches releases, and another re-tests failed session verdicts against
what the pull requests actually did.

### Switches

Four independent toggles, plus a master kill switch. They are separable on purpose —
"review it but do not spend on fixing it" is a useful setting, and so is "do everything
but do not deploy".

| Switch | What it allows |
|---|---|
| `enabled` | The kill switch. Nothing automatic runs without it |
| `autoReviewEnabled` | Sending a reviewer at an open pull request |
| `autoFixEnabled` | Sending a fix run at failing CI or review comments |
| `autoApproveEnabled` | Approving a pull request whose review came back clean |
| `autoReleaseEnabled` | Dispatching the production release after a merge |

Bringing a stale branch up to date is deliberately **not** behind `autoFixEnabled`: it is
one API call with no agent, no model and no runner behind it, and gating it on the
spend switch meant that turning off expensive work also turned off the free work — leaving
green, approved pull requests parked behind a branch that was merely out of date.

---

## What bounds it

Several limits, each answering a different runaway:

- **A spend ceiling per session.** A run that reaches it parks and asks, holding its work
  open for a window rather than discarding it. Raising the ceiling releases it.
- **A cap on fix runs per pull request.** A fix that cannot fix it will not fix it on the
  fourth attempt.
- **A cap on review runs per pull request**, so a reviewer is not re-reading an unchanged
  diff every few minutes.
- **A consecutive-failure circuit breaker** per session. Two failures in a row and
  automatic work stops until somebody looks.

The breaker judges on evidence rather than on the error text: a run that failed at the
protocol but changed files *and* left the session's pull requests green has converged, and
does not count. Both halves are required — edits with red CI is a fix loop making things
worse, and green CI with no edits credits the previous run's success twice.

---

## Approval, and why it is stricter than it looks

Builder can approve its own pull requests, which is the step that unblocks everything else:
`master` requires an approving review, and the bot holds only write access, so without
this a green, reviewed, finding-free pull request still waits on a human to click.

Two properties keep that honest.

**A dispatched review is not a passed review.** The field recording which commit a review
read is stamped at *dispatch*, so reconcile knows not to start a second review against the
same head. It says a review happened, not that it came back clean. Approval rests on a
separate record written only where zero findings were actually recorded — otherwise a
review run that dispatched and then crashed would look identical to a clean one.

**An approval survives Builder's own branch update.** Both repos protect master with
`dismiss_stale_reviews` *and* the requirement that a branch be current — together a closed
loop, since bringing a branch current pushes a commit that dismisses the approval, and the
review cap refuses a third review. It is broken by looking at what the new commit is: an
update-branch merge authored by us, carrying the reviewed head as its first parent, means
the pull request's own commits are untouched and only master moved underneath them. CI
must still be green on the new head, so a semantic conflict dragged in from master is
caught before anything is approved.

---

## Where it can work

Builder builds in `ally-be`, `ally-web`, `ally-ai`, `ally-ai-learn` and `ally-mobile`.
Each repo definition carries the commands the agent must use — its test, lint and
typecheck invocations — so the machine gate runs what CI runs rather than what the agent
guesses.

It runs on more than one coding engine, chosen per session or by a default in settings.
Only two files know anything engine-specific: one installs the engine, one invokes it and
normalises its output. Everything downstream — the event schema, the pipeline endpoints,
the admin UI — is engine-neutral.

Model routing is engine-aware, which it had to become rather than being designed that
way: the configured per-tier defaults are all Anthropic model ids, and several tiers could
reach one whatever engine was going to be handed it. A small build was the worst case,
because it plans on a cheaper tier that read the configured default directly and so
ignored the admin's settings entirely. A default belonging to another engine is now
skipped rather than translated — there is no honest mapping between two vendors' tiers,
and the worst case of skipping is a build that plans on the model somebody actually
picked.

Spend is reported by engines that report it and computed from token counts for engines
that do not. An estimate from a published rate card, not a billed figure — but a run that
cannot be priced cannot be capped, and a ceiling that reads every run on one engine as
free is not a ceiling.

Opportunities on the product roadmap can be handed to Builder directly. Doing so marks the
opportunity as under development, and a sweep moves it to released once every pull request
the session opened has merged **and** deployed. "Shipped" is deliberately stricter than
"merged": a merged pull request whose release failed is the one state a person most needs
to see, rather than have quietly marked done.

---

## Reading the UI honestly

A large share of Builder's machinery exists to stop the page stating the opposite of the
evidence beneath it, because in a system this asynchronous the two drift apart easily.

A failure reads where it happened — as the last entry in the run's own feed. It
used to be two banners, one pinned under the page header and one below the pull
request list, restating around a transcript what the transcript already
described; neither moved when you scrolled past what it described.

A run's fate and a pull request's fate are different facts. A run can fail at the protocol
while its work merges and deploys; a run can succeed having written nothing. The platform
therefore re-tests stored verdicts against evidence on a tick rather than writing them once
and trusting them: a session's failure banner clears when its pull requests are green, a
session settles completed when its work has merged, CI failures keyed to superseded commits
are retired, and a release marked failed is corrected when a successful release of that
target started after the merge.

The general rule, learned the expensive way: **put the correction where the evidence lives.**
A fix for the session verdict was written three times before it worked, twice placed inside
a loop over *open* pull requests — while the strongest evidence that work succeeded is a
merge, which is exactly what removes a pull request from that loop.

---

## Current maturity

Builder has taken work end to end unattended — diagnosing a failing check on a
pull request it had not written, fixing the cause, and going green without a
person touching it. That has happened **once**. Treat it as promising rather
than proven, and keep the session open while it works.

Multi-engine support is newer still, and worth stating precisely because the
first run on a second engine read as a failure and was not one. The engine
planned, coded, tested and wrote a correct fix. What it did not do was follow
two prompt-stated conventions that the platform's own gates depended on — so the
platform recorded a failed build holding working code. Both conventions are now
the runner's job rather than the agent's, but the lesson generalises past the
two that were found: **treat a new engine as exercising the harness, not the
model.**

Two limits are deliberately conservative while that record is short:

- **One build at a time.** Concurrent sessions have never run, so the per-session
  locks and budget accounting are untested in parallel. Raise
  `maxConcurrentBuilds` once there is evidence.
- **A spend ceiling of $25 per session**, which is roughly what a modest feature
  costs. A run that reaches it parks and asks rather than stopping.

If it gets stuck, that is worth reporting rather than working around — the
failure modes below were all found that way, and each one that gets fixed is one
nobody hits again.

## When something looks stuck

- **Nothing is dispatching.** A run parked on a question counts as active and blocks new
  dispatches until its resume exists. The refusals are logged; silence in the logs means
  the loop is genuinely idle, not wedged.
- **"Automatic runs paused" in chat.** The circuit breaker tripped. It announces once per
  trip, not once per refusal.
- **A run failed but the work looks fine.** Check the pull requests before retrying. A
  protocol failure on merged, green work is not a reason to redo it.
- **A release says merged but not deployed.** That is a real state and worth acting on —
  it is not corrected automatically unless a successful release of that target has since
  run.

---

*See also: [Architecture & Data Flow](architecture.md) · [Cross-Repo Agent Guide](agent-guide.md) · [Release Process](../contributing/release-process.md) · [Memory](../memory.md)*
