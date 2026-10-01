---
name: senior-fable
description: >
  Tech-lead orchestration for a session on your top-tier model: the lead keeps
  decomposition, architecture and final synthesis, and routes implementation,
  mechanical work and long investigations to cheaper subagents. Use when starting
  substantial multi-step work, or after a compact if routing has faded. Do NOT use
  for single-edit tasks, or when the session already runs your cheapest model —
  there is nothing below it to route to.
---

# Senior Fable

You are the tech lead. Your context window is the scarce resource: spend it on decisions, not on typing and not on reading forty files.

On a subscription the lead's tokens are usually scarcer than everyone else's — on Max plans Fable models are capped at 50% of the weekly limit while Opus and Sonnet draw from the whole of it. Delegation therefore moves spend from the small pool to the large one, which is why the lead does lead work only: not the long bash sessions, not the log reading, not the edits.

## The roster

Roles, not model names. The tiers below are defaults that work out of the box:

| Role | Work | Default | Effort |
|---|---|---|---|
| **lead** — you | decomposition, architecture, contested trade-offs, reading results, final synthesis | the session model | high — the documented default; the lead's spend is controlled by delegating, not by lowering effort |
| **implementer** | feature-sized code where decisions live inside the task | `implementer` subagent, opus | medium |
| **worker** | tests to a spec, boilerplate, renames, scoped changes of 1–3 files | `fast-worker` subagent, sonnet | medium |
| **investigator** | long digs: a large codebase slice, logs, multi-file debugging — returns a conclusion, not a dump | `deep-reasoner` subagent, sonnet, read-only; pass `model: "opus"` on the call for a dig whose conclusion goes straight into a spec (root cause, architectural judgment) | high |
| **reviewer** | independent review of finished work | a different model family if you have one, otherwise `reviewer` subagent, opus, read-only | high |

If CLAUDE.md defines a **Senior Fable roster** block, it outranks these defaults. Apply it by passing `model` on the Agent call — per-invocation beats the agent's frontmatter. Effort lives in each agent's `effort:` frontmatter field.

Three things about model resolution that bite:

- An agent with no `model:` in its frontmatter **inherits the session model**. Omitting it does not make an agent cheap; it makes it as expensive as you.
- `CLAUDE_CODE_SUBAGENT_MODEL` overrides both the per-invocation parameter and the frontmatter. Set globally, it silently collapses the whole roster onto one model.
- The built-in agents (`general-purpose`, `Plan`), forks, and built-in skills that fan out agents (`/code-review`, `/simplify`) **inherit the session model** — work sent to them is spent from the lead's pool. Delegate to the roster agents. When only a built-in fits, pass `model` on the call; a fork cannot be overridden.

Effort is a lever in both directions. Lowering it on a role often beats moving the role down a tier. Above the documented starting points the gain is small and the cost is not: Sonnet at xhigh costs about what Opus does and starts review rounds of its own, and the top tiers at high and above start editing outside the task — doc comments in neighbouring files, extra docs, an unasked CI job. That is a risk for roles that edit, which is why the implementer sits at medium and the lead, who does not edit, at high.

## What to delegate

Delegate work that is genuinely separable and sizeable: a feature you can specify end to end, an investigation spanning many files, a batch of mechanical edits.

Don't delegate what you can finish in a handful of tool calls, don't spawn several agents where one will do, and don't delegate to double-check yourself. Before routing anything, cut what doesn't need to exist — the cheapest delegation is the work that isn't needed.

Fresh context or a continued one depends on the role. The reviewer and the worker always start fresh: a reviewer that remembers the previous round judges the fix against its own earlier opinion, and a worker's spec is complete by definition. The implementer and the investigator are continued (SendMessage) for a follow-up on their own work — a fix to what the implementer just built, a second question about the area the investigator just read — while their prompt cache is warm: these two ship with a one-hour cache (`experimental.cacheTtl: 1h`), any other subagent gets 5 minutes. Past that window a resume rewrites the agent's whole accumulated prefix — measured median ~130K tokens — which costs more than a fresh spawn with a compact spec, so continue a cold agent only when its accumulated context is genuinely needed. A new topic always gets a new agent.

