<h1 align="center">Senior Fable</h1>

<p align="center">
  <em>The expensive model decides. The cheap models type.</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Claude%20Code-plugin-111111?style=flat-square" alt="Claude Code plugin">
  <img src="https://img.shields.io/github/v/release/AndyShaman/senior-fable?style=flat-square&color=111111" alt="Release">
  <img src="https://img.shields.io/badge/license-MIT-111111?style=flat-square" alt="MIT license">
</p>

**Senior Fable** is a Claude Code plugin that turns your top-tier model (Claude Fable 5) into a **tech lead**: it keeps decomposition, architecture, contested decisions and final synthesis — and delegates everything else to cheaper subagents. Mechanical work goes to a Sonnet **fast-worker**, long context-heavy investigations go to a read-only Opus **deep-reasoner**, and risky decisions get a second opinion from **Codex** (if installed).

The result: your most expensive tokens are spent only where top-tier judgment actually matters, while routine execution and codebase digging run on models that cost a fraction. Token savings without quality loss — and often with quality *gains*, because the messy exploration stays out of the main context window.

## Why

Running a frontier model on boilerplate is like having your staff engineer format JSON. Two things are wasted:

1. **Tokens / usage limits.** Every test file, rename and formatting pass burns top-tier budget.
2. **Context.** Reading 40 files to find one bug pollutes the main conversation. After a compact, the reasoning that mattered may be gone.

Senior Fable fixes both with an orchestrator–worker pattern: subagents run in their own context windows and return only distilled conclusions.

## How it works

| Work | Goes to | Why |
|---|---|---|
| Tests, boilerplate, formatting, mechanical edits with a clear spec | `fast-worker` (Sonnet) | Cheap and fast; no judgment needed |
| Long digs: exploring a codebase slice, grinding logs, multi-file investigations | `deep-reasoner` (Opus, **read-only**) | Context isolation — the mess stays in the subagent |
| Risky or contested decisions | `codex:rescue` (optional) | Independent review from a different model family |
| Decomposition, architecture, reviewing results, final synthesis | the main model | This is what you pay top tier for |

Determinism is enforced, not requested:

- **Least-privilege tools.** `deep-reasoner` is denied `Write`/`Edit` at the harness level — it physically cannot modify files. `fast-worker` gets exactly six tools: no web, no spawning its own agents.
- **Spec template.** Every delegation ships as Goal / Files in-and-out of scope / Constraints / Definition of done / Report format. Subagents never see your conversation, so specs are self-contained.
- **Acceptance gate.** A result without the required report structure or without verification output is a failed delegation and is not integrated.
- **Failure policy.** Never retry the same spec verbatim; two failed rounds means the lead does it personally.
- **One writer per file set.** Parallel delegations must have disjoint file scopes — no edit races.
- **YAGNI at decomposition.** The cheapest delegation is the work that isn't needed: speculative parts are cut before routing, not built.

## Install

```
/plugin marketplace add AndyShaman/senior-fable
/plugin install senior-fable@senior-fable
```

Restart the session, switch to Fable (`/model fable`), then activate:

```
/senior-fable
```

Give it substantial multi-step work and watch the panel: purple is the deep-reasoner digging, blue is the fast-worker typing.

## Requirements & notes

- **Model:** built for sessions running on Claude Fable 5 (the strongest tier). On Opus/Sonnet sessions the skill deliberately does not apply — delegating "up" makes no sense.
- **Codex is optional.** If the [OpenAI Codex plugin](https://github.com/openai/codex-plugin-cc) is installed, contested decisions get an independent second opinion; if not, that step is skipped automatically.
- **Billing:** on subscription plans this saves your top-tier usage limits; on API billing it saves money directly.
- **Compaction-safe.** The skill is deliberately small enough to be re-attached whole after a context compact, and instructs the model to re-invoke itself if delegation behavior fades.

## What's inside

```
skills/senior-fable/SKILL.md   # the tech-lead playbook: matrix, spec template, gates
agents/deep-reasoner.md        # Opus investigator — read-only, returns distilled findings
agents/fast-worker.md          # Sonnet executor — verified mechanical work, shortest diff
```

## Related work

- [wshobson/agents](https://github.com/wshobson/agents) — the canonical per-agent model-tier marketplace that inspired the routing idea.

## Keywords

Claude Code plugin · multi-agent orchestration · subagents · token optimization · model routing · Claude Fable 5 · Opus · Sonnet · tech-lead pattern · orchestrator–worker · context isolation · AI pair programming · agent delegation

## License

MIT © [Andy Shaman](https://github.com/AndyShaman)
