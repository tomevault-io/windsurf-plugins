---
trigger: always_on
description: Multi-discipline Nix flake-based development shell for reverse engineering. Enter the environment with `nix develop` or via direnv.
---

# Reverse Engineering Environment

Multi-discipline Nix flake-based development shell for reverse engineering. Enter the environment with `nix develop` or via direnv.

## Skill System

This environment is organized into a **general-purpose core** (this file) and **discipline-specific skills** that auto-activate based on context. Skills provide specialized tool documentation, workflows, and notes for their domain.

### Available Disciplines

| Skill | Path | Activates On |
|-------|------|-------------|
| Android RE | `.claude/skills/android/SKILL.md` | APK, DEX, smali, ADB, Android app analysis |
| Windows RE | `.claude/skills/windows/SKILL.md` | PE, .exe, .dll, .sys, .NET, Windows binary analysis |
| Web RE | `.claude/skills/web/SKILL.md` | Protobuf, gRPC, HAR, HTTP API, WebSocket, TLS fingerprint, web scraping |

### Adding a New Discipline

1. Create `.claude/skills/<discipline>/SKILL.md` with front matter (`name`, `user-invocable: false`, `description` with trigger keywords).
2. Add discipline-specific tools to `flake.nix` under a `# --- <Discipline>:` comment section.
3. Add discipline-specific Python/Node dependencies to `pyproject.toml`/`package.json`.
4. Document the skill in the table above.
5. Tools shared across disciplines stay in the general sections of `flake.nix` and this file.

## Output Directory Convention

All reverse engineering work products must go in one of two locations:

- **`tmp/`** -- Intermediate and throwaway side products: decompiled source, disassembly output, extracted contents, unpacked resources, Ghidra projects, scratch scripts, etc. This directory is in `.gitignore` and will not be committed. Create subdirectories freely (e.g., `tmp/ghidra_project/`, `tmp/extracted_sample/`).
- **`artifacts/<identifier>/`** -- Final, requested deliverables: analysis reports, annotated code snippets, hook scripts, YARA rules, patch files, or anything the user explicitly asks to keep. Use a meaningful identifier as the subdirectory name (e.g., package namespace `com.example.app`, sample hash, malware family name). This directory is also in `.gitignore`: the difference from `tmp/` is durability, not tracking. Work here is meant to survive cleanup of `tmp/` and to be the thing handed back to the user, but it stays local unless the user asks to publish it elsewhere.

When running tools, always direct output into `tmp/` rather than the repo root. Examples:

```sh
ghidra  # save project to tmp/ghidra_<sample>/
r2 -A sample.bin  # any output files go to tmp/
binwalk -e firmware.bin -C tmp/binwalk_firmware/
```

Never leave tool output in the repo root or in ad-hoc directories outside these two locations.

## Environment Structure

The dev shell is defined in `flake.nix` and organized into tool categories. Python dependencies are declared in `pyproject.toml`, locked by `uv.lock`, and built into a Nix virtualenv via [uv2nix](https://github.com/pyproject-nix/uv2nix). Node.js dependencies are declared in `package.json`, locked by `package-lock.json`, and built via `importNpmLock`; bin scripts from npm packages are automatically on PATH. Ghidra's JDK is configured via `GHIDRA_JAVA_HOME`.

`flake.nix` splits the shell into `tools` (the toolchain), `devTools` (formatter and npm link
hook, dev-shell only), and `envVars` (the environment the tools need anywhere). Three outputs
fall out of that: `devShells.default` as before, `packages.re-tools` -- a `buildEnv` of the
whole toolchain, for `nix profile install` or for another flake to pull in without the shell --
and `lib.<system>.envVars`, the environment `re-tools` expects. Installing `re-tools` alone
gets the binaries but not `GHIDRA_INSTALL_DIR` or `LIBUSB1_SO`, so a consumer must set
`envVars` itself; pyghidra and pyusb both fail without them.

## Installed Tools (General-Purpose)

Discipline-specific tools are documented in their respective skill files. The tools below are available across all RE disciplines.

### Native Binary Reverse Engineering

| Tool | Command | Description |
|------|---------|-------------|
| Ghidra | `ghidra` | NSA's software reverse engineering suite with decompiler; supports x86, x64, ARM, ARM64, MIPS, and more |
| radare2 | `r2 binary` | CLI-first RE framework for disassembly, analysis, patching, and debugging |
| rizin | `rizin binary` | Modern radare2 fork with improved APIs and Ghidra decompiler integration via rz-ghidra |
| binwalk | `binwalk firmware.bin` | Scan and extract embedded files, compressed streams, and filesystems from binaries |

### Dynamic Instrumentation

| Tool | Command | Description |
|------|---------|-------------|
| frida-tools | `frida -p <pid> -l script.js` | Inject JavaScript into running processes for runtime hooking |
| frida-tools | `frida-ps` | List running processes (add `-U` for USB device, `-R` for remote) |
| frida-tools | `frida-trace -p <pid> -i "open*"` | Auto-generate handler stubs for traced functions |

### Static Analysis

| Tool | Command | Description |
|------|---------|-------------|
| YARA | `yara rules.yar target/` | Match file patterns using YARA rules for malware identification |

### Display / Monitor Firmware

| Tool | Command | Description |
|------|---------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [schlarpc/re-shell](https://github.com/schlarpc/re-shell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
