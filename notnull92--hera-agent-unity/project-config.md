---
trigger: always_on
description: CLI tool to control Unity Editor from the command line.
---

# hera-agent-unity

CLI tool to control Unity Editor from the command line.
Unified successor to `hera-agent` + `hera-agent-pro`. All features ship free under MIT.

## 설계 의도

**CLI(단일 Go 바이너리 + localhost HTTP) 구성은 의도된 선택이다.** MCP / WebSocket relay / 영속 서버 / Python 런타임 같은 대안으로 전환하자는 제안은 하지 말 것.

이유:
- 런타임 의존성 0개 — 사용자는 바이너리 하나 + UPM 패키지 하나만 설치
- Stateless — 모든 요청이 독립적이라 세션·재연결 로직 불필요
- 도메인 리로드를 파일시스템 버스(`~/.hera-agent-unity/instances/`, `status/`)로 우회
- 어떤 셸·AI 에이전트·스크립트에서도 호출 가능 (MCP 클라이언트에 묶이지 않음)

**파생 원칙** — decoupled/비대칭이 *의도된* 곳에 결합·통일 제안 금지:

- **CLI ↔ Connector 버전 핸드셰이크 불필요**: 두 버전이 일치한다는 전제 자체가 없음. HTTP+JSON forward-compat과 동적 dispatch가 자연 처리. "버전 매칭 검사 추가하자"는 제안은 모델 밖.
- **양방향/스트리밍 채널 없음**: 단발성 호출이 디폴트. "lock 점유자 보여달라", "진행률 스트림", "실시간 알림" 같은 제안은 모델 밖.
- **단일 에디터 모델 (멀티 에디터 미지원)** 🔒: 한 머신에 Unity 에디터 하나를 전제. 포트 바인딩 구조상 같은 머신에서 멀티 에디터는 실사용 불가 — instance discovery 는 "한" 인스턴스를 해석하고, 재시도·재해석(`doWithReloadRetry` 의 `DiscoverInstance` 포트 추종 등)이 *다른* 에디터를 집을 위험은 **모델 밖**. "여러 에디터 구분/디스앰비규에이션 추가하자", "재해석이 substring 매치로 잘못된 에디터를 고를 수 있다", "PID 로 정확 매칭하자" 같은 지적·제안은 모델 밖 — 멀티 인스턴스 충돌은 발생하지 않는 전제이므로 새 문제로 제기 금지.
- **출력 비대칭은 명령별로 분리** — 세 부류:
  - **표준 envelope tool 명령** (`exec`, `editor`, `console`, `scene`, `menu`, `screenshot`, `reserialize`, `test`, `profiler`, `list`, `describe_type`, `find_method`, `list_assemblies`, `batch`, `log`, `manage_gameobject`, `find_gameobjects`, `manage_components`, `manage_packages`, `unity_docs`, `describe_shader`, `manage_material`, `manage_prefab`, `manage_asset_import`, `manage_ui`, custom tools): 성공/실패 응답은 **compact JSON** 으로 통일 — AI agent 가 소비. 박스 drawing / ANSI escape / 한국어 banner 금지. `humanCategories` 화이트리스트(`cmd/root.go`)에 없으면 자동으로 compact + stderr 장식 억제.
  - **human 명령** (`install`, `uninstall`, `status`, `update`, `doctor`, `help`, `version` + 별칭): `humanCategories` 화이트리스트 등재. `tui.ErrorPanel` / `BoxAccent` / banner / `printUpdateNotice` 유지.
  - **자체 출력 경로 명령** (`asset-config`, `ping`): `printResponse` 를 거치지 않고 직접 출력. `asset-config` 는 기본 styled + `--json` 로 AI 모드. `ping` 은 단일 라인 `port=N alive=N state=... age_ms=N`. `doctor` 도 human 카테고리지만 `--json` / `--agent-rules` 분기 별도.
  - "tool 에러도 인간이 읽는다"는 가정은 audience reality와 어긋남 (실제로 tool 호출 = AI). 새 명령 추가 시 `humanCategories` 등재 여부가 출력 모드를 결정한다.

새 기능을 추가할 때도 이 모델 안에서 풀 것: HTTP 한 번 / 필요하면 파일 폴링.

## Structure

```
cmd/                  # Go CLI — thin passthrough layer
  root.go             # Entry point, flag/arg parsing, humanCategories, default passthrough
  editor.go           # editor command (waitForReady polling)
  test.go             # test command (PlayMode result polling via pollResultFile)
  poll.go             # shared pollResultFile: file-bus result poller w/ exponential
                      # backoff (100ms→1.5s) + state/PID liveness checks (test + packages)
  status.go           # status + ping + waitForAlive/waitForState/waitForReady
                      # (heartbeat reads, same backoff)
  update.go           # self-update from GitHub releases (download + rename dance)
  version_check.go    # periodic update notice (12h interval, human-only)
  asset_config.go     # asset plugin config (TUI default + --json for AI)
  batch.go            # batch (multi-command) dispatch + --dry-run preview
  manage_packages.go  # async job_id dispatch + pollResultFile (file-bus, like test)
  unity_docs.go       # thin passthrough — connector ships its own data set
  install.go          # self-install onto PATH + legacy scrub
  uninstall.go        # self-uninstall (+ uninstall_{unix,windows}.go variants)
  doctor.go           # self-diagnostic; embeds AGENT.md for --agent-rules
  paths.go            # install path resolution (+ paths_windows.go)
  path_check.go       # per-command PATH-mismatch warning (HERA_AGENT_NO_PATH_CHECK)
  deferred_delete_*.go # Windows-safe .bak cleanup after self-update
  AGENT.md            # embedded copy for `doctor --agent-rules` (go:embed)
internal/client/      # Unity HTTP client, instance discovery, SendBatch
                      # + process_{unix,windows}.go (PID liveness check)
internal/assetconfig/ # Asset plugin configuration persistence (config.go only)
internal/tui/         # Terminal UI helpers: style.go, assetconfig.go (bubbletea), detect.go
tools/build-unity-docs/ # One-shot maintainer Go script: Documentation/en/ScriptReference
                        # → unity_docs_<ver>.jsonl(.gz)(.bytes). Run per Unity version.
AgentConnector/       # C# Unity Editor package (UPM) — package.json holds version
  Editor/
    HttpServer.cs     # [InitializeOnLoad] HttpListener + queue + main-thread pump
    CommandRouter.cs  # SemaphoreSlim lock (120s) + Dispatch / DispatchBatch
    Heartbeat.cs      # 1.0s atomic write to ~/.hera-agent-unity/instances/<md5>.json
    ToolDiscovery.cs  # [HeraTool] reflection cache + Levenshtein "did you mean"
    HeraAgent.asmdef
    HeraAgentAssetConfigWindow.cs   # Editor GUI for asset-config
    Core/             # Response, ParamCoercion, ToolParams,
                      # StringCaseUtility, ToolMetadata, UnityPitfalls,
                      # HierarchyPath (Build: Transform→path; Find:
                      # path→GameObject, inactive-aware — shared by
                      # manage_gameobject/components/ui),

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NotNull92/hera-agent-unity](https://github.com/NotNull92/hera-agent-unity) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-08 -->
