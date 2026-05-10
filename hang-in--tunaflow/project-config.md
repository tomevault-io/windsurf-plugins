---
trigger: always_on
description: > This file is the system-level handoff document for AI agents (Claude Code, Codex, Gemini) operating on tunaFlow. Human readers: see [README.md](./README.md) for the product overview and [docs/](./docs/) for architecture and design plans.
---

# tunaFlow — Claude Code Handoff Document

> This file is the system-level handoff document for AI agents (Claude Code, Codex, Gemini) operating on tunaFlow. Human readers: see [README.md](./README.md) for the product overview and [docs/](./docs/) for architecture and design plans.

> 최종 갱신: 2026-04-13 (세션 34 반영)
> SSOT: `docs/reference/dataModelRevised.md` (도메인 모델), `docs/reference/implementationStatus.md` (구현 현황)
> **세션 이력 전체**: `docs/reference/sessionHistory.md` — 새 세션 첫 요청 시 또는 과거 결정 맥락 필요 시 읽을 것

---

## 1. 프로젝트 개요

tunaFlow는 **다중 에이전트 오케스트레이션 클라이언트(AOC)**이다. Tauri 2 + React + TypeScript + Rust + SQLite 기반.

> **"Of the agent, By the agent, For the agent"**
> 도메인 지식을 기반으로 서비스를 구축하는 **인간지능 주도형 개발 어플리케이션**이다.
> 사용자가 도메인 지식과 방향을 결정하고, 에이전트가 그 결정을 최적의 조건에서 실행한다.
> 에이전트가 편해야 결과가 좋아진다는 철학 — ContextPack, identity, memory, retrieval 등 모든 설계는 "에이전트가 불필요한 토큰 낭비 없이, 정확한 맥락으로, 역할 혼동 없이 작업할 수 있는가"를 기준으로 판단한다.

핵심 기능:
- 프로젝트 단위로 Claude / Codex / Gemini / Ollama / LM Studio 에이전트를 실행 (UI 디스패치 기준 5종. OpenCode 는 `src-tauri/src/agents/opencode.rs` 에 소스만 유지, UI 미연결)
- Roundtable(RT) 토론: 여러 에이전트가 순차(Sequential) 또는 병렬(Deliberative)로 토론
- Branch: 대화 중간에서 분기해 독립 실험 후 adopt(요약 삽입)
- Plan/Artifact/Memo: 작업 계획, 산출물, 메모 관리
- ContextPack: 매 요청마다 normalized prompt를 조립 (5개 엔진 공통)
- rawq: 코드 검색 엔진 (sidecar, daemon 모드)
- Skills: vendor별 스킬 snapshot (`~/.tunaflow/skills/`)

---

## 2. 기술 스택

| 계층 | 기술 |
|---|---|
| Desktop shell | Tauri 2 |
| Frontend | React 18 + TypeScript + Zustand 5 + Tailwind CSS 4 |
| Backend | Rust (tauri commands) |
| DB | SQLite (WAL mode, dual read/write connections) |
| Agent CLI | claude, codex(OpenAI), gemini(Google), ollama / lmstudio (openai-compat) — UI 연결 5종. OpenCode 는 소스만 유지, UI 미노출 |
| Markdown | react-markdown + remark-gfm + react-syntax-highlighter (Prism + oneDark) |
| Icons | Lucide React |
| Code search | rawq (sidecar binary, daemon mode) |

---

## 3. 아키텍처 요약

> **상세 참조**: `docs/reference/architecture-detail.md` — 프로젝트 구조, 레이아웃, RT 흐름, Store 구조, DB 스키마, 이벤트 모델. **해당 영역 작업 시에만 읽을 것.**

- **Project-centric**: 모든 데이터는 Project 소속. soft-hide 삭제.
- **Background execution**: `start_*` 커맨드 → 즉시 반환 → background subprocess → 이벤트 통지. DB = SSOT.
- **ContextPack (5-engine parity)**: `build_normalized_prompt_with_budget()` 단일 함수. Lite/Standard/Full auto mode. 동적 예산 배분. UI 연결 엔진: claude / codex / gemini / ollama / lmstudio.
- **Branch**: 대화 분기. shadow conversation. 드로어 또는 고정 패널.
- **RT**: Branch의 확장 모드. Sequential/Deliberative. 드로어 안에서 동작.
- **rawq**: sidecar daemon. 임베딩 기반 코드 검색. FS watcher 자동 재인덱싱.

