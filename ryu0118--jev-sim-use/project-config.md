---
trigger: always_on
description: CLI that drives an iOS Simulator / Android device toward a natural-language goal. It shells out to
---

# jev-sim-use

CLI that drives an iOS Simulator / Android device toward a natural-language goal. It shells out to
[sim-use](https://github.com/lycorp-jp/sim-use) (lycorp-jp, Apache-2.0) to observe and act, and asks
Jev through [swift-jev](https://github.com/d-date/swift-jev) (MIT) which action to take next.

Why it exists: speed. Each step is one small typed Jev call instead of a frontier LLM agent spending a reasoning turn
per tap. Keep it that way: one Jev request per step, no extra round trips, and deterministic work stays in code.

## Development workflow

- `mise run setup` — install tools, configure Git hooks
- `mise run check` — format, lint, AST lint, build, test, docsync
- `mise run test` — run the test suite
- `mise run contract-test` — check the installed sim-use against `SimUseContract` (needs a booted device); run it after upgrading sim-use, then bump `SimUseBootstrap.testedVersion`
- See `.mise.toml` for the full task list (`mise tasks`)
- Git hooks in `.githooks/`: pre-commit runs gitleaks, format, lint, AST lint, docsync; pre-push runs AST lint
- Keep commits small and easy to revert

## Architecture

- `JevSimUse` (executable, binary `jev-sim-use`): `@main` only; starts `JevSimUseCommand`.
- `JevSimUseCLI` (+ `JevSimUseCLITests`): ArgumentParser commands `run` (default, positional goal), `session`
  (`list` / `show` / `tell` / `forget` / `resume`), `exec`
  (execv sim-use with arguments passed through), `doctor`, `config`. Thin: parse, `validate()`, build a request, call
  one Kit Runner, present the outcome, map failures to exit codes (`ExitStatus`: 2 setup, 3 runtime).
  - Commands conform to `ContextualCommand` and take a `CLIContext` (injectable `CLIOutput` + environment); `.live` is
    the only place the CLI reads `ProcessInfo`. CLI tests use `RecordingOutput` and a `FakeSimUse` script on `PATH`.
- `JevSimUseKit` Runners (return values, never print):
  - `RunGoalRunner` (`Agent/`): resolves `JevSettings`, pins the device (`--device` > `$SIM_USE_DEVICE` > the only
    usable device), builds the `RoutingPolicy`, runs `AgentLoop`, reports `RunGoalEvent`s. Every run belongs to a
    session (`SessionStart.new` or `.resume`), saved before and after the loop and deleted once the goal is reached.
    Unfinished sessions expire a week after they last changed (`SessionStore.timeToLive`, pruned on every run and
    `session` command).
  - `SessionRunner` (`Session/`): list / show / tell on `SessionStore` (`$XDG_STATE_HOME/jev-sim-use/sessions`).
  - `DoctorRunner` (`Doctor/`): sim-use, device (reads the screen once), and Jev settings checks → `DoctorReport`.
  - `ConfigRunner` (`Configuration/`): get / set (validated) / unset / list on `UserConfigStore`.
  - `FailureCategory` classifies Runner errors as setup vs runtime.
- `JevSimUseKit/Process`: `CommandRunning` seam; `SubprocessCommandRunner` runs commands through swift-subprocess 1.0
  via ProcessRunning, which collects both streams concurrently and stops reading once the child exits.
- `JevSimUseKit/SimUse`: locate sim-use on `PATH` through `FileManagerProtocol` (not via `/usr/bin/env`, so "not
  installed" is distinct from exit 127), version gate, device pinning, and `--json` envelope decoding.
- `JevSimUseKit/Session`: the supervisor loop. A frontier agent reads `session show` and `exec ui`, adds facts with
  `session tell`, and `session resume`s; there are no per-run hint flags. Resume continues `history`, `notes`, and step
  numbers, but `maxSteps` and loop detection (`AgentProgress`) start fresh, so a stalled or step-limited run can move.
  `UserDirectories` is the one resolver for `HOME` / `XDG_*`.
- `JevSimUseKit/Configuration`: `JevSettings` resolves flag > env > `UserConfig` file > default for the base URL
  (`/v1/systemone` appended) and model. The key comes only from `TYPESAFE_API_KEY`. The tool speaks only TypeSafe's
  wire format; other providers go behind a compatible proxy. `UserConfigStore` uses `FileManagerProtocol`.
- `JevSimUseKit/Agent`: `AgentLoop` observe → plan → act. `JevStepPlanner` sends one request asking which
  operation to run, which target it would use, and whether it would finish the goal.
- `JevSimUseKit/Skill`: `SkillRunner` installs / uninstalls / prints the agent skill. `SkillBundle+Generated.swift` embeds
  `skills/jev-sim-use/` (SSoT: SKILL.md plus `references/*.md`, which SKILL.md links to and `skill install` writes
  alongside it) via `mise run generate-skill`, guarded by `SkillBundleDriftTests`. CLI:
  `jev-sim-use skill install|uninstall|print` (`--client claude|agents` or `--dest`), mirroring `sim-use init`.
- Distribution: `.claude-plugin/marketplace.json` + `.claude/plugins/jev-sim-use` (Claude Code),
  `.agents/plugins/marketplace.json` + `plugins/jev-sim-use` (Codex), `apm.yml` + `.apm/skills` (APM); skill dirs are
  symlinks to `skills/jev-sim-use`. `release.yml` bumps all manifest versions; `install.sh` is the curl installer.
  The docsync rule `skill-cli` ties SKILL.md to the CLI options and `AgentOutcome`: after changing them, update
  SKILL.md, run `mise run generate-skill`, then `docsync update-checksum`.

## sim-use contract (verified against v0.14.0)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Ryu0118/jev-sim-use](https://github.com/Ryu0118/jev-sim-use) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
