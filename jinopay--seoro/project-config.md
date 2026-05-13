---
trigger: always_on
description: - **Tech Stack**: .NET 10.0 + Blazor + Photino.Blazor (desktop) + MudBlazor v9 (UI)
---

# CLAUDE.md

## Critical Context
- **Tech Stack**: .NET 10.0 + Blazor + Photino.Blazor (desktop) + MudBlazor v9 (UI)
- **Solution**: `Seoro.slnx` (new XML solution format)
- **Projects**:
  - `src/Seoro.Desktop` - Desktop app entry point (WinExe, Photino window)
  - `src/Seoro.Shared` - Shared library (models, services, Razor components)
  - `tests/Seoro.Shared.Tests` - xUnit test suite
- **Platform**: Windows x64, macOS ARM64
- **Build**: `dotnet build Seoro.slnx`, release via `dotnet publish` + Velopack
- **Test**: `dotnet test`
- **Auto-update**: Velopack v0.0.1298 (`vpk` tool)
- **CI/CD**: GitHub Actions (`release.yml`) - triggered on `v*` tags
- **Required CLI**: Claude CLI >= 2.1.81
- **Required Runtime**: Node.js 20 (CI/CodeMirror 빌드)
- **Codebase**: ~55,300 lines (.cs, .razor, .css, .js)
- **Latest Version**: 1.17.19 (2026-04-14)
- **Single Instance**: Mutex(`SeoroSingleInstance`) — 다중 실행 방지

