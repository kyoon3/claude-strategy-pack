---
description: Run the business-critic subagent — devil's-advocate pass over your business plan vs current market reality. Cites sources, names falsifiers, not a cheerleader.
---

Dispatch the `business-critic` subagent.

Brief it like this:

> Run a critique pass. Scan for strategy docs (`BUSINESS.md` / `STRATEGY.md`, `ROADMAP.md` / `PLAN.md`, `TODO.md` + any `todo-*.md`, last ~100 lines of `SESSION_LOG.md` / `CHANGELOG.md`, any `pivot_*.md` or memory entries). Then do 1-3 live web searches from your rotation (pick angles you haven't used recently if you can tell from prior logs). Pick 3-4 critique dimensions to go deep on, not all seven. Follow the output format in your system prompt exactly. End with one concrete next move and stop.

If the user passed extra args after `/business-critic`, append them as additional focus instructions (e.g., `/business-critic "focus on pricing"` → tell the subagent to weight monetization heavily).

Do not paraphrase or soften the subagent's findings — surface them verbatim to the user.
