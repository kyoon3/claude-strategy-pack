---
description: Run the business-critic subagent — devil's-advocate pass over your business plan vs current market reality. Cites sources, names falsifiers, not a cheerleader. Pass `pr` to also apply 🔴/🟡 findings directly to the strategy docs and open a draft PR. Pass `report` for the legacy mode that writes the findings to a report file instead of editing docs.
---

Dispatch the `business-critic` subagent.

Brief it like this:

> Run a critique pass. Scan for strategy docs (`BUSINESS.md` / `STRATEGY.md`, `ROADMAP.md` / `PLAN.md`, `TODO.md` + any `todo-*.md`, last ~100 lines of `SESSION_LOG.md` / `CHANGELOG.md`, any `pivot_*.md` or memory entries). Then do 1-3 live web searches from your rotation (pick angles you haven't used recently if you can tell from prior logs). Pick 3-4 critique dimensions to go deep on, not all seven. Follow the output format in your system prompt exactly. End with one concrete next move and stop.

If the user passed extra args after `/business-critic`, append them as additional focus instructions (e.g., `/business-critic "focus on pricing"` → tell the subagent to weight monetization heavily).

Do not paraphrase or soften the subagent's findings — surface them verbatim to the user.

## Optional: apply findings + open a PR (`pr` arg)

If the user's args contain the literal token `pr` (e.g. `/business-critic pr`, `/business-critic pr focus on pricing`), tell the subagent **apply mode is on** and instruct it to follow its Editing contract: apply each 🔴/🟡 finding as a direct edit to the strategy docs, then open a draft PR with the critique report as the PR body. After the subagent returns its verbatim findings, drive the following git workflow — strictly in this order, stopping on the first failure:

1. **Preflight**: confirm the cwd is a git repo (`git rev-parse --show-toplevel`), `gh` is authenticated (`gh auth status`), and the default branch exists on the remote (`gh repo view --json defaultBranchRef -q .defaultBranchRef.name`). If any check fails, print the findings, skip the PR step, and tell the user why. Do NOT attempt to authenticate on the user's behalf.
2. **Branch**: from a clean working tree, `git fetch origin <default-branch> --quiet` then `git checkout -b chore/business-critic-$(date +%Y-%m-%d-%H%M) origin/<default-branch>`. If the tree is dirty, `git stash push -u -m "business-critic-pre"` first and pop after the commit. If stash fails, abort and tell the user.
3. **Confirm the agent already made the edits.** The subagent should have used `Edit` against the strategy-doc surface defined in its Editing contract before returning. Run `git status --porcelain` — if there are zero changes, the agent stayed read-only (e.g., no 🔴/🟡 findings warranted edits). In that case, skip steps 4-6 and tell the user "no edits — critique stands as-is."
4. **Sanity-check the diff.** Run `git diff --name-only` and confirm every changed file is on the allowed surface (`BUSINESS.md` / `BUSINESS_PLAN.md` / `STRATEGY.md` / `ROADMAP.md` / `PLAN.md`, or `todo-*.md` under `.claude/rules/` | `docs/` | repo root scoped to business / launch / growth). If any file is off-surface, `git restore --staged --worktree <file>` it and warn the user — the agent broke contract.
5. **Commit**: `git add` the on-surface files only, then `git commit -m "docs(business): apply business-critic findings $(date +%Y-%m-%d)"`. Do NOT bypass hooks with `--no-verify`.
6. **Push + draft PR**: `git push -u origin <branch>` then `gh pr create --title "business-critic: <today's date> — <headline from the report>" --body "<the full critique report verbatim>" --base <default-branch> --draft`. Draft is the default — the user reviews before landing.
7. **Surface the PR URL** to the user as the last line of your reply. Do not enable auto-merge.

If any step 1-6 fails, still print the subagent's findings and end with one line explaining why the PR step was skipped.

## Legacy: write the findings to a report file (`report` arg)

If the user's args contain the literal token `report` (e.g. `/business-critic report`), use the pre-apply-mode workflow instead: do not edit strategy docs; create `reports/business-critic/YYYY-MM-DD-HHMM.md` containing a short header + the full verbatim critique, commit it, push, and open a draft PR. Useful for scheduled runs where you want a persistent archive of the critique without touching the docs. Steps mirror the `pr` workflow above except step 3 writes the report file instead of expecting agent edits, and step 4's allowed surface is `reports/business-critic/*.md`.

`pr` and `report` are mutually exclusive. If the user passes both, treat `pr` as winning and tell them once that you ignored `report`.
