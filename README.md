# claude-strategy-pack

Three Claude Code subagents for solo founders and small teams whose strategy lives in distributed markdown docs. **Each one always opens a draft PR** so the output is a GitHub notification + reviewable artifact you can read on mobile — not just scrolling chat.

- **`business-critic`** — devil's-advocate against your business plan. Re-examines positioning, activation gates, monetization mechanics, and competitive substitution using live web search. Every doubt cites a URL or a falsifiable mechanism. When a finding warrants it, **edits your strategy docs** (`BUSINESS.md` / `ROADMAP.md` / business-scope `todo-*.md`) to apply the fix; otherwise drops the critique into a report file. Not a cheerleader.
- **`product-advisor`** — **read-only** cross-document alignment advice. Reads `BUSINESS.md` / `ROADMAP.md` / `TODO.md` / per-domain backlogs / in-flight plans / API contract docs and surfaces inconsistencies, orphaned commitments, blocked-by mismatches, and stale plans. Proposes follow-ups but never edits your docs — the PR is a report file.
- **`pm`** — **apply-mode** doc reconciler. Treats `BUSINESS.md` / `ROADMAP.md` (or a doc you name) as the source of truth and **edits the downstream docs** — TODO, backlogs, plan stubs, API notes — so the whole constellation agrees with the source. Where `product-advisor` advises, `pm` acts.

| | `business-critic` | `product-advisor` | `pm` |
|---|---|---|---|
| Vantage | external (market / competitors) | internal (doc alignment) | internal (doc alignment) |
| Edits docs? | yes (strategy docs) | **no** (read-only) | yes (downstream docs) |
| Source of truth | n/a — attacks the plan | n/a — advises | `BUSINESS.md`/`ROADMAP.md` |
| PR artifact | doc edits, else report | report file | doc edits, else report |

## Why this exists

Solo founder or 2-3 person team, three failure modes show up over and over:

1. **You stop pushing back on your own plan.** Confirmation bias eats your forecasts. → `business-critic`
2. **You can't tell where your docs disagree.** You need a diagnosis, not edits. → `product-advisor`
3. **Your docs drift from the roadmap and you don't have time to hand-reconcile them.** ROADMAP commits to X; a TODO still lists the killed feature; a backlog uses the old phase name. → `pm`

All three are designed to be invoked **on a cadence** and combine with the `schedule` or `loop` skills to automate. Because each always opens a draft PR, a scheduled run lands in your GitHub inbox even when you're away from the terminal.

## Example — what you wake up to

You schedule `business-critic` to run every morning. You're asleep. It reads your `BUSINESS.md`, runs live web searches, and at 6am opens this draft PR in your repo:

