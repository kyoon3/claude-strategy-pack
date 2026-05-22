# claude-strategy-pack

Two read-only Claude Code subagents for solo founders and small teams whose strategy lives in distributed markdown docs:

- **`business-critic`** — daily devil's-advocate against your business plan. Re-examines positioning, activation gates, monetization mechanics, and competitive substitution using live web search. Every doubt cites a URL or a falsifiable mechanism. Not a cheerleader.
- **`project-pm`** — weekly cross-document alignment sync. Reads `BUSINESS.md` / `ROADMAP.md` / `TODO.md` / per-domain backlogs / in-flight plan files / API contract docs and surfaces inconsistencies, orphaned commitments, blocked-by mismatches, and stale plans. Proposes follow-up plan stubs but never edits your docs.

Both are **read-only**. They produce reports you act on; they never silently rewrite your strategy.

## Why this exists

If you are a solo founder or 2-3 person team, two failure modes show up over and over:

1. **You stop pushing back on your own plan.** Confirmation bias eats your forecasts. → `business-critic`
2. **Your strategy docs, sprint backlogs, and in-flight plans drift apart.** ROADMAP commits to X, no one owns X in any TODO, frontend backlog has half of X under a different name. → `project-pm`

Both agents are designed to be invoked **on a cadence** — daily for the critic, weekly for the PM — not just on demand. Combine with the `schedule` or `loop` skills to automate.

## Installation

### Via plugin install (recommended)

```
/plugin install github.com/kyoon3/claude-strategy-pack
```

Restart your Claude Code session after install so the subagents and slash commands are picked up.

### Manual (drop-in)

Clone this repo and copy `agents/*.md` + `commands/*.md` into your project's `.claude/agents/` and `.claude/commands/`. Restart the session.

## Usage

```
/business-critic                 # daily critique pass (chat output only)
/business-critic "focus pricing" # weight a specific dimension
/business-critic pr              # also open a draft GitHub PR with the findings
/business-critic pr focus pricing  # both options combine

/pm                              # weekly alignment sync (chat output only)
/pm "focus on <initiative>"      # weight a cross-domain initiative
/pm pr                           # also open a draft GitHub PR with the report
```

Both commands accept an optional focus arg. Pass the literal token `pr` to additionally
save the findings to `reports/<agent>/<timestamp>.md` and open a **draft** PR against the
current repo's default branch — useful for scheduled routines so the output becomes a
GitHub notification + persistent artifact instead of scrolling chat. Requires `gh` to be
authenticated; if not, the command falls back to chat-only and tells you why.

### Recommended cadence

| Agent | Cadence | Trigger |
|---|---|---|
| `business-critic` | Daily (or weekday morning) | `/loop 1d /business-critic` or scheduled via `schedule` skill |
| `project-pm` | Weekly (Monday morning) | Manual or `/loop 7d /pm` |
| Both | Before any phase transition | Manual one-off |

## What the agents expect to find

The agents adapt to your repo layout but scan for these typical paths. None are required — missing files are skipped silently.

**Strategy layer**: `BUSINESS.md`, `STRATEGY.md`, `ROADMAP.md`, `PLAN.md`, `TODO.md`

**Recent activity**: `SESSION_LOG.md`, `CHANGELOG.md`, `decisions/` directory

**Domain backlogs** (anywhere in `.claude/rules/`, `docs/`, or repo root): `todo-*.md`, `BACKLOG-*.md`

**In-flight plans**: `docs/plans/`, `docs/superpowers/plans/`, `plans/`, `.specs/`

**API contracts**: `docs/api.md`, `openapi.yaml`, `schema.graphql`

**Memory / pivots**: `memory/MEMORY.md`, `.claude/memory/`, `project_*.md`, `pivot_*.md`

If you keep strategy somewhere unusual, the agents will ask once and remember (or you can amend their system prompts under `agents/`).

## What they do NOT do

- They never edit any doc, plan, or backlog file. They produce reports.
- `business-critic` does not propose features or implementation. It evaluates the plan.
- `project-pm` does not critique the business or the code. It only reconciles documents.
- Neither agent invents statistics. If a number is cited, a URL is cited with it.

## Pairing

Both agents are designed to refer to each other:

