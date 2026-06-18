---
trigger: always_on
description: Guidance for AI agents working in this repository.
---

# AGENTS.md

Guidance for AI agents working in this repository.

## Project identity

`goahk` is a Windows-first automation runtime inspired by AutoHotkey v2, implemented as a Go-native, script-first library.

The primary user experience is writing small Go programs that bind hotkeys to built-in actions or arbitrary Go callbacks. Go is the scripting language. Config files are compatibility plumbing, not the product direction.

Canonical authoring style:

```go
app := goahk.NewApp()

app.Bind("Ctrl+Alt+B", goahk.SendText("basic script trigger"))

app.Bind("Ctrl+Shift+V", goahk.Func(func(ctx *goahk.Context) error {
    text, err := ctx.Clipboard.ReadText()
    if err != nil {
        return err
    }
    return ctx.Input.Paste(strings.ToUpper(text))
}))

app.Bind("Escape", goahk.ControlStop())

if err := app.Run(context.Background()); err != nil {
    log.Fatal(err)
}
```

All new public examples, docs, tests, and APIs should reinforce this model.

## Highest-priority rules

- Treat `goahk.NewApp().Bind(...).Run(...)` as the canonical workflow.
- Prefer Go functions, action helpers, callback services, and fluent builder methods over config/schema changes.
- Do not make JSON, TOML, YAML, or any other config format the primary authoring model.
- Do not introduce a custom scripting language, AHK parser, DSL, daemon, installer, service host, or machine-wide runtime unless explicitly requested.
- Preserve existing config compatibility unless the task explicitly says to change it, but keep compatibility code secondary.
- Keep simple scripts simple. Avoid speculative abstractions that make one-file automation programs harder to write.
- Return clear errors for user-facing script mistakes. Do not panic for validation, binding, hotkey, action, callback, or compatibility-config errors.
- Keep runtime internals independent from the public `goahk` package.
- Keep Windows-specific syscall, COM, unsafe, hook, SendInput, clipboard, UIA, and window code behind internal service boundaries and build tags.
- Do not use GUI/tooling code as the design center. Maintain the automation runtime and script-first API as the stable core.

## Repository map

Important areas:

- `goahk/` — public script-first Go API. This package is the primary product surface.
- `internal/program/` — internal normalized binding/program model and validation.
- `internal/runtime/` — compiler, dispatcher, supervisor, lifecycle, shutdown, control-plane/work-plane behavior.
- `internal/hotkey/` — hotkey parsing, normalization, conflict detection, listener abstractions, Windows registration/listening.
- `internal/actions/` — built-in action registry, execution, action adapters, callback bridge.
- `internal/input/` — keyboard and mouse input services.
- `internal/clipboard/` — clipboard formats, history, watch helpers, Windows/native clipboard access.
- `internal/window/` — active window, matching, enumeration, activation, geometry.
- `internal/uia/` and `internal/inspect/` — UI Automation and inspection backend services.
- `internal/process/` — launching/opening external processes or OS targets.
- `internal/app/` — app/bootstrap/lifecycle/reload wiring.
- `internal/config/` — compatibility adapter only. Do not build new primary UX here.
- `internal/testutil/` — fakes, stubs, golden helpers.
- `cmd/goahk/` — compatibility runner/CLI entrypoint.
- `cmd/goahk-inspect/` — inspection CLI.
- `examples/` — runnable script-first examples.
- `docs/` — architecture, usage, testing, build, ADRs, and code-first documentation.

Ignore GUI implementation details when making core runtime/API decisions. Do not add instructions, tests, or architecture that assume the current GUI shell is permanent.

## Public API design rules

### Shape of the API

New user-facing behavior should normally appear as one of these:

1. A small public action helper in `goahk/`.
2. A method on `*goahk.App` or `*goahk.BindingBuilder`.
3. A callback-facing service or method on `*goahk.Context`.
4. An internal runtime/action/service capability compiled from the public API.

Preferred public style:

```go
app := goahk.NewApp()

app.Bind("Ctrl+Alt+T", goahk.SendText("hello"))

app.On("Ctrl+Shift+R").Replace().Do(goahk.Func(func(ctx *goahk.Context) error {
    for i := 0; i < 10; i++ {
        if err := ctx.Err(); err != nil {
            return err
        }
        ctx.Log("working", "step", i)
        if !ctx.Sleep(250 * time.Millisecond) {
            return ctx.Err()
        }
    }
    return nil
}))
```

Avoid adding public APIs that require users to construct internal model structs directly.

### Naming conventions

Public Go API naming should be concise, idiomatic, and script-friendly:

- Constructors use `NewX` when they create durable values, for example `NewApp`.
- Binding verbs use short imperative names: `Bind`, `On`, `Do`, `Run`.
- Fluent policy methods use PascalCase action words: `Serial`, `Drop`, `QueueOne`, `Parallel`, `Replace`.
- Public action helpers use PascalCase and read like script commands:
  - `SendText`
  - `SendKeys`
  - `SendChord`
  - `Open`
  - `OpenURL`
  - `OpenFolder`
  - `Launch`
  - `StartApplication`
  - `ClipboardRead`
  - `ClipboardWrite`
  - `ClipboardAppend`
  - `ClipboardPrepend`
  - `ActivateWindow`
  - `CopyActiveWindowTitle`
  - `ListOpenApplications`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [multiplex55/goahk](https://github.com/multiplex55/goahk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
