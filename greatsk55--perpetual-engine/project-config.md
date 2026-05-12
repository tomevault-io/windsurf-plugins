---
trigger: always_on
description: AI 에이전트 스타트업 프레임워크 - 토큰만 투자하면 AI가 사업을 만든다.
---

# Perpetual Engine

AI 에이전트 스타트업 프레임워크 - 토큰만 투자하면 AI가 사업을 만든다.

## 프로젝트 개요
- npm으로 설치 가능한 CLI 프레임워크
- CEO, CTO, PO, Designer, QA, Marketer 에이전트 팀이 자율적으로 스타트업 운영
- tmux 기반 멀티 에이전트 병렬 실행
- 로컬 대시보드(http://localhost:3000)로 실시간 모니터링 (칸반보드, 에이전트 상태)
- 모든 동작은 CLI와 GUI(대시보드) 양쪽에서 가능

## 기술 스택
- CLI: Node.js + TypeScript + Commander.js
- Dashboard: Express + WebSocket + 인라인 React (Tailwind CDN)
- State: File-based JSON (kanban.json, sprints.json)
- Agent Runtime: tmux + Codex CLI
- 문서 관리: Markdown + Git

## 프로젝트 구조
```
src/
├── cli/           # CLI 명령어 (Commander.js)
├── core/          # 핵심 비즈니스 로직 (CLI/대시보드 공유)
│   ├── project/   # 프로젝트 초기화, 설정 관리
│   ├── agent/     # 에이전트 정의, 레지스트리, 프롬프트 빌더, 스킬 매핑
│   ├── session/   # tmux 세션 관리
│   ├── state/     # kanban/sprint CRUD (file-store 기반)
│   ├── workflow/  # 워크플로우 엔진, 오케스트레이터
│   ├── metrics/   # 메트릭스 기반 기획 평가 시스템
│   ├── context/   # 문서 기반 컨텍스트 관리
│   └── messaging/ # 메시지 큐, 회의 시스템
├── dashboard/     # Express API + WebSocket + HTML 클라이언트
└── utils/         # 로거, YAML, 경로, 에러
```

## 주요 명령어
```bash
perpetual-engine init <name>           # 프로젝트 생성
perpetual-engine setup                 # 대화형 설정 (작업 언어/회사/프로덕트 등)
perpetual-engine start                 # 에이전트 + 대시보드 시작 (기본: 빈 프로젝트에만 CEO 자동 기동)
perpetual-engine start --no-ceo        # CEO 자동 기동 건너뛰기 (워처만 가동)
perpetual-engine start --force-ceo     # 기존 스프린트가 있어도 CEO 재계획 강제
perpetual-engine stop                  # 모든 에이전트 종료
perpetual-engine team                  # 팀 목록
perpetual-engine status                # 상태 요약
perpetual-engine board                 # 터미널 칸반보드
perpetual-engine message <msg>                  # 팀에게 메시지 (기본 urgent — 진행 중 워크플로우 인터럽트)
perpetual-engine message <msg> --to <role>      # 특정 역할에게 보내기 (기본 to: all → ceo)
perpetual-engine message <msg> --normal         # 우선순위 없이 일반 큐로 보내기
perpetual-engine task run <id>         # 태스크 강제 실행 (의존성/상태 무시)
perpetual-engine task suspend <id>     # 태스크 일시 중단
perpetual-engine task resume <id>      # 중단된 태스크 재개
perpetual-engine task list [-s status] # 태스크 목록 (상태 필터 가능)
```

## 아키텍처 원칙
- **SSOT**: kanban.json이 태스크 상태의 유일한 소스, metrics.json이 메트릭스의 유일한 소스, `docs/development/feature-<slug>/components.json` 이 컴포넌트 분해의 유일한 소스
- **CLI/Core 분리**: CLI와 대시보드 모두 같은 core 모듈 사용
- **파일 기반 통신**: 에이전트 간 통신은 파일 시스템(messages/, docs/) 기반
- **세션 독립성**: 각 워크플로우 페이즈는 새 Codex 세션에서 실행
- **메트릭스 기반 의사결정**: 모든 기획에 측정 지표/기간을 설계하고, 달성도로 다음 행동(확대/유지/반복개선/방향전환/폐기) 결정
- **작업 언어 일원화**: setup 시 선택한 언어(`config.localization`)를 PromptBuilder가 모든 에이전트 시스템 프롬프트 최상단에 주입 — 대화·문서·칸반·커밋·자문 요청 모두 동일 언어. 코드 식별자/외부 API/URL은 원문 유지.
- **한 역할당 동시 세션 1개 (프로젝트 단위)**: tmux 세션명이 `<prefix>-<role>` 형식이라 같은 prefix 안에서 같은 역할로 세션을 동시에 2개 이상 띄울 수 없다. prefix 는 프로젝트별로 격리되므로 다른 프로젝트의 같은 역할은 충돌하지 않는다(아래 멀티 인스턴스 항목). 새 태스크 디스패치 경로를 추가할 때는 `Orchestrator.processingRoles` 락을 반드시 거치고, 세션명을 `role` 이 아닌 다른 키로 쓸 거면 SessionManager 의 activeSessions 키 전체를 함께 바꿔야 한다.
- **같은 머신 멀티 인스턴스 격리**: `config.runtime.session_prefix` / `dashboard_port` 가 프로젝트마다 자동 도출(`projectRoot` sha256)되어 박힌다 — 두 PE 프로젝트를 동시에 `start` 해도 tmux 세션명/대시보드 포트가 안 겹친다. CLI `start --port <n>` 로 일회성 포트 override. 새 글로벌 리소스(소켓/PID/락 파일 등) 도입 시 `runtime` 섹션에 격리 키를 추가하고 `deriveRuntimeDefaults` 가 채우게 해야 한다 — 하드코딩 금지. 새 CLI 가 SessionManager 를 만들 땐 `new SessionManager()` 가 아니라 `createSessionManager(projectRoot)` helper 를 써야 prefix 가 자동 적용된다.
- **컴포넌트 단위 개발 + 자체 테스트 실행 루프**: development 페이즈는 `development-plan` → `development-component`(컴포넌트마다 반복) → `development-integrate` 3단으로 쪼갠다. 한 컴포넌트는 한 세션에서 5–15분 안에 끝난다. **unit + ui 두 종 테스트는 필수** (snapshot/integration/e2e 는 선택). 한 세션 안에서 구현 + 테스트 작성 + Bash 로 테스트 명령 실행을 모두 처리하고 모든 테스트가 통과해야 종료. `tech_stack.test_runners.<kind>` 가 `{ tool, command }` 객체로 명시되면 워크플로우 엔진이 페이즈 종료 후 그 `command` 를 다시 한 번 Bash 로 실행해 종료코드 0 인지 검증하고, 실패 시 같은 페이즈를 자동 재시도한다. 실행 결과는 `docs/development/feature-<slug>/components/<comp>.test-output.md` 에 기록되어 다음 재시도 세션의 `inputDocPaths` 로 전달된다.
- **세션 종료는 idle 감지 (wall-clock 타임아웃 없음)**: 작업 중엔 무제한 기다린다. `<role>.log` mtime 이 `idleTimeoutMs`(기본 10분) 이상 정체되면 hang 으로 보고 `stopAgent` 후 페이즈 fail → onFailure 가 자기 자신이라 자동 재시도. 30초마다 진행 로그(`[role] 진행 중 (X분 경과, 마지막 출력 Y초 전)`) 출력. **새 페이즈를 추가할 때 wall-clock `Phase.timeoutMs` 를 다시 도입하지 말 것** — 페이즈별 한도가 필요하면 idle 임계값으로 표현한다. 외부 명령은 `--silent` 금지(진행 로그가 흘러야 idle 오인 안 됨).
- **단발 세션도 락 + idle-watch 로 보호 (부트스트랩/메시지 디스패치)**: `Orchestrator.runRoleSession({ role, label, start })` 헬퍼로 모든 비-task 세션을 감싼다. `sessionAborters: Map<role, AbortController>` 에 등록 → urgent 메시지가 같은 role 에 들어오면 abort + stopAgent 후 새 세션. **normal 메시지는 같은 role 의 작업이 살아있으면 markAsRead 도 하지 말고 보류** — 다음 watcher tick 에 자동 재시도. 이전엔 모든 메시지가 무조건 stopAgent 해서 분석 directive 가 통째로 잘렸음. 30초마다 진행 로그가 stdout 에 흐른다 (`[role/label] 진행 중 ...`). 새 세션 경로 추가 시 반드시 `runRoleSession` 으로 감쌀 것 — 안 그러면 watcher 누락 + urgent 인터럽트 안 됨 + stop drain 누락. idle-watch 로직은 `src/core/session/session-watcher.ts` SSOT (`waitForSessionCompletion`).
- **세션 로그는 `tee -a` (append) 로 누적**: `tee` truncate 모드면 새 세션 시작마다 직전 출력이 통째로 사라져 디버깅 흔적이 없어진다. session-manager 의 3개 startAgent 경로(`startAgent` / `startEphemeralAgent` / `startMeetingSession`) 모두 `tee -a` 사용. 새 세션 종류 추가 시 같은 패턴 유지 — 이 미러가 끊기면 idle-watch 가 출력 정체로 오인해 hang 처리한다.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [greatsk55/perpetual-engine](https://github.com/greatsk55/perpetual-engine) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
