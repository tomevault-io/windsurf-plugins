---
trigger: always_on
description: Tako's protocol is v0: do not keep any legacy code, backward compatibility shims, or deprecated paths — break things freely until protocol v1.
---

# Instructions for AI agents working on the Tako codebase

Tako's protocol is v0: do not keep any legacy code, backward compatibility shims, or deprecated paths — break things freely until protocol v1.

## Key Principles

1. **Keep SPEC.md in sync with code** - Whenever you modify code, update SPEC.md if needed (avoid insignificant implementation details; focus on user-facing behavior and architecture)

2. **Move finalized behavior to SPEC.md** - Keep SPEC.md as the source of truth for implemented behavior; keep planning out of in-repo TODO files

3. **TDD is mandatory for Rust crates and SDK code** - Write tests before implementation. No backend/SDK feature is complete without passing tests. Website (`website/`) changes are explicitly exempt.

4. **Don't over-engineer** - Simple is better than complex. Avoid premature abstractions, extra configurability, or handling scenarios that can't happen

5. **Trust internal code** - Only validate at system boundaries (user input, external APIs). Don't add defensive checks between trusted internal components

6. **Keep README files in sync for touched components** - When code changes setup, commands, or run/test flow, update the relevant README.md files. README content should stay basic and practical (what it is, how to run), not a specification.

7. **Keep SPEC-derived website docs in sync** - The following docs are agent-maintained outputs derived from `SPEC.md` and must be updated whenever `SPEC.md` behavior changes:
   - `website/src/pages/docs/how-tako-works.md`
   - `website/src/pages/docs/tako-toml.md`
   - `website/src/pages/docs/presets.md`
   - `website/src/pages/docs/troubleshooting.md`
   - `website/src/pages/docs/cli.md`
   - `website/src/pages/docs/deployment.md`
   - `website/src/pages/docs/development.md`

8. **Runtime behavior lives in plugins** - Runtime definitions (install commands, launch args, entrypoint paths) are in `tako-runtime/src/plugins/`. Preset definitions live in `presets/`.
   - `presets/{language}.toml` — family preset definitions (sections per preset)

9. **Never commit with known failures** - Do not commit when tests or pre-commit hooks are known to be failing. Fix the issues first. If fixing is impractical, get explicit user confirmation before committing.

10. **Keep files small and focused** - No single source file should grow beyond ~800 lines. When adding code to a file that's already large, split it first. But don't split by line count alone — split by responsibility. Ask "is this the right boundary?" not just "is this file too big?" Each module should have one clear responsibility. Prefer sibling submodules (e.g. `commands/init/scaffold.rs`) over letting a file accumulate unrelated concerns. Tests belong in their own `tests.rs` submodule, not inline in large files.

11. **Don't test against removed code or features** - Never write assertions that check a removed symbol, field, flag, message, or API is absent (e.g. `assert!(!output.contains("OLD_NAME"))`, `expect(...).not.toContain("deprecated-flag")`). Once the feature is gone from the code, it cannot reappear, and the test becomes dead weight that rots over time. Negative assertions are only valid when they encode a _current_ behavioral distinction (e.g. "in CI mode, ANSI colors are suppressed"; "NODE_ENV appears in ProcessEnv but not in TakoBaseEnv"). If you catch yourself adding `!contains(...)` for something that no longer exists anywhere in the codebase, delete the assertion instead.

## Project Structure

**Rust Crates:**

- `tako-core/` - Shared protocol types (Command, Response enums)
- `tako-runtime/` - Runtime plugins, download engine, caching
- `tako-server/` - Remote server runtime (proxy, instances, TLS, sockets)
- `tako/` - CLI tool (all commands)

**Registry:**

- `presets/` - Preset definitions (e.g. tanstack-start)

**SDK (current implementation):**

- `sdk/javascript/` - `tako.sh` JavaScript/TypeScript SDK package (npm)

**Website:**

- `website/` - Marketing/docs site (do not add automated tests for this component)

## Build & Test Commands

```bash
# Build all
cargo build

# Build release
cargo build --release

# Test all crates
cargo test

# Test specific crate
cargo test -p tako
cargo test -p tako-runtime
cargo test -p tako-server

# SDK (current JS/TS implementation)
cd sdk/javascript && bun install
bun run build && bun run typecheck
bun test
```

## Commit Messages

Use Conventional Commits for all commits:

- Format: `type(scope): short summary`
- Common types: `feat`, `fix`, `docs`, `refactor`, `test`, `chore`, `perf`, `ci`
- If scope is broad/mixed across the repo, use `chore(repo): ...`

## Code References

When referring to code, use format: `file_path:line_number`

Example: "Parse app name in `tako/src/app/name.rs:42`"

## Architecture Overview

### Data Flow

1. Developer: `tako deploy` → build locally → SFTP to server
2. Server: Unpack to `/opt/tako/{app}/releases/{version}/`
3. tako-server: Rolling update via unix socket protocol
4. Proxy: Pingora routes to healthy instances

### Key Components

**tako-core:** Minimal, protocol-only. Don't add features here.

**tako-server:**

- `proxy/` - Pingora HTTP/HTTPS proxy

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lilienblum/tako](https://github.com/lilienblum/tako) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
