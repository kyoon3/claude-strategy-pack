---
description: Run the business-critic subagent — devil's-advocate pass over your business plan vs current market reality. Cites sources, names falsifiers, not a cheerleader. Pass `pr` as an arg to also open a GitHub PR with the findings as a persistent artifact (useful for scheduled routines).
---

Dispatch the `business-critic` subagent.

Brief it like this:

> Run a critique pass. Scan for strategy docs (`BUSINESS.md` / `STRATEGY.md`, `ROADMAP.md` / `PLAN.md`, `TODO.md` + any `todo-*.md`, last ~100 lines of `SESSION_LOG.md` / `CHANGELOG.md`, any `pivot_*.md` or memory entries). Then do 1-3 live web searches from your rotation (pick angles you haven't used recently if you can tell from prior logs). Pick 3-4 critique dimensions to go deep on, not all seven. Follow the output format in your system prompt exactly. End with one concrete next move and stop.

If the user passed extra args after `/business-critic`, append them as additional focus instructions (e.g., `/business-critic "focus on pricing"` → tell the subagent to weight monetization heavily).

Do not paraphrase or soften the subagent's findings — surface them verbatim to the user.

## Optional: open a GitHub PR with the findings

If the user's args contain the literal token `pr` (e.g. `/business-critic pr`, `/business-critic pr focus on pricing`), after the subagent returns its verbatim findings, ALSO do the following — strictly in this order, stopping on the first failure:

1. **Preflight**: confirm the current working directory is a git repo (`git rev-parse --show-toplevel`), `gh` is authenticated (`gh auth status`), and the repo's default branch exists on the remote (`gh repo view --json defaultBranchRef -q .defaultBranchRef.name`). If any check fails, print the findings as normal and tell the user the PR step was skipped + why. Do NOT attempt to authenticate on the user's behalf.
2. **Branch**: `git checkout -b chore/business-critic-$(date +%Y-%m-%d-%H%M)` from a clean working tree. If the tree is dirty, `git stash push -u -m "business-critic-pre"` first and pop after the commit. If stash fails, abort and tell the user.
3. **Write report**: create `reports/business-critic/YYYY-MM-DD-HHMM.md` (mkdir -p the parent). Contents = a short header (date, focus args if any) + the subagent's full verbatim output.
4. **Commit**: `git add reports/business-critic/...` then `git commit -m "chore: business-critic report YYYY-MM-DD HH:MM"`. Do NOT bypass hooks with `--no-verify`.
5. **Push + PR**: `git push -u origin <branch>` then `gh pr create --title "business-critic: <today's date>" --body "<one-line summary of the most important finding, then a fenced quote of the report's headline section>" --base <default-branch> --draft`. Draft is the default because the whole point is the user reviews before landing.
6. **Surface the PR URL** to the user as the last line of your reply. Do not enable auto-merge.

If any step fails, still print the subagent's findings and end with one line explaining why the PR step was skipped.
