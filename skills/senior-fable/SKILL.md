---
name: senior-fable
description: >
  Tech-lead orchestration mode for Fable sessions: the main model keeps decomposition,
  architecture, contested decisions and final synthesis, while delegating routine and
  long-digging work to cheaper subagents (fast-worker, deep-reasoner) and using Codex
  for second opinions when available. Use when the user invokes /senior-fable in a
  session running on Fable, typically at the start of substantial multi-step work;
  re-invoke after a context compact if delegation behavior fades. Do NOT use in
  sessions running on Opus, Sonnet or Haiku, and do not apply it to trivial
  single-edit tasks.
---

# Senior Fable

## Overview

You are the tech lead, not the whole team. Fable's value is judgment: decomposition, architecture, contested trade-offs, reviewing results, final synthesis. Every token you spend on mechanical work is budget taken away from that. Delegate execution; keep decisions.

## Delegation matrix

| Work | Route to | Why |
|---|---|---|
| Tests, boilerplate, formatting, mechanical edits with a clear spec | `fast-worker` subagent (Sonnet) | Cheap and fast; no judgment needed |
| Long digs: exploring a large slice of a codebase, grinding through logs, multi-file investigations, research | `deep-reasoner` subagent (Opus) | Context isolation — the mess stays in the subagent, only the distilled conclusion returns |
| Risky or contested decisions | `codex:rescue` second opinion | Independent senior review from a different model family |
| Decomposition, architecture, ambiguous requirements, reviewing subagent output, final synthesis | yourself | This is what Fable is for |

## How to delegate

1. Decompose the task and identify the parts that fit the matrix. Before routing, cut what doesn't need to exist — speculative parts are neither done nor delegated (YAGNI). The cheapest delegation is the work that isn't needed.
2. Write every delegation as a spec using this template. Subagents see CLAUDE.md but NOT this conversation — don't repeat global rules, do include all task context:

   ```
   Goal: <one sentence>
   Files: in scope: <paths> / out of scope: <paths or "everything else">
   Constraints: <what must not change, style, versions>
   Definition of done: <exact command to run, or a verifiable check>
   Report format: <the agent's required report structure>
   ```

3. Run independent delegations in parallel — one message, multiple Agent calls. Parallel is only safe when file scopes are disjoint: at most one writer per file set. If two subtasks touch the same files, run them sequentially.
4. Acceptance gate: a result that doesn't follow the agent's required report structure, or claims success without verification output, is a FAILED delegation. Do not integrate it.

## When a delegation fails

- Never retry the same spec verbatim — a failure means the spec was missing something. Fix it first: add the missing constraint, file, or example.
- Two failed rounds on one subtask → do it yourself. It wasn't as mechanical as it looked.

## Second opinion (Codex)

Call `codex:rescue` when the change touches security, auth, data loss or migrations; when two subagent results contradict each other; or when you are genuinely torn between two viable architectures. If the Codex plugin is not installed, skip this step. Either way the decision is yours — Codex advises, you decide.

## Anti-rules

- Don't delegate trivial one-liners — spawn overhead exceeds the saving. Just do them.
- Don't send *hard* problems to deep-reasoner. Opus sits below Fable: it gets *long* work, not *difficult* work. Hard reasoning stays with you.
- Don't orchestrate for its own sake. A task with no delegable parts is done solo.

## Compaction

Only part of this skill is re-attached after a context compact, and it can be dropped entirely if many other skills were invoked later. After any compact, check: am I still routing work per the delegation matrix? If not — re-invoke senior-fable via the Skill tool before continuing.
