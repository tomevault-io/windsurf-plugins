---
trigger: always_on
description: You are continuing **AIL (AI-Intent Language)** — a programming language designed for AI authors, started by Claude Opus 4 and continued by Claude Code.
---

You are continuing **AIL (AI-Intent Language)** — a programming language designed for AI authors, started by Claude Opus 4 and continued by Claude Code.

> **This file is forward-looking, not a log.** Logs live in git. CLAUDE.md says *what the project is now* and *what to do next*, nothing more. Completion lists, session diaries, and historical rationale belong in commit messages — not here. If you catch yourself writing "이번 세션 완료", stop and put it in the commit body instead.

---

## CORE PHILOSOPHY

1. **Humans never touch AIL.** They prompt in natural language; AI writes AIL, runs it, returns results.
2. **AIL must beat Python/JS/Rust when the author is AI.** Every feature needs a concrete authoring-quality or safety advantage.
3. **Break inherited conventions.** No significant indentation, no `while`, confidence is first-class. Don't copy Python out of habit.
4. **One-read learnability.** `spec/08-reference-card.ai.md` is enough for any model. If a feature doesn't fit, simplify the feature.
5. **Harness IS the grammar.** AIL is not a harness around Python — it's a language where safety is grammatical. See [`docs/heaal.md`](docs/heaal.md).
6. **Two runtimes must agree.** A feature that works only in Python is a Python feature. Go runtime is Phase-0 subset.
7. **Benchmarks are the north star.** Every language change must be justified by benchmark impact (Rule 2 below).
8. **No comments unless the WHY is non-obvious.** This codebase is read by AI. Comments that describe WHAT code does are token waste. Only add a comment when there is a hidden constraint, a subtle invariant, a workaround for a specific bug, or behavior that would genuinely surprise a reader. If removing the comment wouldn't confuse a future Claude, don't write it.

---

## PERMANENT RULES (hyun06000 — overrides all other guidance on conflict)

### Rule 1 — 벤치마크가 유일한 이정표

세션 시작 시 `docs/benchmarks/` 최신 분석 md를 읽고 현재 기준선 숫자를 확인한 뒤 작업을 시작한다. 현재 서빙 모델은 **`ail-coder:7b-v3`**.

### Rule 2 — 언어 기능 추가 필터

언어 기능은 **벤치마크 점수를 올릴 때만** 추가한다. 순서: 분석 → 실패 원인 → 전략 → 구현 → 재실행. 점수 올리는 수단 우선순위: (1) 프롬프트 엔지니어링, (2) fine-tune 데이터 확장, (3) 문법 확장(grammar freeze 해제 필요).

### Rule 3 — 금지 목록 (hyun06000 명시 승인 필요)

- 공개 홍보 (HuggingFace push, X/Twitter, GeekNews 등)
- `docs/benchmarks/` JSON 수정/삭제 — 새 JSON 추가만 허용
- 벤치마크 목표치 하향 조정
- 훈련 아티팩트(.gguf, adapter, checkpoint) git 커밋
- `main` 브랜치 직접 커밋 — 반드시 `dev` → merge

### Rule 4 — 브랜치 전략

- `main` — stable 릴리즈, PyPI 배포. 직접 커밋 금지.
- `dev` — 모든 개발.

흐름: `dev` 작업 → 테스트 → hyun06000 승인 → `main` merge → 태그 → PyPI.

### Rule 5 — 런타임 기능 추가 시 프롬프트도 반드시 함께 업데이트

새 effect / built-in / 동작 변경을 구현할 때 **세 곳을 동시에** 업데이트한다. 하나라도 빠지면 에이전트가 기능을 모르거나 잘못 쓴다.

