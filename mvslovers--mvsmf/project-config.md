---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.


## Project Overview

mvsMF is a z/OSMF REST API implementation for MVS 3.8j, built as a CGI module for Mike Rayborn's HTTPD server. It enables modern clients (Zowe Explorer, Zowe CLI, JetBrains IDEs) to interact with classic MVS systems via standard z/OSMF REST endpoints.

## C Standard Override

**This project uses `-std=gnu99`**, overriding the root CLAUDE.md's strict C89 rule.

Implications:
- `//` line comments are allowed
- Mixed declarations and statements are allowed
- `snprintf`, `stdbool.h`, designated initializers are available
- VLAs are still forbidden (stack constraints)
- All variable declarations should still prefer top-of-block for readability

Cross-compiled for MVS/370 using the `c2asm370` compiler (GCC 3.2.3 fork). All other platform constraints from the root CLAUDE.md still apply (24-bit addressing, EBCDIC, no POSIX, memory efficiency, etc.).

## Development Workflow

1. Every bug fix or feature requires a **GitHub Issue**. If none exists, create one first.
2. **Plan and analyze first** using the Opus 4.6 model. Implementation follows using Sonnet 4.6.
3. Develop each fix/feature on a **feature branch**.
4. When done, merge via **Pull Request** and close the issue with a short comment.
5. **Never reference AI or Claude** in commit messages, comments, PR descriptions, or anywhere else in the project.

## Custom Commands

### /fix-issue \<number\>

Autonomous workflow for resolving a GitHub issue end-to-end:

1. **Read the issue** — `gh issue view <number> --repo mvslovers/mvsmf`
2. **Check for spec** — If the issue has label `uss-phase1`, read `doc/uss-spec.md` first
3. **Create a feature branch** — `git checkout -b issue-<number>-<short-description>`
4. **Analyze** — Identify affected files, understand the existing patterns in nearby code
5. **Implement** — Write code following the conventions in this CLAUDE.md
6. **Verify syntax** — Run `make compiledb` and check clangd diagnostics (no errors)
7. **Update tests** — Add/update tests in `tests/` matching the change
8. **Update docs** — If touching an endpoint handler, update `doc/endpoints/`
9. **Commit** — Descriptive message, no AI references. Reference the issue: `Fixes #<number>`
10. **Push and create PR** — `gh pr create --title "..." --body "Fixes #<number>"`
11. **Summary** — Report what was done, what to verify on the live MVS system

If any step fails, stop and report the issue rather than guessing.

### /sync-docs

Synchronize endpoint documentation with current implementation:

1. Scan all `add_route()` calls in `src/mvsmf.c`
2. For each route, check if matching doc exists in `doc/endpoints/`
3. Report missing or outdated documentation
4. Optionally generate stubs for undocumented endpoints

## Specifications

### USS/UFS Feature Pack

The authoritative specification for all USS-related work is `doc/uss-spec.md`. **Read it before working on any issue labeled `uss-phase1`.** It contains:

- Gap analysis: z/OSMF USS API vs. libufs capabilities
- Architecture decisions (router wildcard, session lifecycle, MBT integration)
- UFSD error code → HTTP status mapping (complete table)
- Encoding rules and I/O patterns
- Implementation plan with task dependencies

## Build System (mbt)

mvsMF uses [mbt](https://github.com/mvslovers/mbt) as its build tool (Git submodule in `mbt/`). Clone with `--recursive` or run `git submodule update --init`.

### Build Commands

```bash
make doctor        # verify environment (MVS connectivity, tools)
make bootstrap     # resolve dependencies, allocate MVS datasets
make build         # cross-compile C → ASM, assemble on MVS, NCAL link
make link          # final linkedit into load module
make install       # copy load module to install dataset
make compiledb     # generate compile_commands.json for clangd
make clean         # remove .s, .o files, build stamps
make distclean     # deep clean (also removes contrib/ and .mbt/)
```

The build chain is: C source → assembly via c2asm370 → upload to SOURCE PDS → IFOX00 assemble + IEWL NCAL link on MVS. Final linkedit produces the MVSMF load module.

### Dependencies (from project.toml)

```toml
[dependencies]
"mvslovers/crent370" = ">=1.0.6"
"mvslovers/httpd" = "=3.3.1-dev"
```

`make bootstrap` resolves these from GitHub Releases, downloads headers into `contrib/`, and provisions NCALIB/MACLIB datasets on MVS.

### Configuration

Local settings go in `.env` (gitignored). See `.env.example` for the template. Key variables: `MBT_MVS_HOST`, `MBT_MVS_PORT`, `MBT_MVS_USER`, `MBT_MVS_PASS`, `MBT_MVS_HLQ`.

clangd provides IDE diagnostics (configured in `.clangd`).

## Architecture

### Request Processing Pipeline

```
HTTP Request → cgxstart.c (CGI init) → mvsmf.c (router setup)
  → router.c (URL decode, method parse, route match, path var extraction)
    → Middleware chain (authmw.c, logmw.c)
      → API handler (dsapi.c, jobsapi.c, infoapi.c)
        → JSON response (json.c)
```

### Key Source Files

- **mvsmf.c**: Entry point. Registers all routes and middleware, initializes the router and HTTPD session.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mvslovers/mvsmf](https://github.com/mvslovers/mvsmf) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
