---
description: Full alignment cycle in one run — diagnoses with product-advisor (read-only), then applies with pm (edits the TODO/backlog), producing ONE draft PR. This is the daily driver. To debug a single stage, run /product-advisor (diagnosis only) or /pm (apply only) on their own.
---

Run the full alignment cycle in one invocation. Internally this is two stages — `product-advisor` (diagnose, read-only) then `pm` (apply) — kept as separate subagents so you can rerun either alone to isolate a problem. But this command produces a **single draft PR** containing both the diagnosis and the applied edits.

Pass any extra args (e.g. `/align "focus on the API contract"`) through to both stages as focus instructions.

## Stage 1 — diagnose (`product-advisor`, read-only)

Dispatch the `product-advisor` subagent with its standard brief: scan `BUSINESS.md`/`STRATEGY.md`, `ROADMAP.md`, `TODO.md` + any `todo-*.md`/`BACKLOG-*.md`, recent `SESSION_LOG.md`, in-flight plans, API docs, and `git log` drift signal; pick 3-5 alignment dimensions; produce its alignment report in the format from its system prompt. Capture the full verbatim report.

**Write the report to `reports/product-advisor/YYYY-MM-DD-HHMM.md`** (mkdir -p the parent) in the working tree. Do NOT open a PR for it on its own — it is the input to stage 2. product-advisor must not edit any doc (read-only).

## Stage 2 — apply (`pm`)

Dispatch the `pm` subagent. It reads the report stage 1 just wrote (its input #2), treats the open 🔴/🟡 findings as a worklist, and applies every fix that is a TODO/backlog edit — plus reconciles the TODO/backlog family against `BUSINESS.md`/`ROADMAP.md`. It never edits source-of-truth docs (`BUSINESS.md`/`ROADMAP.md`) or plan/spec files. Capture pm's verbatim reconciliation report (it cites which advisor findings it applied vs deferred).

## One draft PR (both stages on one branch)

Drive this git workflow — stop on the first failure:

1. **Preflight**: cwd is a git repo (`git rev-parse --show-toplevel`), `gh` authenticated (`gh auth status`), default branch exists on remote. If any fails, print both reports and say the PR step was skipped + why. Do NOT authenticate on the user's behalf.
2. **Branch**: from a clean tree, `git fetch origin <default-branch> --quiet` then `git checkout -b chore/align-$(date +%Y-%m-%d-%H%M) origin/<default-branch>`. If dirty, `git stash push -u -m align-pre` first, pop after commit.
3. **Stage the artifacts**: the stage-1 report file under `reports/product-advisor/` AND the stage-2 edits. Sanity-check the edits: every changed file outside `reports/` must be a TODO/backlog doc (`TODO.md`, `todo-*.md`/`BACKLOG-*.md`, API-contract notes, `SESSION_LOG.md`). It must NOT include a source-of-truth doc (`BUSINESS.md`/`ROADMAP.md`/`STRATEGY.md`/`PLAN.md`), a plan/spec file (`docs/plans/`, `docs/superpowers/plans/`, `plans/`, `.specs/`), or any code/test/hook. `git restore --staged --worktree` any offender and warn — pm broke contract.
4. **Commit**: `git commit -m "docs: align cycle $(date +%Y-%m-%d) — diagnose + apply"`. Do NOT use `--no-verify`. If a project commit gate needs a review marker, satisfy it the project-documented way; if you can't, surface the blocker and stop.
5. **Push + draft PR**: `git push -u origin <branch>` then `gh pr create --title "align: <today's date> — <pm headline>" --body "<pm's reconciliation report verbatim>" --base <default-branch> --draft`. Add the `align` label if it exists (`gh pr edit --add-label align 2>/dev/null || true`).
6. **Surface the PR URL** as the last line. Do not merge or enable auto-merge.

If stage 2 applied no edits (everything already aligned), the PR still lands with just the stage-1 advisor report as the artifact — the cycle never silently no-ops.

Surface both subagents' verbatim output to the user, then the PR URL.
