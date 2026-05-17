---
description: Run the project-pm subagent — cross-domain alignment sync across strategy / roadmap / per-domain backlogs / plans / API docs. Surfaces inconsistencies, orphaned commitments, and proposes follow-up plans without editing docs.
---

Dispatch the `project-pm` subagent.

Brief it like this:

> Run a cross-domain alignment sync. Scan for `BUSINESS.md` / `STRATEGY.md`, `ROADMAP.md`, `TODO.md`, last ~100 lines of `SESSION_LOG.md` / `CHANGELOG.md`, then any per-domain backlog under `.claude/rules/`, `docs/`, or repo root (e.g., `todo-*.md`, `BACKLOG-*.md`). Pull the 5 most recently modified plan / spec files from `docs/plans/`, `docs/superpowers/plans/`, `plans/`, or `.specs/`. Check API contract docs (`docs/api.md`, `openapi.yaml`, etc.). Pull `git log --oneline -20` and `git log --since='2 weeks ago' --name-only` over docs paths and the main API source file for drift signal. Pick 3-5 alignment dimensions (A-H) to go deep on, not all eight. Follow the output format in your system prompt exactly. End with one suggested next move and stop.

If the user passed extra args after `/pm`, append them as additional focus instructions (e.g., `/pm "focus on Instagram integration"` → weight cross-domain prerequisite checks on that initiative).

Do not paraphrase or soften the subagent's findings — surface them verbatim. The alignment table and proposed-task blocks are meant to be copy-pasted by the user, so preserve formatting.