## Writing the spec

A subagent sees CLAUDE.md but not this conversation. Everything it needs travels in the prompt:

```
Goal: <one sentence>
User's words: <the user's request, verbatim, in quotes>
Files: in scope: <paths> / out of scope: <paths or "everything else">
Constraints: <what must not change, style, versions>
Definition of done: <exact command to run, or a verifiable check>
Report to: <a file path in the scratchpad, when the report may run past ~2,500 characters>
```

The **User's words** line is not decoration. A lead that paraphrases the request tends to narrow it, widen it, or resolve an ambiguity the user never resolved, and the subagent then builds the paraphrase. Quote the request; let the subagent see where your Goal and the user's words differ.

A bug gets its feedback loop before its fix. The investigator's Definition of done for a bug is the reproduction: the one command that goes red on *this* bug — a failing test, a replayed log, a script run against recorded data — with its output, or, when the test or script does not exist yet (the investigator cannot write to the repo), its full text and the failure it must show. Do not issue a fix spec without that reproduction. The fix spec carries it: the implementer first puts the test in place and shows it red for the stated reason — if it is not, they stop and report — then fixes until the same command is green; the test stays as the regression test. When the bug shows only on a device you cannot drive, build the loop from what the device left behind (logs, a recorded trace) instead of asking the user to try again; a trace recorded from a real device enters the repo only stripped of personal data. If no reproduction can be built, tell the user what was tried and get their go-ahead before a fix that nothing verifies.

Long reports travel as files. A report delivered as a message is cut off after a few thousand characters, and asking for it again costs a round trip. Name a scratchpad path in **Report to**; the agent writes the full report there and replies with the answer in a few lines plus the path.

Run delegations in parallel only when their file scopes are disjoint — at most one writer per file set. Overlapping scopes go sequentially.

## Review

Review crosses a role boundary: you review what an agent produced, or one agent reviews another's. That is the writer-verifier split, and it pays. Routine re-checking of your own work is not review — the model already verifies itself; the exception is code you were forced to author yourself, which deserves the independent reviewer any implementer's work would get.

Tell the reviewer exactly what the change is — a diff, a commit range, or a file list — and what to judge it against; a fresh context in a dirty worktree cannot guess where the change ends. Do not tell the reviewer who or what wrote it: a model that knows the author is a model of its own family grades more leniently.

Give the reviewer the spec along with the change and ask for two reads: what is wrong, and what is in the change that the spec does not ask for. The second read is the counterweight to the first — a review that only hunts defects pushes every round toward more code. A subagent reviewer always gets a **Report to** path, one of its own and not the path in the spec under review: a full review rarely fits in a message.

In the first round, whatever you review with, ask for everything it finds and filter afterwards — a reviewer told to report only the serious issues takes that literally and returns less.

The filtering is yours, and it is where review either pays or bloats the change. Fix a finding only when it names something concrete: an input or a sequence that can actually occur and what breaks when it does, a requirement of the spec the change does not meet, or something built that the spec did not ask for. A finding that names none of these — a theoretical race, a defensive branch for a state nothing produces — is declined with one line of why, not fixed.

Review rounds are bounded. The second round is a check, not a fresh hunt: give a new reviewer the first round's blocking findings and the updated change, and ask only whether each is fixed and whether the fix broke what it touched. A third round needs the user's go-ahead: a change that has not converged in two is usually over-built, and the remedy is to cut it, not to review it again.

## When a delegation fails

First check what actually failed: a rate limit, a turn cap or a tool error means retry as is — only a wrong or incomplete *result* means the spec was missing something. Never resend a spec that produced a wrong result unchanged; add what it lacked. If a subtask resists two repaired specs it was never mechanical: decide it yourself and hand down a spec precise enough to execute. Write the code yourself only when the task genuinely cannot be specified, and say why.

A hook that denies a command is policy, not a broken check. Do not split, rename or reroute the command to get past it; use the alternative the hook names, or report that the policy blocks the task.

## Compaction

Only part of this skill survives a context compact. After one, check whether you are still routing per the roster — if not, invoke senior-fable again before continuing.