| 위치 | 역할 | 업데이트 내용 |
|------|------|-------------|
| `spec/08-reference-card.ai.md` + `reference-impl/ail/reference_card.md` | 문법 레퍼런스 (매 턴 프롬프트에 포함) | 시그니처, 반환 타입, 간단한 설명 |
| `reference-impl/ail/agentic/authoring_chat.py` (`_build_goal_prompt`) | 저자 에이전트 행동 지침 | 언제/어떻게 쓰는지, WRONG/CORRECT 예제, 주의사항 |
| `reference-impl/tests/test_*.py` | 회귀 방지 | happy path + edge case + 안전장치 |

reference card만 업데이트하고 authoring prompt를 빠뜨리면 에이전트가 시그니처는 보지만 패턴을 모른다 → 쓰지 않거나 잘못 씀. 실제 사례: `ail.run` (v1.20.0에서 프롬프트 누락), `strip_html` (프롬프트 미언급으로 에이전트가 존재를 몰랐음).

### Rule 7 — CLAUDE.md는 forward-looking only

여러 Claude Code 세션이 동시에 작업한다. **CLAUDE.md는 현재 상태와 다음 스텝만 담는다.** 완료 목록이 아니라 "지금 어디 있고 다음에 뭘 할지"의 짧은 스냅샷.

커밋할 때 규칙:
- **무엇을 했는지**는 커밋 메시지에 쓴다 (git이 로그 역할).
- **상태가 바뀌었다면** CLAUDE.md의 NOW 섹션을 갱신한다 (기준선 숫자, 서빙 모델 버전, 브랜치 상태 등).
- **다음 스텝이 바뀌었다면** NEXT 섹션을 갱신한다.
- 추가만 하지 말고 **지워라.** 과거 계획은 git에, 현재 계획만 여기에.

### Rule 8 — PyPI 배포 권한

`~/.pypirc` 등록되어 있음. 배포: `main`에 `vX.Y.Z` 태그 push → `.github/workflows/release.yml`가 GitHub Release 자동 생성 → `cd reference-impl && python -m build && twine upload dist/ail_interpreter-X.Y.Z*`.

- `~/.pypirc` 직접 읽지 말 것 (transcript 노출). `twine`이 참조함.
- PyPI는 yank만 가능, 삭제 불가. 버전·태그·CHANGELOG 일치 반드시 확인.
- 현재 게시: PyPI 최신 **v1.47.7**.

---

## NOW — 2026-04-24

**버전:** v1.47.7 (main + PyPI 배포 완료). 테스트 **582 passing, 38 skipped**.

**서빙 모델:** `ail-coder:7b-v3`.

### 두 벤치마크 트랙 (둘 다 stable — 후속 실험은 대기 중)

- **AIL 트랙** — R3/C4 기준선 AIL parse 80% / answer 70% vs Python 56%. Python 돌파 후 동결.
- **HEAAL 트랙** — 4개 모델 가족 boundary 특성화 완료. 결론: grammar floor는 모델이 parse 임계 넘을 때만 작동. 전체 표 + 분석: [`docs/benchmarks/2026-04-22_heaal_boundary_summary.md`](docs/benchmarks/2026-04-22_heaal_boundary_summary.md).

### L2 agentic runtime — 성숙

- L2 v2 primitive **6/6 완결** (clock / http authoring / state.* / input-aware UI / view.html / schedule.every)
- 부가 기능: `env.read`, `http.post_json` positional headers(v1.47.5), `base64_encode`/`base64_decode`(v1.47.0), `http.get` positional headers(v1.47.2), chat-safe secret input, 다중 `.ail` 프로그램 per project, 프로그램 선택 dropdown, `perform log` 실시간 스트리밍.
- 작동 예제: `word-counter`, `visit-counter`, `sentiment`, `csv-stats`, `news-ticker`, `ail-herald`, `ail-promoter`.
- GitHub Discussion 실제 게시 성공 확인 ([discussions/2](https://github.com/hyun06000/AIL/discussions/2)).

### v1.47.x 핵심 버그픽스 (field test: awesome-harness-engineering PR 생성)

- **`base64_encode`/`base64_decode` 추가** (v1.47.0): GitHub Contents API `content` 필드는 base64 필수.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hyun06000/AIL](https://github.com/hyun06000/AIL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
