---
trigger: always_on
description: These rules apply to every task in this project unless explicitly overridden.
---

# AGENTS.md — 6-rule

These rules apply to every task in this project unless explicitly overridden.
Bias: caution over speed on non-trivial work. Use judgment on trivial tasks.
Each rule ends with a ❌/✅ pair — match the pattern, not the slogan.

## Rule 0 — Modern CLI only (enforced by PreToolUse hook)
Mappings: `find`→`fd`, `grep`→`rg`, `cat`→`bat` (or Read), `ls`→`eza`, `diff`→`delta`, JSON parsing → `jq` (never `python -c "import json"`).
The hook `.claude/hooks/legacy-cli-pretool.sh` will **deny** any Bash call whose first segment-token is a legacy tool and tell you the replacement — reissue with the modern equivalent, don't retry the same command. `git grep` / `git diff` are fine.
- ❌ `find . -name "*.go" | xargs grep TODO`
- ✅ `fd -e go -x rg TODO`

## Rule 1 — Think, ask, surface conflicts
State assumptions before coding. If two interpretations are both plausible, present them and ask — don't pick silently. If two patterns in the codebase contradict, pick one (more recent / more tested), say why, flag the other for cleanup; never blend them. Use the model only for judgment work (classification, drafting, summarization, extraction); for routing / retries / deterministic transforms, write code — don't ask the model.
- ❌ Picking interpretation A and producing 200 lines of code for it; or writing a retry loop by prompting the model.
- ✅ "I see two readings: A or B. Going with A because X — confirm if you meant B." / Retries live in a `for` loop with explicit backoff.

## Rule 2 — Minimal, surgical, conformant changes
Smallest diff that solves the stated problem. No speculative features, no abstractions for single-use code, no "improvements" to adjacent code / comments / formatting. Match the codebase's existing style even if you disagree — if you genuinely think a convention is harmful, surface it; don't fork silently. Senior-engineer test: would they call this overcomplicated or out-of-scope? If yes, simplify.
- ❌ Bug fix that also renames variables in nearby functions "while we're here", or introduces a `Strategy` interface for one caller.
- ✅ Smallest diff that fixes the bug; new abstraction only when ≥2 real call sites exist.

## Rule 3 — Read before you write
Before adding code: read the relevant exports, immediate callers, shared utilities in `libs/`. "Looks orthogonal" is dangerous — structure usually exists for a reason. Confirm a new helper has a real call site before committing it; `unusedfunc` / `unusedparams` are blocking findings, not advisories.
- ❌ Writing `parseDate()` helper and trusting nothing similar exists.
- ✅ `rg -i 'parseDate|ParseDate' libs/ tools/` first, then either reuse or add.

## Rule 4 — Goal-driven loop
Define success criteria up front, then iterate until verified. Don't follow a fixed step list — strong criteria let you self-correct. For feature work, "compiles" and "`go vet` clean" are not "feature works" — exercise the actual behavior and cite the evidence (command run, output observed).
- ❌ "Done — `go build ./...` passes."
- ✅ "Criteria: import R41 into `dewu-burgeon-sales-daily` for week N. Ran `./bin/...`; row count matches source xlsx (1,234); spot-checked 3 rows against `usage.md` query."

## Rule 5 — Report honestly: checkpoint, fail loud, tests verify intent
**Checkpoint** after each significant step — what's done, what's verified, what's left; if you lose track, stop and restate. **Fail loud** — "completed" is wrong if anything was skipped silently; "tests pass" is wrong if any were skipped or marked `t.Skip`; surface uncertainty, don't hide it. **Tests encode intent**, not just behavior — a test that can't fail when the business rule changes is broken; assert *why* the value matters (the rule), not just *what* it is right now.
- ❌ "All 3 subtasks done!" when subtask 2 silently fell through to a default, or a test that just re-encodes the current return value with no link to the business rule.
- ✅ "2 of 3 done. Subtask 2 hit Y — need your call on Z before continuing." / `assert sale_price == cost * (1 + REQUIRED_MARGIN)` instead of `assert sale_price == 13.75`.

# Build & run

Requires Zig `0.15.2`, the Windows SDK shader compilers (`fxc.exe` for SM5/DXBC and `dxc.exe` with `dxil.dll` beside it for signed SM6/DXIL), and the LunarG Vulkan SDK (`dxc.exe` plus `spirv-val.exe` for SPIR-V). The two DXC installs are not interchangeable: the Vulkan SDK one emits SPIR-V but ships no `dxil.dll`, and D3D12 rejects unsigned DXIL. Windows-only. Dependencies are declared in `build.zig.zon`; `win32` is marked `lazy = true`. The build discovers the newest installed SDK versions from their standard locations or the `WindowsSdkVerBinPath` / `VULKAN_SDK` environment variables; use `-Dfxc-path=<path>`, `-Ddxil-dxc-path=<path>`, or `-Ddxc-path=<path>` to override discovery.

- `zig build` — build the `Mostty` executable into `zig-out/bin/`.
- `zig build run -- [args]` — build and run; everything after `--` is forwarded as cmdline args (see `src/Cmdline.zig`: `--ttf <path>`, `--font-size <float>`).
- `zig build test` — run the unit test step (compiles the same root file as the exe; there are very few tests today).
- `zig build -Doptimize=ReleaseSmall` — what the README's "less than 2 MB" Windows binary refers to.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [missdeer/mostty](https://github.com/missdeer/mostty) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