- `business-critic` flags non-business findings as `Out-of-scope-but-noticed → project-pm`
- `project-pm` flags business concerns as `Out-of-scope-but-noticed → business-critic`

This keeps each agent focused while letting the other catch what falls out of scope.

## Configuration

No configuration is required. Both agents read your docs and adapt.

If you want to customize critique dimensions, web-search rotation, or output verbosity, edit `agents/business-critic.md` or `agents/project-pm.md` directly — they are pure markdown prompts.

## 한국어 사용 가이드

`claude-strategy-pack`은 분산된 markdown 문서로 전략을 관리하는 솔로 파운더 / 소규모 팀을 위한 두 read-only 서브에이전트입니다.

### 두 에이전트의 역할 분리

| | **business-critic** | **project-pm** |
|---|---|---|
| 시점 | 외부 (시장 / 경쟁사 / 회의론자) | 내부 (문서 간 정합성) |
| 톤 | 적대적, 가설 깨기 | 차분, 갭 / 충돌 지적 |
| 인풋 | 전략 doc + 라이브 웹서치 | 모든 도메인 backlog + plan + API doc + git log |
| 아웃풋 | 비판 + falsifier | 정합성 표 + 제안 task + plan stub |
| 편집 | read-only | read-only |
| 호출 | `/business-critic` | `/pm` |

### 일반적 운영 패턴

- **매일 아침** `/business-critic` — 시장 / 경쟁 시각, 가정 재검토 (`/loop 1d /business-critic`)
- **매주 월요일** `/pm` — 한 주 시작 시 갭 검출 + 이번 주 task 정합화
- **Phase 전환 직전** 둘 다 한 번 — 외부 견적 + 내부 준비도 동시 검토

### 한국 솔로 파운더 사용 사례

이 플러그인은 다음과 같은 상황을 가정하고 설계됐습니다:

- `BUSINESS.md` / `ROADMAP.md` / `TODO.md` + 도메인별 `todo-*.md` (백엔드 / 프론트 / DB / 인프라 / 비즈니스) 가 분산돼 있음
- 한 주에 여러 PR이 머지되면서 문서가 따로 놂 — ROADMAP은 X를 약속했는데 어떤 TODO에도 X 담당이 없음
- 솔로라서 외부 시각을 강제로 받을 사람이 없어 confirmation bias가 쌓임

`business-critic`은 한국어 전략 문서를 그대로 읽고 한국 경쟁사 / 한국어 뉴스도 웹서치합니다 — 본인 `BUSINESS.md`에 적힌 경쟁사 이름을 기준으로 검색하므로 한국 ad-tech / SNS / 콘텐츠 도메인 모두 커버됩니다. 출력 메소돌로지는 영문이지만 인용은 원문 유지.

### 출력 예시

`/business-critic`은 아래 형식으로 답변합니다:

```
# Business Critique — 2026-05-17

## Headline
이번 주 가장 약한 가정 한 줄

## Today's angle
오늘 깊게 본 3-4개 차원과 이유

## Findings
### 1. <구체적 우려> [severity: 🔴]
**Claim in plan** (`BUSINESS.md:42`): "..."
**Why I doubt it**: 메커니즘 1-3줄
**Evidence**: URL 또는 비교 가능한 케이스
**Falsifier**: 어떤 실험 / 숫자가 내 의심을 깰지

## Market signal of the day
오늘 웹서치에서 발견한 한 가지 + 링크

## What I'm NOT worried about today
검토했지만 안전하다고 판단한 것들

## Suggested next move
이번 주 할 일 한 가지
```

`/pm`은 도메인 정합성 표 + finding 블록 + 필요 시 plan stub 초안을 제공.

### 설치 (한국어)

```
/plugin install github.com/kyoon3/claude-strategy-pack
```

설치 후 Claude Code 세션 재시작. 그러면 `/business-critic`, `/pm` 슬래시 커맨드가 인식됨.

수동 설치를 원하면 `agents/*.md` + `commands/*.md`를 본인 프로젝트 `.claude/agents/`와 `.claude/commands/`에 복사 후 세션 재시작.

## License

MIT. See [LICENSE](LICENSE).

## Contributing

Issues and PRs welcome at https://github.com/kyoon3/claude-strategy-pack. The agents are intentionally generic — please keep project-specific examples in your own fork rather than upstreaming them.
