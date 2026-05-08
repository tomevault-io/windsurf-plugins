---
trigger: always_on
description: Go CLI for [Qdrant Cloud](https://cloud.qdrant.io), built with Cobra / Viper and gRPC.
---

# qcloud-cli

## Project overview

Go CLI for [Qdrant Cloud](https://cloud.qdrant.io), built with Cobra / Viper and gRPC.

- **Module:** `github.com/qdrant/qcloud-cli`
- **Binary:** `qcloud` (built to `build/qcloud`)
- **Go version:** 1.26+
- **Key dependencies:** `cobra`, `viper`, `google.golang.org/grpc`, `qdrant-cloud-public-api` (generated gRPC stubs)

## Project structure

```
cmd/qcloud/              # main entrypoint — creates State, builds root command, runs it
internal/
  cli/                   # root cobra command, global flags, subcommand registration
  cmd/                   # one sub-package per top-level subcommand
    cluster/             # cluster.go (parent) + list/describe/create/delete
    version/             # version subcommand
    clusterutil/         # shared cluster helpers (e.g. wait-for-healthy)
    output/              # shared output formatting helpers
    util/                # shared command utilities
  qcloudapi/             # gRPC client wrapper for the Qdrant Cloud API
  state/                 # State struct (shared deps: config, lazy gRPC client)
    config/              # Viper-based config (file, env vars, flags
```

## Build & verification

**Always use Makefile targets — never raw `go build`, `go test`, or linter commands.**

| Target           | What it does                                  |
|------------------|-----------------------------------------------|
| `make build`     | Compile binary to `build/qcloud`              |
| `make test`      | Run all tests                                 |
| `make lint`      | Run golangci-lint (installs it if missing)    |
| `make format`    | Run golangci-lint with `--fix`                |
| `make bootstrap` | Install tool dependencies via `mise install`  |
| `make clean`     | Remove build artifacts                        |

To verify your changes, you should run the following makefile targets:
```bash
make lint
make build
make test
```

If make lint fails from formatting problems, use `make format` to fix them.

## Conventions

### Long descriptions and examples — mandatory

Every leaf command and group command **must** have a `Long` description and an `Example` block.

**`Long`:**
- First line expands the `Short` description into a full sentence.
- Blank line, then one or two paragraphs explaining behaviour, use cases, and important caveats.
- Use the proto service/message comments as the authoritative source of truth for what a resource or operation does.
- Do NOT describe individual flags — only document unusual or non-obvious flag interactions.

**`Example`:**
- One example per meaningful use case (basic call, common flag combinations, scripting).
- Prefix every line with `# ` comment explaining what the example does.
- Real command invocations with plausible IDs/values.

All five base types (`ListCmd`, `DescribeCmd`, `CreateCmd`, `UpdateCmd`, `Cmd`) expose `Long` and `Example` as top-level struct fields. Never set them inside `BaseCobraCommand()`.

### Tests — mandatory

Every new command package **must** ship tests. This is not optional.

- Place tests in `internal/cmd/<group>/` as `<file>_test.go` using `package <group>_test`.
- Use `testutil.NewTestEnv` + `testutil.Exec` — never call command functions directly.
- When adding a new gRPC service, also add a `fake_<service>.go` in `internal/testutil/` and register it in `server.go` / `TestEnv`.
- Cover: table output (assert header columns + key values), JSON output (unmarshal and assert), request fields sent to server, backend errors (use `Returns(nil, fmt.Errorf(...))` and assert `require.Error`), input errors (missing args, wrong flags).
- Run `make test` before declaring done.

### Subcommand pattern

Each subcommand group lives in `internal/cmd/<group>/`:

1. A public `NewCommand(s *state.State) *cobra.Command` creates the parent and registers sub-commands.
2. Leaf commands are unexported (`newListCommand`, `newDeleteCommand`, …).
3. All commands receive `*state.State` — use it to access config and the lazy gRPC client.

### State passing

`main` → `state.New(version)` → passed to every command constructor. Commands call `s.Client(ctx)` to get the gRPC client (created on first use) and `s.AccountID()` for the current account.

### Base command types (`internal/cmd/base`)

All leaf commands are built using one of five generic base types. Always prefer these over raw `cobra.Command`.

#### `base.ListCmd[T]`
For listing resources. `OutputTable` must be set. The base automatically registers `--no-headers` and handles header suppression. By default the command takes no positional args; set `Args` to accept them.

```go
base.ListCmd[*foov1.ListFoosResponse]{
    Use:   "list",
    Short: "List all foos",
    Fetch: func(s *state.State, cmd *cobra.Command) (*foov1.ListFoosResponse, error) {
        // call gRPC, return response
    },
    OutputTable: func(_ *cobra.Command, w io.Writer, resp *foov1.ListFoosResponse) (output.TableRenderer, error) {
        t := output.NewTable[*foov1.Foo](w)
        t.AddField("ID", func(v *foov1.Foo) string { return v.GetId() })
        t.SetItems(resp.GetItems())
        return t, nil
    },
}.CobraCommand(s)
```

#### `base.DescribeCmd[T]`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [qdrant/qcloud-cli](https://github.com/qdrant/qcloud-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
