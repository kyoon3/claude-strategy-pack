---
description: Run the product-advisor subagent — read-only cross-domain alignment advice across strategy / roadmap / per-domain backlogs / plans / API docs. Surfaces inconsistencies, orphaned commitments, and proposes follow-ups without editing docs. Always opens a draft PR with the advice as a report file. For actually reconciling the docs (apply-mode), use /pm instead.
---

Dispatch the `product-advisor` subagent.

Brief it like this:

> Run a cross-domain alignment sync. Scan for `BUSINESS.md` / `STRATEGY.md`, `ROADMAP.md`, `TODO.md`, last ~100 lines of `SESSION_LOG.md` / `CHANGELOG.md`, then any per-domain backlog under `.claude/rules/`, `docs/`, or repo root (e.g., `todo-*.md`, `BACKLOG-*.md`). Pull the 5 most recently modified plan / spec files from `docs/plans/`, `docs/superpowers/plans/`, `plans/`, or `.specs/`. Check API contract docs (`docs/api.md`, `openapi.yaml`, etc.). Pull `git log --oneline -20` and `git log --since='2 weeks ago' --name-only` over docs paths and the main API source file for drift signal. Pick 3-5 alignment dimensions (A-H) to go deep on, not all eight. Follow the output format in your system prompt exactly. End with one suggested next move and stop. You are READ-ONLY — do not edit any doc; propose, don't apply.

If the user passed extra args after `/product-advisor`, append them as additional focus instructions (e.g., `/product-advisor "focus on Instagram integration"` → weight cross-domain prerequisite checks on that initiative).

Do not paraphrase or soften the subagent's findings — surface them verbatim. The alignment table and proposed-task blocks are meant to be copy-pasted by the user, so preserve formatting.

## Always open a draft PR with the advice (report file)

product-advisor is read-only, so the artifact is a **report file**, never doc edits. After the subagent returns its verbatim findings, drive the following git workflow — strictly in this order, stopping on the first failure:

1. **Preflight**: confirm the cwd is a git repo (`git rev-parse --show-toplevel`), `gh` is authenticated (`gh auth status`), and the default branch exists on the remote (`gh repo view --json defaultBranchRef -q .defaultBranchRef.name`). If any check fails, print the findings and tell the user the PR step was skipped + why. Do NOT attempt to authenticate on the user's behalf.
2. **Branch**: from a clean working tree, `git fetch origin <default-branch> --quiet` then `git checkout -b chore/product-advisor-$(date +%Y-%m-%d-%H%M) origin/<default-branch>`. If the tree is dirty, `git stash push -u -m "advisor-pre"` first and pop after the commit. If stash fails, abort and tell the user.
3. **Write report**: create `reports/product-advisor/YYYY-MM-DD-HHMM.md` (mkdir -p the parent). Contents = a short header (date, focus args if any) + the subagent's full verbatim output, **including the alignment table and proposed-task blocks intact**.
4. **Commit**: `git add reports/product-advisor/...` then `git commit -m "docs: product-advisor alignment report $(date +%Y-%m-%d-%H%M)"`. Do NOT bypass hooks with `--no-verify`. If a project commit gate needs a review marker (e.g. a pre-commit hook), satisfy it the project-documented way; if you can't, surface the blocker and stop.
5. **Push + draft PR**: `git push -u origin <branch>` then `gh pr create --title "product-advisor: <today's date> — <headline>" --body "<the full verbatim report>" --base <default-branch> --draft`. Draft is the default — the advice needs user adjudication before any of it lands. Add the `product-advisor` label if it exists; otherwise skip silently (`gh pr edit --add-label product-advisor 2>/dev/null || true`).
6. **Surface the PR URL** to the user as the last line of your reply. Do not enable auto-merge.

If any step 1-5 fails, still print the subagent's findings and end with one line explaining why the PR step was skipped. **The PR is the artifact** — scheduled routines depend on it landing somewhere reviewable; never silently no-op.
