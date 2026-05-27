---
description: Run the pm subagent — apply-mode doc reconciler. Reads the source-of-truth docs (BUSINESS.md / ROADMAP.md, or a doc you name) and edits the downstream docs (TODO, backlogs, plan stubs, API notes) so they agree with the source. Always opens a draft PR with the edits. For read-only advice instead of edits, use /product-advisor.
---

Dispatch the `pm` subagent.

Brief it like this:

> Run a source-of-truth reconciliation. Treat `BUSINESS.md` / `STRATEGY.md` and `ROADMAP.md` / `PLAN.md` as authoritative (unless I name a different source below). Read them first, then scan the downstream docs: `TODO.md`, any `todo-*.md` / `BACKLOG-*.md` under `.claude/rules/` `docs/` or repo root, the 5 most recently modified plan/spec files, and API-contract docs. Pull `git log --oneline -20` + `git log --since='2 weeks ago' --name-only` over docs paths for drift signal. Follow your Reconciliation contract: diagnose first (produce the report), then EDIT the downstream docs so they agree with the source — never edit the source docs, never touch code/tests/hooks. Pick the dimensions where the source↔downstream gap is widest. If nothing needs reconciling, return the report without forcing edits. Follow the output format in your system prompt exactly.

If the user passed extra args after `/pm`, append them as additional focus instructions (e.g., `/pm "source is docs/spec.md"` → treat that file as the source of truth; `/pm "focus on the API contract"` → weight dimension that heaviest).

Do not paraphrase or soften the subagent's findings — surface them verbatim to the user along with the PR URL.

## Always open a draft PR

After the subagent returns its verbatim report, drive the following git workflow — strictly in this order, stopping on the first failure:

1. **Preflight**: confirm the cwd is a git repo (`git rev-parse --show-toplevel`), `gh` is authenticated (`gh auth status`), and the default branch exists on the remote (`gh repo view --json defaultBranchRef -q .defaultBranchRef.name`). If any check fails, print the report and tell the user the PR step was skipped + why. Do NOT attempt to authenticate on the user's behalf.
2. **Branch**: from a clean working tree, `git fetch origin <default-branch> --quiet` then `git checkout -b chore/pm-$(date +%Y-%m-%d-%H%M) origin/<default-branch>`. If the tree is dirty, `git stash push -u -m "pm-pre"` first and pop after the commit. If stash fails, abort and tell the user.
3. **Pick mode based on what the agent did:**
   - **Reconcile mode (preferred)** — if `git status --porcelain` shows changes, the agent edited downstream docs. Sanity-check: every changed file must be a downstream doc (`TODO.md`, `todo-*.md` / `BACKLOG-*.md`, plan/spec stubs, API-contract notes, or `SESSION_LOG.md`). **It must NOT include any source-of-truth doc** (`BUSINESS.md` / `ROADMAP.md` / `STRATEGY.md` / `PLAN.md`) unless the user explicitly named it as a target, nor any code/test/hook file. If a forbidden file changed, `git restore --staged --worktree <file>` it and warn the user — the agent broke contract.
   - **Report fallback** — if `git status --porcelain` is empty (everything already agreed), create `reports/pm/YYYY-MM-DD-HHMM.md` with a short header + the full verbatim report. `git add reports/pm/...`. This guarantees the routine always produces a PR artifact.
4. **Commit**: `git commit -m "docs: pm reconcile downstream docs to source $(date +%Y-%m-%d)"` for reconcile mode, or `git commit -m "docs: pm reconciliation report $(date +%Y-%m-%d-%H%M)"` for report fallback. Do NOT bypass hooks with `--no-verify`. If a project commit gate needs a review marker, satisfy it the project-documented way; if you can't, surface the blocker and stop.
5. **Push + draft PR**: `git push -u origin <branch>` then `gh pr create --title "pm: <today's date> — <headline from the report>" --body "<the full verbatim report>" --base <default-branch> --draft`. Draft is the default — reconciliations need user adjudication before they land. Add the `pm` label if it exists; otherwise skip silently (`gh pr edit --add-label pm 2>/dev/null || true`).
6. **Surface the PR URL** to the user as the last line of your reply. Do not enable auto-merge.

If any step 1-5 fails, still print the subagent's report and end with one line explaining why the PR step was skipped. **The PR is the artifact** — scheduled routines depend on it landing somewhere reviewable; never silently no-op.
