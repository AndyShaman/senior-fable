<h1 align="center">Senior Fable</h1>

<p align="center">
  <em>The expensive model decides. The cheap models type.</em>
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Claude%20Code-plugin-111111?style=flat-square" alt="Claude Code plugin">
  <img src="https://img.shields.io/github/v/release/AndyShaman/senior-fable?style=flat-square&color=111111" alt="Release">
  <img src="https://img.shields.io/badge/license-MIT-111111?style=flat-square" alt="MIT license">
</p>

**Senior Fable** is a Claude Code plugin that turns your top-tier session model into a **tech lead**: it keeps decomposition, architecture, contested decisions and final synthesis — and routes everything else to a roster of cheaper subagents. Feature-sized coding goes to an **implementer**, mechanical work to a **worker**, long investigations to a read-only **investigator**, and finished work gets an independent **reviewer**.

The result: your most expensive tokens are spent only where top-tier judgment matters, and the messy exploration stays out of the main context window.

## Why

Running a frontier model on boilerplate is like having your staff engineer format JSON. Two things are wasted:

1. **Tokens / usage limits.** Every test file, rename and formatting pass burns top-tier budget.
2. **Context.** Reading 40 files to find one bug pollutes the main conversation. After a compact, the reasoning that mattered may be gone.

Senior Fable fixes both with an orchestrator–worker pattern: subagents run in their own context windows and return only distilled conclusions.

## The roster

Roles, not model names. The plugin ships working defaults; every tier is yours to override.

| Role | Work | Ships as |
|---|---|---|
| **lead** | decomposition, architecture, reviewing results, final synthesis | your session model |
| **implementer** | code where decisions live inside the task | `implementer` (opus) |
| **worker** | tests to a spec, boilerplate, formatting, renames | `fast-worker` (sonnet) |
| **investigator** | long digs that return a conclusion, not a dump | `deep-reasoner` (opus, no file editing) |
| **reviewer** | independent review of finished work | `reviewer` (opus, no file editing) — or any different-family reviewer you have (e.g. a Codex CLI plugin) |

Design decisions the skill enforces:

- **Writer–verifier split.** Review crosses a role boundary: the reviewer never saw the reasoning that produced the change, so it judges the result on its own terms. Routine self-re-checking is explicitly *not* review — Claude 5 generation models already verify their own work.
- **No file editing for observer roles.** `deep-reasoner` and `reviewer` have `Write`/`Edit` denied at the harness level and are instructed to change nothing — the same pattern Claude Code's built-in Explore agent uses. (Honest caveat: `Bash` stays available, so this is a strong guardrail, not a cryptographic guarantee.)
- **Self-contained specs.** Subagents see CLAUDE.md but not your conversation, so every delegation carries its goal, file scope, constraints and definition of done.
- **Failure triage.** A transient failure (rate limit, turn cap) means retry; only a wrong result means the spec was missing something — and a spec that produced a wrong result is never resent unchanged.
- **Delegation restraint.** Claude 5 models delegate readily; the skill spends its words on when *not* to spawn: nothing that fits in a handful of tool calls, no fleets where one agent will do, no delegating to double-check yourself.

## Make it your own

Two knobs, no forking:

**1. Roster override.** Add a `## Senior Fable roster` block to your user or project CLAUDE.md — it outranks the plugin's defaults. Format: one `role: model` line per role you want to change; unlisted roles keep their defaults. Model values are anything Claude Code accepts — an alias (`haiku`, `sonnet`, `opus`), a full model ID, or a tool you route to (e.g. a different-family CLI for `reviewer`).

```markdown
## Senior Fable roster
worker: haiku
reviewer: codex (via the Codex CLI plugin), effort high
```

**2. Effort.** Often the better cost lever than switching tiers: a stronger model at low effort can beat a weaker one at high effort, for less. Plugin agents support an `effort:` frontmatter field — set it per role, or note it in your roster block.

One warning: the `CLAUDE_CODE_SUBAGENT_MODEL` environment variable overrides both frontmatter and per-call model choices. Set globally, it silently collapses the entire roster onto one model.

## Install

```
/plugin marketplace add AndyShaman/senior-fable
/plugin install senior-fable@senior-fable
```

Restart the session, make sure you're on your top-tier model, then activate:

```
/senior-fable
```

Give it substantial multi-step work and watch the panel: purple is the investigator digging, green is the implementer building, blue is the worker typing, red is the reviewer judging.

## Requirements & notes

- **Model:** built for sessions running your strongest tier (Claude Fable 5, or Opus 5 if that's your top). On your cheapest model the skill deliberately does not apply — there is nothing below it to route to.
- **A second model family is optional.** If you have an external reviewer (such as the OpenAI Codex plugin), point the reviewer role at it for a cross-family opinion; otherwise the bundled read-only `reviewer` covers the writer–verifier split in a fresh context.
- **Billing:** on subscription plans this saves your top-tier usage limits; on API billing it saves money directly.
- **Compaction-safe.** The skill is small enough to be re-attached whole after a context compact, and instructs the model to re-invoke itself if delegation behavior fades.

## What's inside

```
skills/senior-fable/SKILL.md   # the tech-lead playbook: roster, spec shape, review boundary
agents/implementer.md          # feature-sized coding from a spec — flags defects instead of building them
agents/fast-worker.md          # mechanical execution — verified, shortest working diff
agents/deep-reasoner.md        # long investigations — no file editing, returns distilled findings
agents/reviewer.md             # independent review — no file editing, reports everything, sorted by severity
```

## Design notes (v1.1)

Reworked for Claude 5 generation models following Anthropic's July 2026 context-engineering guidance: rules the model no longer needs were removed (mandatory verification steps, report-format gates, rigid failure procedures), and what remains is mechanics the model cannot infer — model resolution order, billing tiers, harness behavior — plus interfaces enforced by tool permissions rather than prose.

## Related work

- [wshobson/agents](https://github.com/wshobson/agents) — the canonical per-agent model-tier marketplace that inspired the routing idea.

## Keywords

Claude Code plugin · multi-agent orchestration · subagents · token optimization · model routing · Claude Fable 5 · Opus · Sonnet · tech-lead pattern · orchestrator–worker · writer–verifier · context isolation · agent delegation

## License

MIT © [Andy Shaman](https://github.com/AndyShaman)
