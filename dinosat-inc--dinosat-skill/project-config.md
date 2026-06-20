---
trigger: always_on
description: Formal verification of Solidity smart contracts via Halmos symbolic execution and Z3 SMT solver. Generates proof tests, runs Halmos per-contract, produces a verification report. Triggers on 'formally verify', 'halmos', 'symbolic tests', 'prove this contract', 'formal verification', 'dinosat'.
---


# DinoSAT — Formal Verification for Solidity

Orchestrator for a formal verification pipeline. Reads Solidity contracts, detects Z3-hostile patterns, generates symbolic test suites, runs Halmos/Z3, and produces a verification report.

`$SKILL_DIR` = the directory containing this SKILL.md file. Resolve it from the path you loaded this skill from.

## Mode Selection

- **Default** (no arguments): verify all `.sol` files in `src/` (exclude `interfaces/`, `lib/`, `mocks/`, `test/`).
- **`$filename ...`**: verify the specified file(s) only.
- **`--report-only`**: skip test generation, run Halmos on existing `test/dinosat/` tests and produce report.

---

Before doing anything else, print this exactly:

Before doing anything else, output this text directly (not via a tool call — just print it as your response text):

```
██████╗ ██╗███╗   ██╗ ██████╗ ███████╗ █████╗ ████████╗
██╔══██╗██║████╗  ██║██╔═══██╗██╔════╝██╔══██╗╚══██╔══╝
██║  ██║██║██╔██╗ ██║██║   ██║███████╗███████║   ██║
██║  ██║██║██║╚██╗██║██║   ██║╚════██║██╔══██║   ██║
██████╔╝██║██║ ╚████║╚██████╔╝███████║██║  ██║   ██║
╚═════╝ ╚═╝╚═╝  ╚═══╝ ╚═════╝ ╚══════╝╚═╝  ╚═╝   ╚═╝

Formal Verification Engine | https://dinosat.com
```

---

## Orchestration

### Phase 1 — Environment & Discovery

Make these parallel tool calls in one message:

a. Bash `forge --version` — verify Foundry installed
b. Bash `halmos --version` — verify Halmos installed (require v0.1.13+)
c. Read `foundry.toml` — check for `[profile.halmos]` section
d. Bash `find src/ -name "*.sol" -not -path "*/interfaces/*" -not -path "*/lib/*" -not -path "*/mocks/*" -not -path "*/test/*"` — enumerate in-scope contracts
e. Glob `**/references/z3-tractability.md` — resolve `$SKILL_DIR/references/` path
f. Bash `ls test/dinosat/ 2>/dev/null` — check if halmos tests already exist

**Gate 1 — Prerequisites.** If `halmos --version` fails, print: `Halmos is required. Install: pip install halmos`. Stop.

**Foundry profile setup.** Check `foundry.toml` for an existing `[profile.halmos]` section:

1. **No `[profile.halmos]` exists** — append the block below using profile name `halmos`. Set `$HALMOS_PROFILE = halmos`.
2. **`[profile.halmos]` exists and its `test` field is `test/dinosat`** — use it as-is. Set `$HALMOS_PROFILE = halmos`.
3. **`[profile.halmos]` exists but its `test` field points elsewhere** (e.g., `test/halmos`) — do NOT modify it. Instead, append a new `[profile.dinosat]` block. Set `$HALMOS_PROFILE = dinosat`. Inform the user: `Existing [profile.halmos] points to <path> — created [profile.dinosat] to avoid conflict.`

Block to append (use `$HALMOS_PROFILE` as the profile name):

```toml
[profile.$HALMOS_PROFILE]
src = "src"
test = "test/dinosat"
out = "out-halmos"
libs = ["lib"]
optimizer = false
evm_version = "shanghai"
ast = true
```

All subsequent commands that reference `FOUNDRY_PROFILE=halmos` MUST use `FOUNDRY_PROFILE=$HALMOS_PROFILE` instead. This includes Phase 3d (compilation), Phase 4a (Halmos execution), and any test file doc comments.

Create `test/dinosat/` directory if it does not exist.

### Phase 2 — Source Analysis & Pattern Detection

**Phase 2a — Parallel reads.** Make these parallel tool calls in one message:

a. Read ALL in-scope `.sol` files from Phase 1d (batch into multiple Read calls)
b. Read `$SKILL_DIR/references/z3-tractability.md`
c. Read `$SKILL_DIR/references/proof-techniques.md`
d. Read `$SKILL_DIR/references/property-catalog.md`
e. Read `$SKILL_DIR/references/test-templates.md`

**Phase 2b — Pattern scanner.** After all reads complete, make these parallel Grep tool calls in one message to detect Z3-hostile patterns across ALL source files. Use the Grep tool (NOT bash grep) with ripgrep syntax:

1. Detect Math.mulDiv calls — pattern: `Math\.mulDiv|mulDiv\(`, path: `src/`, glob: `*.sol`
2. Detect Math.sqrt calls — pattern: `Math\.sqrt|sqrt\(`, path: `src/`, glob: `*.sol`
3. Detect assembly blocks — pattern: `assembly\s*\{`, path: `src/`, glob: `*.sol`
4. Detect symbolic exponentiation patterns — pattern: `\*\*\s*[a-z]`, path: `src/`, glob: `*.sol`
5. Count if/else branch depth per function — pattern: `\} else if|\} else \{`, path: `src/`, glob: `*.sol`

All Grep calls should use `output_mode: "content"` with line numbers enabled (`-n: true`).

**Phase 2c — Protocol-type detection.** Classify the protocol archetype(s) using `$SKILL_DIR/references/protocol-types.md`. Scan for detection signals (e.g., `swap` + `addLiquidity` → AMM, `borrow` + `liquidate` → Lending). A protocol may be hybrid — apply properties from ALL matching archetypes.

Print to user: `Protocol type: <type> [+ <type2> if hybrid]`

**Phase 2c.1 — Threat model.** Using the detected protocol type(s), source code analysis, and `$SKILL_DIR/references/protocol-types.md`, build a threat model for the report. This goes into the "Threat Model" section of `DINOSAT_REPORT.md`. Identify:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dinosat-inc/dinosat-skill](https://github.com/dinosat-inc/dinosat-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
