---
trigger: always_on
description: Keyboard-driven TUI for Docker Swarm management ("k9s for Docker Swarm"). Single static Go binary, Bubble Tea framework, zap logging.
---

# SwarmCLI

Keyboard-driven TUI for Docker Swarm management ("k9s for Docker Swarm"). Single static Go binary, Bubble Tea framework, zap logging.

## Quick Reference

```bash
# Build & run
go build -v -o swarmcli .
SWARMCLI_ENV=dev LOG_LEVEL=debug go run .

# Unit tests
go test ./...

# Integration tests (full E2E against real Docker Swarm via DinD)
./test-setup/testenv.sh integration           # up → deploy → test → down
./test-setup/testenv.sh up                    # start Swarm environment only
./test-setup/testenv.sh deploy                # deploy test stack
./test-setup/testenv.sh test                  # run integration tests
./test-setup/testenv.sh test TestScaleWhoami  # single test
./test-setup/testenv.sh down                  # teardown + cleanup
KEEP=1 ./test-setup/testenv.sh integration    # keep env running after tests
TEST_LOG=1 ./test-setup/testenv.sh test       # enable test logging

# Lint (CI uses this)
golangci-lint run ./... --build-tags=integration

# Logs
tail -f ~/.local/state/swarmcli/app-debug.log   # dev mode
tail -f ~/.local/state/swarmcli/app.log          # prod mode (JSON)
```

## Architecture

```
main.go                    Entry point; version injection via ldflags, tea.NewProgram()
app/
  app.go                   View factory registry, Init(), command autoload via _ "swarmcli/commands"
  hooks.go                 PreUpdateHook registration; StartupOverlay; RegisterShutdownHook / RunShutdownHooks (BE port-forward manager registers CloseAll here)
  model.go                 Central state: Model struct (viewport, currentView, viewStack, commandInput, searchInput, systemInfo)
  update.go                Main message router: navigation, resize, events, key dispatch
views/
  view/interface.go        View contract: Update/View/Init/Name/OnEnter/OnExit/HasErrors/ShortHelpItems + Filterable interface
  stacks/                  Stack list → drill into services
  services/                Service list (filterable by stack/node/all), scale/restart actions
  tasks/                   Task list per service/stack
  nodes/                   Cluster node list
  secrets/                 Secret management
  configs/                 Config management
  logs/                    Service log streaming
  contexts/                Docker context switcher
  help/                    Keybinding cheat sheet
  inspect/                 JSON inspect viewer
  networks/                Network list
  loading/                 Loading spinner
  commandinput/            ":" command bar
  searchinput/             "/" search filter bar (app-level, drives Filterable views)
  confirmdialog/           Confirmation prompts
  scaledialog/             Scale replica input
  helpbar/                 Dynamic keybinding bar
  systeminfo/              Header with cluster info
  viewstack/               Navigation stack (push/pop)
commands/
  api/                     Command context & arg parsing
  command/                 Built-in commands (help, contexts, stacks, services, etc.)
  autoload.go              Blank import triggers init() registration
docker/
  client.go                Context-aware Docker client factory
  snapshot.go              In-memory cache (3s TTL, sync.RWMutex, atomic refresh flag)
  events.go                Docker event stream subscription
  service.go               Service ops: scale, restart, update
  node.go, task.go         Entity queries (TaskEntry includes ContainerID, populated from task.Status.ContainerStatus.ContainerID with nil-guard)
  stack.go                 Stack queries
  secret.go, config.go     Secret/config CRUD
registry/
  registry.go              Global command map: Register(), Get(), All(), Suggest()
utils/log/
  logger.go                zap wrapper: Init(), L(), Sync(), SetLevel(), lumberjack rotation
```

## Key Patterns

- **Bubble Tea MVC**: Input → Update() → tea.Cmd → View(). All state changes via `tea.Msg` types.
- **View Stack**: `viewStack.Push(old)` / `Pop()` for breadcrumb navigation.
- **View Factory**: `viewRegistry[name]` maps view names to constructor functions, registered in `app.Init()`.
- **Command Registry**: Commands in `commands/command/` auto-register via `init()` + `registry.Register()`. Accessed via `:` input.
- **Command Spec**: Commands optionally implement `registry.CommandWithSpec` (`Spec() registry.CommandSpec`, discovered by type assertion like `Aliaser`). The spec declares `Usage`, `Flags` (the allow-list), and `Examples`. `api.ParseInput` is the single chokepoint that, in order: short-circuits `Passthrough` specs, intercepts `--help`/`-h`/`-help` (and `:help <cmd>`) into a per-command help screen reusing the detailed help view, then rejects any undeclared flag (**global strict**, with a `did you mean --x?` suggestion). Unknown-flag rejection means every registered command MUST declare a spec — a missing/empty spec rejects all flags. `Passthrough:true` is the narrow escape-hatch for delegating/unavailable stubs (e.g. the OSS `bootstrap` stub): it skips both help interception and validation so every arg reaches `Execute` unchanged and the command keeps its own messaging (and no Pro flag internals leak into OSS — see Pro Feature Boundary).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Eldara-Tech/swarmcli](https://github.com/Eldara-Tech/swarmcli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
