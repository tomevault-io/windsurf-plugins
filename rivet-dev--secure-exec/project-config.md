---
trigger: always_on
description: - primary accent color: #38BDF8 (bright sky blue), light variant: #7DD3FC
---

## Brand

- primary accent color: #38BDF8 (bright sky blue), light variant: #7DD3FC
- secondary accent color: #CC0000 (red), light variant: #FF3333
- website: https://secureexec.dev
- docs: https://secureexec.dev/docs
- GitHub: https://github.com/rivet-dev/secure-exec
- GitHub org is `rivet-dev` — NEVER use `anthropics` or any other org in GitHub URLs for this repo
- the docs slug for Node.js compatibility is `nodejs-compatibility` (not `node-compatability` or other variants)

## NPM Packages

- every publishable package must include a `README.md` with the standard format: title, tagline, and links to website, docs, and GitHub
- if `package.json` has a `"files"` array, `"README.md"` must be listed in it
- **no hardcoded monorepo/pnpm paths** — NEVER resolve dependencies at runtime using hardcoded relative paths into `node_modules/.pnpm/` or monorepo-relative `../../../node_modules/` walks; use `createRequire(import.meta.url).resolve("pkg/path")` or standard Node module resolution instead
- **no phantom transitive dependencies** — if published runtime code calls `require.resolve("foo")` or `import("foo")`, `foo` MUST be declared in that package's `dependencies` (not just available transitively in the monorepo)
- **`files` array must cover all runtime references** — if compiled `dist/` code resolves paths outside `dist/` at runtime (e.g., `../src/polyfills/`), those directories MUST be listed in the `"files"` array; verify with `pnpm pack --json` or `npm pack --dry-run` before publishing

## Testing Policy

- NEVER mock external services in tests — use real implementations (Docker containers for databases/services, real HTTP servers for network tests, real binaries for CLI tool tests)
- tests that validate sandbox behavior MUST run code through the secure-exec sandbox (NodeRuntime/proc.exec()), never directly on the host
- CLI tool tests (Pi, Claude Code, OpenCode) must execute inside the sandbox: Pi runs as JS in the VM, Claude Code and OpenCode spawn their binaries via the sandbox's child_process.spawn bridge
- for host-binary CLI/SDK regressions (Claude Code, OpenCode), pair the sandbox `child_process.spawn()` probe with a direct `kernel.spawn()` control for the same binary command; if direct kernel command routing works but sandboxed spawn hangs, the blocker is in the Node child_process bridge path rather than the tool binary, provider config, or HostBinaryDriver mount
- sandbox `child_process.spawn()` does not yet honor `stdio` option semantics for host-binary commands, so headless CLI tests that need EOF on stdin should explicitly call `child.stdin.end()` instead of assuming `stdio: ['ignore', ...]` will close it
- real-provider CLI/SDK tool-integration tests must stay opt-in via an explicit env flag and load credentials at runtime from exported env vars or `~/misc/env.txt`; never commit secrets or replace the live provider path with a mock redirect when the story requires real traffic
- real-provider NodeRuntime CLI/tool tests that need a mutable temp worktree must pair `moduleAccess` with a real host-backed base filesystem such as `new NodeFileSystem()`; `moduleAccess` alone makes projected packages readable but leaves sandbox tools unable to touch `/tmp` working files
- e2e-docker fixtures connect to real Docker containers (Postgres, MySQL, Redis, SSH/SFTP) — skip gracefully via `skipUnlessDocker()` when Docker is unavailable
- interactive/PTY tests must use `kernel.openShell()` with `@xterm/headless`, not host PTY via `script -qefc`
- before fixing a reported runtime, CLI, SDK, or PTY bug, first reproduce the broken state and capture the exact visible output (stdout, stderr, event payloads, or terminal screen) in a regression or work note; do not start by guessing at the fix
- terminal-output and PTY-rendering bugs must use snapshot-style assertions against exact strings or exact screen contents under fixed rows/cols, not loose substring checks
- if expected terminal behavior is unclear, run the same flow on the host as a control and compare the sandbox transcript/screen against that host output before deciding what to fix
- be liberal with structured debug logging for complex interactive or long-running sessions so later manual repros can be diagnosed from artifacts instead of memory
- debug logging for complex sessions should go to a separate sink that does not contaminate stdout/stderr protocol output; prefer structured `pino` logs with enough context to reconstruct process lifecycle, PTY events, command routing, and failures, while redacting secrets
- kernel blocking-I/O regressions should be proven through `packages/core/test/kernel/kernel-integration.test.ts` using real process-owned FDs via `KernelInterface` (`fdWrite`, `flock`, `fdPollWait`) rather than only manager-level unit tests
- inode-lifetime/deferred-unlink kernel integration tests must use `InMemoryFileSystem` (or another inode-aware VFS) and await the kernel's POSIX-dir bootstrap; the default `createTestKernel()` `TestFileSystem` does not exercise inode-backed FD lifetime semantics

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rivet-dev/secure-exec](https://github.com/rivet-dev/secure-exec) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
