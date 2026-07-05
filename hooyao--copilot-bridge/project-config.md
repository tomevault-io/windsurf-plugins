---
trigger: always_on
description: Guidance for AI coding agents (Antigravity, Gemini CLI, Claude Code, …) working
---

# AGENTS.md

Guidance for AI coding agents (Antigravity, Gemini CLI, Claude Code, …) working
in this repo. Antigravity / Gemini load this file automatically; Claude Code
reads `CLAUDE.md`, which points here. **This file is the single source of truth
for how to build, run, and contribute** — `docs/` holds the deep references.

## What this is

A .NET 10 **Native AOT** reverse proxy that re-exposes the **GitHub Copilot LLM
API** under per-client URL prefixes, so Claude Code / Codex / Gemini CLI can use
Copilot as their model backend. Ships as a single **~12 MB native binary** with
no .NET runtime dependency — Native AOT is chosen specifically to keep the binary
small, so trimming + source-generated JSON are mandatory, not optional. Builds
for **win-x64, win-arm64, linux-x64, osx-arm64** (each on its own runner — Native
AOT cannot cross-OS compile).

**As-built (M1, done):** Claude Code (Anthropic Messages shape) →
`POST /cc/v1/messages` → **byte-level passthrough** to Copilot's *native*
`/v1/messages` endpoint. There is **no** Anthropic↔OpenAI translation on this
path: Copilot turned out to expose a native Anthropic endpoint (discovered after
the first design; see `docs/design.md` v0.2). Translating to OpenAI (Codex) and
Gemini shapes is future work (M3/M4).

> ⚠️ **Watch for stale "planned" prose.** Parts of `CLAUDE.md` ("Architecture
> (planned)", "first milestone … translated to Copilot OpenAI Chat Completions")
> and the original design doc describe an OpenAI-Chat-Completions translation
> path. That was the **pre-research plan**; the shipped M1 is Anthropic
> passthrough. When two docs disagree, trust **`docs/pipeline-design.md`** (the
> architectural contract) and **`README.md`** (as-built status).

## Environment

- **Cross-platform**: builds + runs on Windows, Linux, and macOS. Development is
  primarily on **Windows** + PowerShell (Bash also available for POSIX scripts).
- **.NET 10 SDK**.
- **AOT linker toolchain** (only needed to *publish* the native binary, per OS;
  JIT build/run/test need none of it): Windows → **Visual Studio C++ Build Tools**
  + **Windows SDK**; Linux → **`clang` + `zlib1g-dev`**; macOS → **Xcode Command
  Line Tools**. You can only AOT-build for the OS you are on — Native AOT does not
  cross-OS compile, so the four release RIDs each build on their own CI runner.

## Setup

```pwsh
git clone https://github.com/hooyao/copilot-bridge
cd copilot-bridge
```

**`references/` is not in the repo** (gitignored — it's a separate, read-only
checkout). Several docs and code comments cite it as the canonical Copilot
protocol reference. To get it:

```pwsh
git clone https://github.com/ericc-ch/copilot-api references/copilot-api
```

Read it before touching protocol-level code (headers, auth flow, model ids).
**Never edit anything under `references/`.**

## Build / run / publish / test

```pwsh
# Develop (JIT, fast iteration)
dotnet run --project src/CopilotBridge.Cli -- serve      # default port 8765

# Debug build (whole solution)
dotnet build CopilotBridge.slnx

# Unit tests — pure logic, no network/Copilot. This is what CI runs.
dotnet test tests/CopilotBridge.UnitTests
# Solution-wide, skipping the live-Copilot integration harness:
dotnet test CopilotBridge.slnx --filter "Category!=Integration"
# Integration harness — real claude.exe + live Copilot login (tests/harness/
# README.md). Tagged [Trait("Category","Integration")]; run explicitly.
dotnet test tests/CopilotBridge.Playground
dotnet test --filter FullyQualifiedName~<TestName>       # single test
```

**Publish the single-file AOT binary (the release artifact):**

```pwsh
# Windows (local convenience wrapper):
.\build-aot.bat
#  → .\publish\copilot-bridge.exe

# Any OS, explicit RID (win-x64 | win-arm64 | linux-x64 | osx-arm64):
dotnet publish src/CopilotBridge.Cli -c Release -r <rid> -o ./publish
```

`build-aot.bat` handles the **Windows-local** AOT-linker prerequisites for you:
it adds `vswhere.exe` to PATH, uses it to locate the VS install, runs
`VsDevCmd.bat` to put MSVC `link.exe` on PATH, then calls `dotnet publish`.

⚠️ **Why the script locally (don't skip it on a Windows dev box):** a bare
`dotnet publish -c Release` fails the AOT native-link step on most local setups.
The compiler (ILC) shells out to `vswhere.exe` to find `link.exe`, but `vswhere`
(`C:\Program Files (x86)\Microsoft Visual Studio\Installer\`) isn't on PATH by
default — not even inside a VS Developer prompt — so it fails with
`'vswhere.exe' is not recognized`. The script fixes exactly that. JIT
`run`/`build`/`test` need none of this. **CI does not use the script**: the
GitHub-hosted images expose the toolchain (MSVC / clang / Xcode CLT) to the SDK's
AOT targets directly, so the workflows call a plain `dotnet publish -r <rid>`.
The release pipeline builds all four RIDs, each on its own runner
(`windows-latest`, `windows-11-arm`, `ubuntu-latest`, `macos-14`), and attaches
every archive (+ a macOS `.pkg`) to one GitHub Release.

> **Agents publishing manually (when you can't run `build-aot.bat` directly):**
> importing the VsDevCmd environment is **not enough** — VsDevCmd does not add the
> VS Installer dir (where `vswhere.exe` lives) to PATH, and ILC's link step calls a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hooyao/copilot-bridge](https://github.com/hooyao/copilot-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
