# Plugin Directory Submission — Status & Resubmission

## Current status (2026-05-22)

- **Submission status received from Anthropic**: `Published`
- **Visible in `anthropics/claude-plugins-community/.claude-plugin/marketplace.json`**: **NO**
- **Verification command**:
  ```bash
  curl -s https://raw.githubusercontent.com/anthropics/claude-plugins-community/main/.claude-plugin/marketplace.json \
    | jq '.plugins[] | select(.source.url | test("kyoon3/claude-strategy-pack"))'
  ```
  Empty result confirms the plugin is not in the marketplace despite the `Published` status.

## Why direct PR is not an option

The community marketplace repo (`anthropics/claude-plugins-community`) explicitly states:

> Pull requests opened directly against this repo are closed automatically — all changes flow from the internal review pipeline.

The only ingestion path is the submission form: <https://clau.de/plugin-directory-submission>

## Resubmission package

Use these values when filling the form:

| Field | Value |
|---|---|
| Plugin repo URL | `https://github.com/kyoon3/claude-strategy-pack` |
| Default branch | `main` |
| Plugin manifest path | `.claude-plugin/plugin.json` |
| Plugin name | `strategy-pack` |
| Version | `0.2.0` (after `feat/pr-mode` merges; `0.1.0` currently on `main`) |
| One-line description | Two read-only agents for solo founders: `/business-critic` (daily devil's-advocate vs your plan, web-search evidence) + `/pm` (weekly cross-doc alignment auditor). |
| Keywords | business, strategy, project-management, alignment, drift-detection, solo-founder, devils-advocate |
| License | MIT |
| Author / GitHub | kyoon3 |

## If resubmission still doesn't propagate within 48h

1. Open an issue at `anthropics/claude-plugins-community` with title:
   `Submission "Published" but not propagated to marketplace.json — kyoon3/claude-strategy-pack`
2. Reference any prior known propagation issues (search the repo's issue tracker for "Published not in marketplace" or "propagation").
3. Include: submission timestamp, the `Published` notification screenshot/email, and the verification `curl | jq` output proving absence.

## Why this matters for the routines

The taleweavers project schedules `/business-critic` daily and `/pm` weekly via remote routines. The routines work even now because we **vendor** the plugin locally at `.claude/plugins/strategy-pack/` (see `taleweavers/scripts/update-plugins.sh` and `.claude/hooks/session-start.sh`). Marketplace propagation only affects *discoverability* for other users — not our own usage. Treat the propagation as a nice-to-have, not a blocker.