---

## 5. 현재 상태 (세션 40 기준)

- **DB**: v46 / **Rust**: 485 tests / **Frontend**: 317 tests
- **현재 브랜치**: `main` (v0.1.0-beta 공개)
- **알려진 이슈**
  - RT 라운드 중 제한적인 중간 인터럽션 (구조적 변경 여지 남음)
  - window-state: dev 모드 Ctrl+C 종료 시 상태 미저장
  - JSONL 빠른 완료 감지 실패 (P1): PTY 응답 UI 미반영 간헐적 발생
  - bge-m3 CPU 스파이크 수정됨 (s35): ONNX 스레드 제한 + 세마포어 + 점진적 인덱싱
  - Ollama / LM Studio base URL override UI 부재 (Issue #175, `customEndpointConfigPlan_2026-04-24` 로 MVP 준비)
- **전체 이력**: `docs/reference/sessionHistory.md`

---

> **§6~9 (RT 흐름, Store 구조, DB 스키마, 이벤트 모델)**: `docs/reference/architecture-detail.md` 참조. 해당 영역 작업 시에만 읽을 것.

---

## 10. 세션 이력

> 전체 이력: `docs/reference/sessionHistory.md` — 새 세션 첫 요청 시 또는 과거 결정 맥락 필요 시 읽을 것

---

## 11. 다음 우선순위

### P0: 완료
- ~~PTY write queue (FIFO 순서 보장)~~ — s24 완료
- ~~ptySpawnLock → per-conversation Map~~ — s24 완료
- ~~PTY 완료 후 결과 미표시 (adoptBranch 충돌)~~ — s25 완료
- ~~adopt 중 스트리밍 메시지 소멸~~ — s25 완료
- ~~main 머지 준비~~ — s23 완료
- ~~리팩토링 v3 Tier 1(2.4+2.5) + Tier 2(2.6+2.8)~~ — s26 완료

### P1: 진행 대상
- ~~**리팩토링 v3 잔여**~~ ✅ — s27~s29 완료 (http_api/pty/executor/threadSlice/streamingUtils 모두 분리)
- ~~**ContextPack DB/assembly 완전 분리**~~ ✅ — `send_common/` 4파일로 이미 분리 완료 (context_loading/prompt_assembly/persistence/mod)
- ~~**브랜치 label git 스타일 slug화**~~ ✅ — s24~s27에서 `slugify_label()` 구현 완료
- **라이트 모드** ✅ — ~~oklch 기반 다크/라이트 토글 (디자인 시스템 Phase 2)~~
- ~~RT 전용 페르소나 설계~~ ✅ — s27 완료 (role_guidance() 4종)
- Project-per-window 아키텍처 (`docs/ideas/projectPerWindowIdea.md`) — VS Code 패턴
- KnowledgeLayer trait — 6번째 소스 추가 시 도입
- Insight Phase H(auto-export) ✅ / J(plan done→findings resolved) ✅ / I(tool-request:insight 핸들러) ✅ — s29 완료
- 온보딩 메타에이전트 (`docs/ideas/onboardingMetaAgentIdea.md`)

### P2: 후순위
- 디자인 시스템 확대 — text-tf-*/prose-* 토큰 점진 교체
- Gemini SDK 직접 통합 (보조 경로, CLI 기본 유지)
- smoke test 복구
- Trace Phase 2: Git 상태 + OTel 중첩 스팬
- Codex app-server 프로토콜 분석

---

## 12. 빌드 / 실행 / 테스트

```bash
# 개발 실행
npm run tauri dev

# 빌드 검증
npx tsc --noEmit              # TypeScript
npx vite build                # Frontend
cd src-tauri && cargo check   # Rust

# 테스트
npx vitest run                # Frontend (317 tests)
cd src-tauri && cargo test --lib  # Rust unit tests (485 tests)

# rawq sidecar 준비
./scripts/build-rawq.sh       # macOS/Linux
./scripts/build-rawq.ps1      # Windows

# Skills snapshot 발행
./scripts/publish-skills.sh
```

---

## 13. 문서 참조

| 문서 | 용도 |
|---|---|
| `docs/reference/sessionHistory.md` | **세션 이력 전체** — 새 세션 시작 시 또는 과거 결정 맥락 필요 시 읽기 |
| `docs/reference/dataModelRevised.md` | 도메인 모델 SSOT |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hang-in/tunaFlow](https://github.com/hang-in/tunaFlow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-28 -->
