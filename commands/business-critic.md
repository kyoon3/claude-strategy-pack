---
description: Run the business-critic subagent — devil's-advocate pass over your business plan vs current market reality. Always opens a draft PR with the findings (either as direct edits to strategy docs when findings warrant it, or as a report file fallback). Cites sources, names falsifiers, not a cheerleader.
---

Dispatch the `business-critic` subagent.

Brief it like this:

> Run a critique pass. Scan for strategy docs (`BUSINESS.md` / `STRATEGY.md`, `ROADMAP.md` / `PLAN.md`, `TODO.md` + any `todo-*.md`, last ~100 lines of `SESSION_LOG.md` / `CHANGELOG.md`, any `pivot_*.md` or memory entries). Then do 1-3 live web searches from your rotation (pick angles you haven't used recently if you can tell from prior logs). Pick 3-4 critique dimensions to go deep on, not all seven. **Apply mode is on by default** — follow your Editing contract: apply each 🔴/🟡 finding as a direct edit to the strategy docs. If no 🔴/🟡 finding warrants a doc edit (everything was 🟢 or already addressed in the plan), do not force edits — just return the critique. Follow the output format in your system prompt exactly. End with one concrete next move and stop.

If the user passed extra args after `/business-critic`, append them as additional focus instructions (e.g., `/business-critic "focus on pricing"` → tell the subagent to weight monetization heavily).

Do not paraphrase or soften the subagent's findings — surface them verbatim to the user.

## Always open a draft PR

After the subagent returns its verbatim findings, drive the following git workflow — strictly in this order, stopping on the first failure:

1. **Preflight**: confirm the cwd is a git repo (`git rev-parse --show-toplevel`), `gh` is authenticated (`gh auth status`), and the default branch exists on the remote (`gh repo view --json defaultBranchRef -q .defaultBranchRef.name`). If any check fails, print the findings and tell the user the PR step was skipped + why. Do NOT attempt to authenticate on the user's behalf.
2. **Branch**: from a clean working tree, `git fetch origin <default-branch> --quiet` then `git checkout -b chore/business-critic-$(date +%Y-%m-%d-%H%M) origin/<default-branch>`. If the tree is dirty, `git stash push -u -m "business-critic-pre"` first and pop after the commit. If stash fails, abort and tell the user.
3. **Pick mode based on what the agent did:**
   - **Apply mode (preferred)** — if `git status --porcelain` shows changes, the agent applied findings as doc edits. Sanity-check: every changed file must be on the allowed surface (`BUSINESS.md` / `BUSINESS_PLAN.md` / `STRATEGY.md` / `ROADMAP.md` / `PLAN.md`, or `todo-*.md` under `.claude/rules/` | `docs/` | repo root scoped to business / launch / growth). If any file is off-surface, `git restore --staged --worktree <file>` it and warn the user.
   - **Report fallback** — if `git status --porcelain` is empty (no findings warranted doc edits, or the agent stayed conservative), create `reports/business-critic/YYYY-MM-DD-HHMM.md` containing a short header + the full verbatim critique. `git add reports/business-critic/...`. This guarantees the routine always produces a PR artifact.
4. **Commit**: `git commit -m "docs(business): apply business-critic findings $(date +%Y-%m-%d)"` for apply mode, or `git commit -m "chore: business-critic report $(date +%Y-%m-%d-%H%M)"` for report fallback. Do NOT bypass hooks with `--no-verify`. **Do NOT create, touch, or sign any review-marker files** — patterns like `.claude/.review-ok-*`, `.claude/.readability-ok-*`, `.claude/.dev-ok`, `*-approved`, `*-reviewed`, or anything that looks like a human-review attestation. Those markers exist precisely so an automated agent cannot self-approve its own diff; forging them invalidates the review gate. If the commit is gate-blocked, your only legitimate options are (a) work from a fresh git worktree at `.claude/worktrees/<short-name>/` (per the project's workflow rule), or (b) surface the blocker and stop. Looking for any other way around the gate is out of scope.
5. **Push + draft PR**: `git push -u origin <branch>` then `gh pr create --title "business-critic: <today's date> — <headline from the report>" --body "<the full critique report verbatim>" --base <default-branch> --draft`. Draft is the default — the user reviews before landing. Add the `business-critic` label if it exists; otherwise skip silently (`gh pr edit --add-label business-critic 2>/dev/null || true`).
6. **Surface the PR URL** to the user as the last line of your reply. Do not enable auto-merge.

If any step 1-5 fails, still print the subagent's findings and end with one line explaining why the PR step was skipped. **The PR is the artifact** — scheduled routines depend on it landing somewhere reviewable; never silently no-op.
