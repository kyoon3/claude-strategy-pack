# claude-strategy-pack

Two read-only Claude Code subagents for solo founders and small teams whose strategy lives in distributed markdown docs:

- **`business-critic`** — daily devil's-advocate against your business plan. Re-examines positioning, activation gates, monetization mechanics, and competitive substitution using live web search. Every doubt cites a URL or a falsifiable mechanism. Not a cheerleader.
- **`project-pm`** — weekly cross-document alignment sync. Reads `BUSINESS.md` / `ROADMAP.md` / `TODO.md` / per-domain backlogs / in-flight plan files / API contract docs and surfaces inconsistencies, orphaned commitments, blocked-by mismatches, and stale plans. Proposes follow-up plan stubs but never edits your docs.

Both are **read-only**. They produce reports you act on; they never silently rewrite your strategy.

## Why this exists

If you are a solo founder or 2-3 person team, two failure modes show up over and over:

1. **You stop pushing back on your own plan.** Confirmation bias eats your forecasts. → `business-critic`
2. **Your strategy docs, sprint backlogs, and in-flight plans drift apart.** ROADMAP commits to X, no one owns X in any TODO, frontend backlog has half of X under a different name. → `project-pm`

Both agents are designed to be invoked **on a cadence** — daily for the critic, weekly for the PM — not just on demand. Combine with the `schedule` or `loop` skills to automate.

## Installation

### Via plugin install (recommended)

```
/plugin install github.com/kyoon3/claude-strategy-pack
```

Restart your Claude Code session after install so the subagents and slash commands are picked up.

### Manual (drop-in)

Clone this repo and copy `agents/*.md` + `commands/*.md` into your project's `.claude/agents/` and `.claude/commands/`. Restart the session.

## Usage

```
/business-critic                 # daily critique pass
/business-critic "focus pricing" # weight a specific dimension

/pm                              # weekly alignment sync
/pm "focus on <initiative>"      # weight a cross-domain initiative
```

Both commands accept an optional focus arg.

### Recommended cadence

| Agent | Cadence | Trigger |
|---|---|---|
| `business-critic` | Daily (or weekday morning) | `/loop 1d /business-critic` or scheduled via `schedule` skill |
| `project-pm` | Weekly (Monday morning) | Manual or `/loop 7d /pm` |
| Both | Before any phase transition | Manual one-off |

## What the agents expect to find

The agents adapt to your repo layout but scan for these typical paths. None are required — missing files are skipped silently.

**Strategy layer**: `BUSINESS.md`, `STRATEGY.md`, `ROADMAP.md`, `PLAN.md`, `TODO.md`

**Recent activity**: `SESSION_LOG.md`, `CHANGELOG.md`, `decisions/` directory

**Domain backlogs** (anywhere in `.claude/rules/`, `docs/`, or repo root): `todo-*.md`, `BACKLOG-*.md`

**In-flight plans**: `docs/plans/`, `docs/superpowers/plans/`, `plans/`, `.specs/`

**API contracts**: `docs/api.md`, `openapi.yaml`, `schema.graphql`

**Memory / pivots**: `memory/MEMORY.md`, `.claude/memory/`, `project_*.md`, `pivot_*.md`

If you keep strategy somewhere unusual, the agents will ask once and remember (or you can amend their system prompts under `agents/`).

## What they do NOT do

- They never edit any doc, plan, or backlog file. They produce reports.
- `business-critic` does not propose features or implementation. It evaluates the plan.
- `project-pm` does not critique the business or the code. It only reconciles documents.
- Neither agent invents statistics. If a number is cited, a URL is cited with it.

## Pairing

Both agents are designed to refer to each other:

- `business-critic` flags non-business findings as `Out-of-scope-but-noticed → project-pm`
- `project-pm` flags business concerns as `Out-of-scope-but-noticed → business-critic`

This keeps each agent focused while letting the other catch what falls out of scope.

## Configuration

No configuration is required. Both agents read your docs and adapt.

If you want to customize critique dimensions, web-search rotation, or output verbosity, edit `agents/business-critic.md` or `agents/project-pm.md` directly — they are pure markdown prompts.

## License

MIT. See [LICENSE](LICENSE).

## Contributing

Issues and PRs welcome at https://github.com/kyoon3/claude-strategy-pack. The agents are intentionally generic — please keep project-specific examples in your own fork rather than upstreaming them.
