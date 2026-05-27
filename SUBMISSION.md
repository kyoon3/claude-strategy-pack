# Plugin Directory Submission — Status & Package

## Status (2026-05-27)

- **Prior submission (v0.2.0)**: returned `Published` but never propagated to the community `marketplace.json` — root cause was a manifest that failed `claude plugin validate` (the `repository` field was an object; the schema wants a string). The nightly sync silently rejected it.
- **Fixed**: `repository` is now a string; `claude plugin validate .` passes clean on `main` (v0.4.1). Ready to resubmit.

## Pre-submit checklist

- [x] `claude plugin validate .` → passed (both plugin.json + marketplace.json)
- [x] `main` is the default branch, repo is public
- [x] `.claude-plugin/plugin.json` is valid, version `0.4.1`
- [x] No open PRs; HEAD reflects the final 3-agent lineup

## Submission package

Form: <https://claude.ai/settings/plugins/submit> (or platform.claude.com/plugins/submit). Direct PRs to `anthropics/claude-plugins-community` are auto-closed — the form is the only ingestion path.

| Field | Value |
|---|---|
| Plugin repo URL | `https://github.com/kyoon3/claude-strategy-pack` |
| Default branch | `main` |
| Plugin manifest path | `.claude-plugin/plugin.json` |
| Plugin name | `strategy-pack` |
| Version | `0.4.1` |
| One-line description | Three strategy agents for solo founders — each opens a draft PR you review on your phone. The flow: run /business-critic first to pressure-test your plan against the live market (web-sourced devil's-advocate that edits your strategy docs when it finds a hole). Then run one of the two doc-keepers — /product-advisor for read-only alignment advice you act on yourself, or /pm to auto-reconcile your downstream docs (TODO, backlogs) with BUSINESS.md/ROADMAP.md. |
| Keywords | business, strategy, project-management, alignment, drift-detection, solo-founder, devils-advocate |
| License | MIT |
| Author / GitHub | kyoon3 |

## After submitting

- Wait 24–48h (the public catalog syncs nightly from the review pipeline).
- Verify it landed:
  ```bash
  curl -s https://raw.githubusercontent.com/anthropics/claude-plugins-community/main/.claude-plugin/marketplace.json \
    | jq '.plugins[] | select(.name == "strategy-pack")'
  ```
  Non-empty result = live. Others can then `/plugin install strategy-pack@claude-community`.

## If it still doesn't propagate within 48h

1. Open an issue at `anthropics/claude-plugins-community`:
   `Submission validated + "Published" but not in marketplace.json — kyoon3/claude-strategy-pack`
2. Include: submission timestamp, the `Published` notification, and the `curl | jq` output proving absence.

## This does not block your own usage

The taleweavers project runs `/business-critic` + `/product-advisor` + `/pm` via remote routines that load the **vendored** copy at `.claude/plugins/strategy-pack/` (see `session-start.sh`). Marketplace propagation only affects *discoverability for other people* — never your own routines. Treat it as nice-to-have, not a blocker.