## Architecture Overview
Seoro is a cross-platform desktop GUI client for Claude Code (Anthropic's CLI).
It wraps the Claude CLI process and provides a rich Blazor-based UI for chat sessions,
git operations, file management, plugins, and more.

```
src/
  Seoro.Desktop/              # 데스크톱 앱 진입점 (10 서비스)
    Program.cs                    # Main entry - DI 컨테이너 (~75개 서비스 등록), Photino 윈도우 초기화
    Services/                     # 플랫폼별 서비스 (아래 Desktop Services 참고)
  Seoro.Shared/                # 핵심 로직 (플랫폼 독립)
    Models/                       # 데이터 모델 45개 + ViewModels/ 4개
    Services/                     # 비즈니스 로직 147개 파일 (16개 하위 폴더)
      Cli/                          # CLI 프로바이더 추상화 (7개)
      Claude/                       # Claude CLI 서비스 (9개)
      Codex/                        # Codex CLI 서비스 (3개)
      Chat/                         # 채팅 & 스트리밍 (16개)
        StreamEventHandlers/          # 스트림 이벤트 핸들러 파이프라인 (12개)
      Sessions/                     # 세션 관리 (12개)
      Git/                          # Git 통합 (15개)
      Settings/                     # 설정 관리 (8개)
      Knowledge/                    # 컨텐츠 & 지식 (12개)
      Account/                      # 계정 관리 (4개)
      Plugin/                       # 플러그인 시스템 (8개)
      Gamification/                 # 게이미피케이션 (4개)
      Notification/                 # 알림 (3개)
      Infrastructure/               # 인프라 (18개)
      Migration/                    # 스키마 마이그레이션 (9개)
      Platform/                     # 플랫폼 인터페이스 (7개)
    Components/                   # Blazor 컴포넌트 101개 (19개 폴더)
      Accounts/ Chat/ Dashboard/ Files/ Hooks/ Instructions/
      Layout/ Mcp/ Memory/ Notifications/ Onboarding/ Plugins/
      Rules/ Sessions/ Settings/ Settings/Sections/ Setup/ Shared/ Sidebar/ Tools/
    SeoroConstants.cs          # 공유 상수 및 제한값
tests/
  Seoro.Shared.Tests/          # 유닛 테스트 (xUnit)
```

## Key Services (Shared)

### CLI Provider System (멀티 CLI 추상화)
- `ICliProvider` - CLI 프로바이더 인터페이스 (Claude, Codex 등)
- `CliProviderFactory` - CLI 프로바이더 생성 팩토리
- `CliAvailabilityService` - CLI 설치 여부 및 가용성 확인
- `CliSendOptions` - CLI 전송 옵션
- `ProviderCapabilities` - 프로바이더별 기능 정의

### Claude CLI
- `ClaudeService` - Claude CLI 프로세스 관리, 스트리밍 JSON 이벤트 처리
- `ClaudeCliResolver` - Claude CLI 실행 파일 탐색 및 버전 확인
- `ClaudeArgumentBuilder` - Claude CLI 인수 조합
- `DependencyCheckService` - Claude CLI 설치 여부 및 버전 검증

### Codex CLI
- `CodexService` - Codex CLI 프로세스 관리
- `CodexCliResolver` - Codex CLI 실행 파일 탐색
- `CodexArgumentBuilder` - Codex CLI 인수 조합

### Core
- `ProcessRunner` - 프로세스 실행 추상화 (IProcessRunner)
- `ProcessErrorClassifier` - 프로세스 오류 분류 및 사용자 친화적 메시지
- `ShellService` - 셸 실행, macOS 기본 셸 자동 감지

### Chat & Streaming
- `ChatState` - UI 상태 관리 (메시지, 스트리밍, 탭)
- `ChatEventBus` - 채팅 이벤트 발행/구독 (pub-sub)
- `ChatMessageOrchestrator` - 메시지 송수신 조율
- `MessageManager` - 메시지 저장/로드/삭제
- `StreamEventProcessor` - 스트림 이벤트 파이프라인 디스패치
- `StreamingStateManager` - 스트리밍 상태 추적 (시작/진행/완료)
- `ContentGrouper` - 연속 콘텐츠 블록 그룹화
- `TabManager` - 채팅 탭 관리
- `SystemPromptBuilder` - 시스템 프롬프트 조립 (토큰 예산 관리)
- `PlanSessionTransferBuilder` - 플랜 세션 전환 빌더

### Stream Event Handlers (12개)
- `SystemInitHandler` → `MessageStartHandler` → `ContentBlockStartHandler` → `ContentBlockDeltaHandler` → `ContentBlockStopHandler` → `MessageDeltaHandler` → `AssistantMessageHandler` → `UserMessageHandler` → `ResultHandler` → `ErrorHandler`

### Session & Workspace
- `SessionService` - 세션 CRUD 및 영속화
- `SessionInitializer` - 새 세션 초기화 로직
- `SessionStatusMachine` - 세션 상태 머신 (idle/running/error)
- `SessionReplayService` - 세션 타임라인 리플레이 및 내비게이션
- `SessionListDataService` / `SessionListFacade` - 세션 목록 데이터 및 UI 파사드
- `ActiveSessionRegistry` - 활성 세션 레지스트리 (동시 세션 제한)
- `StatsCacheService` - 사용량 통계 캐싱 및 집계 (stats-cache.json)

### Git
- `GitService` / `IGitService` - Git 작업 (clone, diff, branches, merge-tree 시뮬레이션) + 캐싱
- `GitBranchWatcherService` - 실시간 Git 브랜치 추적 및 변경 감지
- `WorktreeSyncService` - Git 워크트리 동기화
- `DiffParser` - Diff 출력 파싱
- `MergeStatusService` - 세션별 머지 상태 계산 (`MergeStatusKind`: Unknown / Clean / BehindTarget / ConflictExpected / UncommittedDirty / InConflict / FetchFailed)
- `ConflictWatcherService` - `.git/MERGE_HEAD` FSW 감시로 실제 충돌 상태 실시간 발행
- `BranchRefNormalizer` - git ref 표기 정규화 (`refs/heads/`, `origin/` 등 → 순수 브랜치명)
- `GitHubUrlHelper` - GitHub 원격 URL 파싱 및 PR·비교 웹 URL 생성 (순수 함수, DI 없음)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JinoPay/Seoro](https://github.com/JinoPay/Seoro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