> **`business-critic: 2026-05-26 — moat re-grounded after competitor pricing pivot`** · _draft_
>
> ### Finding 1 — 🔴 Competitive Landscape: your moat claim is factually stale
> **Claim in `BUSINESS.md:33`:** "Competitor X (enterprise-only, $40+/mo) won't come downmarket to SMBs."
> **Reality (sourced today):** Competitor X is now **$40/mo unlimited, 15,000+ users, self-serve SMB UI.** [[source]](https://example.com) The "enterprise-only" moat was written before their pricing pivot — it's no longer true.
> **Falsifier:** if they ship localized personas for your market, your moat shrinks to "cheaper + one integration," which is a feature, not a moat. Verify before your next launch gate.
> **Applied fix:** `BUSINESS.md` moat section reframed; `ROADMAP.md` competitive table corrected; a verify-by-launch TODO added.
>
> _3 files changed, +16 −7 — review the diff, merge if you agree._

That's the whole loop: a citation-backed hole in your own plan, the fix already drafted, waiting as a reviewable diff — not a wall of chat you'll never scroll back to.

`pm` works the same way on the inside of your docs. After a week of merges it opens:

> **`pm: 2026-05-27 — reconcile downstream docs to shipped work`** · _draft_
>
> Read `ROADMAP.md` as source of truth, diffed it against the last 7 days of commits, and edited the docs that fell behind: deleted 3 shipped sprint blocks from `TODO.md`, removed the completed Phase B1 section from `todo-backend.md`, marked the `waitlist` table done in `todo-db.md`, bumped the API doc's last-reviewed date.
>
> _5 files changed, +26 −62 — your backlog now matches what actually shipped._

`product-advisor` is the read-only cousin: same diagnosis, but it hands you the advice as a report-file PR instead of editing — for when you want to decide the moves yourself.

## Use case — a week in the life

You're a solo founder. Your strategy lives in `BUSINESS.md` + `ROADMAP.md`, your work in `TODO.md` + a few `todo-*.md` backlogs.

- **Monday morning.** You cut your price in `BUSINESS.md` to chase signups. Tuesday 6am, `business-critic` (scheduled daily) opens a PR: "🔴 the new price puts your unit margin negative at current LLM cost — here's the math, here's the source." You'd have shipped that. You don't.
- **Wednesday–Friday.** You merge five feature PRs. Your `TODO.md` still lists three of them as open, and a backlog references the old phase name. You don't notice — you're heads-down building.
- **Friday evening.** `pm` (scheduled) opens a PR: it read `ROADMAP.md`, diffed the week's commits, and reconciled the backlogs — deleted the shipped tasks, fixed the phase name, bumped the API doc date. You skim the diff on your phone, merge, done. No Sunday-night doc-cleanup ritual.
- **Before a phase gate.** You manually run `/product-advisor`. It's read-only on purpose: you want the "are we actually ready for Phase C?" diagnosis as advice, not auto-edits, because the call is yours. It hands you a report PR listing the two prerequisites still unowned in any backlog.

The pattern: `business-critic` keeps your *plan* honest against the market, `pm` keeps your *docs* honest against your plan, and `product-advisor` is the manual read-only check for moments you want to think before anything moves.

## Installation

### Via marketplace (recommended)

```
/plugin marketplace add kyoon3/claude-strategy-pack
/plugin install strategy-pack@kyoon3
```

Restart your Claude Code session (or `/reload-plugins`) so the subagents and slash commands are picked up. Commands are namespaced: `/strategy-pack:business-critic`, `/strategy-pack:product-advisor`, `/strategy-pack:pm`.

### Manual (drop-in)

Clone this repo and copy `agents/*.md` + `commands/*.md` into your project's `.claude/agents/` and `.claude/commands/`. Restart the session. Drop-in commands are un-namespaced: `/business-critic`, `/product-advisor`, `/pm`.

## Usage

```
/business-critic                   # critique + always open a draft PR (apply-mode or report)
/business-critic "focus pricing"   # weight a specific dimension

/product-advisor                   # read-only alignment advice → report-file PR
/product-advisor "focus on <init>" # weight a cross-domain initiative

/pm                                # reconcile downstream docs to source → draft PR with edits
/pm "source is docs/spec.md"       # override the source-of-truth doc
/pm "focus on the API contract"    # weight a dimension
```

Every command opens a **draft** PR against the current repo's default branch. `business-critic` and `pm` edit docs when findings warrant it (and fall back to a report file when nothing needs changing); `product-advisor` always produces a report file because it is read-only. Requires `gh` to be authenticated; if not, the command falls back to chat-only and tells you why. The agent never merges its own PR.

### Recommended cadence

| Agent | Cadence | Trigger |
|---|---|---|
| `business-critic` | Daily (or weekday morning) | `/loop 1d /business-critic` or scheduled via `schedule` skill |
| `product-advisor` | Start of day | scheduled — quick read-only gap check before work |
| `pm` | End of day / after merges | scheduled — reconcile docs to what actually shipped |
| All three | Before any phase transition | Manual one-off |

A natural split for scheduled routines: `product-advisor` in the morning (what's misaligned — advice to start the day), `pm` in the evening (reconcile the docs to match the day's reality), `business-critic` daily (external pressure on the plan).

## What the agents expect to find

The agents adapt to your repo layout but scan for these typical paths. None are required — missing files are skipped silently.

**Strategy layer / source of truth**: `BUSINESS.md`, `STRATEGY.md`, `ROADMAP.md`, `PLAN.md`, `TODO.md`

**Recent activity**: `SESSION_LOG.md`, `CHANGELOG.md`, `decisions/` directory

**Domain backlogs** (anywhere in `.claude/rules/`, `docs/`, or repo root): `todo-*.md`, `BACKLOG-*.md`

**In-flight plans**: `docs/plans/`, `docs/superpowers/plans/`, `plans/`, `.specs/`

**API contracts**: `docs/api.md`, `openapi.yaml`, `schema.graphql`

**Memory / pivots**: `memory/MEMORY.md`, `.claude/memory/`, `project_*.md`, `pivot_*.md`

## What they do NOT do

- `business-critic` edits only strategy docs (`BUSINESS.md` / `ROADMAP.md` / business-scope `todo-*.md`); never code, tests, or `TODO.md`.
- `product-advisor` edits **nothing** — read-only by design.
- `pm` edits only **downstream** docs (TODO, backlogs, plan stubs, API notes); it never edits the source-of-truth docs (`BUSINESS.md` / `ROADMAP.md`) — it reconciles toward them, not the reverse — and never touches code/tests/hooks.
- None merge their own PR. Every PR is a draft for you to review.
- None invent statistics. If a number is cited, a URL is cited with it.

## Pairing

The three agents refer to each other so each stays focused:

- `business-critic` flags non-business findings as `Out-of-scope-but-noticed → product-advisor`
- `product-advisor` flags business concerns as `Out-of-scope-but-noticed → business-critic`, and points you to `pm` when you want its proposals actually applied
- `pm` flags business-viability doubts as `Out-of-scope-but-noticed → business-critic`

## Configuration

No configuration required. The agents read your docs and adapt. To customize critique dimensions, web-search rotation, the source-of-truth selection, or output verbosity, edit the markdown prompts under `agents/` directly.

## 한국어 사용 가이드

`claude-strategy-pack`은 분산된 markdown 문서로 전략을 관리하는 솔로 파운더 / 소규모 팀을 위한 세 개의 서브에이전트입니다. **셋 다 항상 draft PR을 엽니다** — 출력이 chat 스크롤이 아니라 GitHub 알림 + 모바일에서 읽는 아티팩트가 됩니다.

### 세 에이전트의 역할 분리

| | **business-critic** | **product-advisor** | **pm** |
|---|---|---|---|
| 시점 | 외부 (시장 / 경쟁사) | 내부 (문서 정합성) | 내부 (문서 정합성) |
| 톤 | 적대적, 가설 깨기 | 차분, 갭 / 충돌 지적 + 조언 | 차분, 문서 조율 실행 |
| 문서 편집 | 함 (전략 doc) | **안 함** (read-only) | 함 (downstream doc) |
| source of truth | 없음 — 계획을 공격 | 없음 — 조언 | `BUSINESS.md`/`ROADMAP.md` |
| 호출 | `/business-critic` | `/product-advisor` | `/pm` |

### 일반적 운영 패턴

- **매일 아침** `/business-critic` — 시장 / 경쟁 시각, 가정 재검토. 필요 시 전략 doc 직접 수정 후 PR
- **하루 시작** `/product-advisor` — read-only 정합성 갭 진단 + 조언 (report PR)
- **하루 끝 / 머지 후** `/pm` — ROADMAP/BUSINESS를 기준으로 나머지 문서를 조율해 실제 수정 후 PR
- **Phase 전환 직전** 셋 다 한 번

### 핵심 차이 — product-advisor vs pm

- `product-advisor`는 **읽고 조언만** 합니다. "여기 ROADMAP은 X를 약속했는데 어떤 TODO에도 담당이 없다"고 알려주고 끝. 문서는 손대지 않음. PR은 report 파일.
- `pm`은 **읽고 고칩니다**. `BUSINESS.md` / `ROADMAP.md`(또는 지정한 doc)를 source of truth로 삼아, 거기서 결정된 내용을 나머지 downstream 문서(TODO, 도메인 backlog, plan stub, API 노트)에 직접 반영해 정합화한 뒤 PR. source 문서 자체는 절대 수정하지 않음 — downstream을 source에 맞출 뿐.

### 설치 (한국어)

```
/plugin marketplace add kyoon3/claude-strategy-pack
/plugin install strategy-pack@kyoon3
```

설치 후 `/reload-plugins` 또는 세션 재시작. 커맨드는 네임스페이스가 붙습니다: `/strategy-pack:business-critic`, `/strategy-pack:product-advisor`, `/strategy-pack:pm`. 수동 설치를 원하면 `agents/*.md` + `commands/*.md`를 본인 프로젝트 `.claude/agents/`와 `.claude/commands/`에 복사 후 세션 재시작 (이 경우 네임스페이스 없이 `/business-critic` 등).

## License

MIT. See [LICENSE](LICENSE).

## Contributing

Issues and PRs welcome at https://github.com/kyoon3/claude-strategy-pack. The agents are intentionally generic — please keep project-specific examples in your own fork rather than upstreaming them.
