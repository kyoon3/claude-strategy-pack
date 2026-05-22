---
description: Run the project-pm subagent — cross-domain alignment sync across strategy / roadmap / per-domain backlogs / plans / API docs. Surfaces inconsistencies, orphaned commitments, and proposes follow-up plans without editing docs. Pass `pr` as an arg to also open a GitHub PR with the alignment report (useful for scheduled routines).
---

Dispatch the `project-pm` subagent.

Brief it like this:

> Run a cross-domain alignment sync. Scan for `BUSINESS.md` / `STRATEGY.md`, `ROADMAP.md`, `TODO.md`, last ~100 lines of `SESSION_LOG.md` / `CHANGELOG.md`, then any per-domain backlog under `.claude/rules/`, `docs/`, or repo root (e.g., `todo-*.md`, `BACKLOG-*.md`). Pull the 5 most recently modified plan / spec files from `docs/plans/`, `docs/superpowers/plans/`, `plans/`, or `.specs/`. Check API contract docs (`docs/api.md`, `openapi.yaml`, etc.). Pull `git log --oneline -20` and `git log --since='2 weeks ago' --name-only` over docs paths and the main API source file for drift signal. Pick 3-5 alignment dimensions (A-H) to go deep on, not all eight. Follow the output format in your system prompt exactly. End with one suggested next move and stop.

If the user passed extra args after `/pm`, append them as additional focus instructions (e.g., `/pm "focus on Instagram integration"` → weight cross-domain prerequisite checks on that initiative).

Do not paraphrase or soften the subagent's findings — surface them verbatim. The alignment table and proposed-task blocks are meant to be copy-pasted by the user, so preserve formatting.

## Optional: open a GitHub PR with the alignment report

If the user's args contain the literal token `pr` (e.g. `/pm pr`, `/pm pr focus on Instagram`), after the subagent returns its verbatim findings, ALSO do the following — strictly in this order, stopping on the first failure:

1. **Preflight**: confirm the current working directory is a git repo (`git rev-parse --show-toplevel`), `gh` is authenticated (`gh auth status`), and the repo's default branch exists on the remote (`gh repo view --json defaultBranchRef -q .defaultBranchRef.name`). If any check fails, print the findings as normal and tell the user the PR step was skipped + why. Do NOT attempt to authenticate on the user's behalf.
2. **Branch**: `git checkout -b chore/pm-$(date +%Y-%m-%d-%H%M)` from a clean working tree. If the tree is dirty, `git stash push -u -m "pm-pre"` first and pop after the commit. If stash fails, abort and tell the user.
3. **Write report**: create `reports/pm/YYYY-MM-DD-HHMM.md` (mkdir -p the parent). Contents = a short header (date, focus args if any) + the subagent's full verbatim output, **including the alignment table and proposed-task blocks intact**.
4. **Commit**: `git add reports/pm/...` then `git commit -m "chore: pm alignment report YYYY-MM-DD HH:MM"`. Do NOT bypass hooks with `--no-verify`.
5. **Push + PR**: `git push -u origin <branch>` then `gh pr create --title "pm: <today's date>" --body "<count of inconsistencies surfaced + the most urgent proposed follow-up>" --base <default-branch> --draft`. Draft is the default because the proposed follow-ups need user adjudication before they land.
6. **Surface the PR URL** to the user as the last line of your reply. Do not enable auto-merge.

If any step fails, still print the subagent's findings and end with one line explaining why the PR step was skipped.
