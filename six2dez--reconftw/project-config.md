---
trigger: always_on
description: **Analysis Date:** 2026-05-13
---

# Coding Conventions

**Analysis Date:** 2026-05-13

## Shell Settings

**Entry point** (`reconftw.sh` top of file):
```bash
set -o pipefail
set -E
set +e          # fail-soft: individual commands may fail
IFS=$'\n\t'
```

**Libraries** (`lib/*.sh`) use `set -o pipefail` only — not `set -e`, because most functions return non-zero deliberately.

**Modules** (`modules/*.sh`) use NO `set` directives; they inherit from the sourcing shell.

## Source Guard Pattern

Every library file (`lib/`) begins with:
```bash
[[ -n "$_FOO_LOADED" ]] && return 0
declare -r _FOO_LOADED=1
```

Examples:
- `lib/common.sh`: `[[ -n "$_COMMON_SH_LOADED" ]] && return 0`
- `lib/parallel.sh`: `[[ -n "$_PARALLEL_SH_LOADED" ]] && return 0`
- `lib/ui.sh`: `[[ -n "${_UI_SH_LOADED:-}" ]] && return 0`

Tests that need to re-source a library must first unset the guard: `_COMMON_SH_LOADED=""`.

Modules (`modules/*.sh`) use a different guard — they check that `SCRIPTPATH` is set:
```bash
[[ -z "${SCRIPTPATH:-}" ]] && { echo "Error: This module must be sourced by reconftw.sh" >&2; exit 1; }
```

## Function Naming

**Declaration syntax:** Always use `function name() { ... }` form, never the POSIX `name() {` form.

```bash
function start_func() { ... }
function end_func() { ... }
function validate_domain() { ... }
```

Exception: `lib/ui.sh` and `lib/common.sh` helper functions omit `function` keyword to stay consistent with the POSIX-style helpers (`ui_init()`, `ensure_dirs()`, etc.). Both styles appear; prefer `function name()` for any new public API.

**Naming conventions:**
- Public module functions: `snake_case` prefixed with module context (`sub_passive`, `sub_crt`, `geo_info`)
- Private helpers: `_snake_case` prefix (`_print_status`, `_print_error`, `_print_module_start`, `_parallel_emit_job_output`)
- UI layer: `ui_` prefix (`ui_init`, `ui_header`, `ui_summary`, `ui_batch_end`)
- Lifecycle wrappers: `start_func` / `end_func` (call these at top/bottom of every recon function)
- Validation functions: `validate_*` / `sanitize_*` (defined in `lib/validation.sh` and `modules/utils.sh`)

## Variable Naming

**Globals (UPPER_SNAKE_CASE):**
- Config flags: `SUBPASSIVE`, `SUBCRT`, `PARALLEL_MODE`, `OUTPUT_VERBOSITY`
- Runtime state: `LOGFILE`, `SCRIPTPATH`, `DIFF`, `DRY_RUN`, `AXIOM`
- Error codes: `E_SUCCESS=0`, `E_INVALID_DOMAIN=20`, `E_INVALID_IP=21` (readonly, defined in `lib/validation.sh`)

**Locals (lowercase):**
```bash
local file="$1"
local domain="$2"
local count=0
```
Always declare locals with `local`. Always quote variable expansions.

**Color variables** (`bred`, `bgreen`, `bblue`, `byellow`, `yellow`, `reset`, `cyan`) are defined in `reconftw.cfg` and overridden to empty strings in no-color mode by `lib/ui.sh`.

## CLI Flag Pattern

CLI flags are captured as `CLI_*` variables during argument parsing, then re-applied AFTER `reconftw.cfg` is sourced (lines ~513-575 of `reconftw.sh`). This ensures config file defaults do not clobber command-line flags.

```bash
# During parsing (before config load):
'--quiet')    CLI_OUTPUT_VERBOSITY=0 ;;
'--verbose')  CLI_OUTPUT_VERBOSITY=2 ;;
'--parallel') CLI_PARALLEL_MODE=true ;;

# After config load (reconftw.sh ~line 513):
if [[ -n "${CLI_OUTPUT_VERBOSITY:-}" ]]; then
    OUTPUT_VERBOSITY="${CLI_OUTPUT_VERBOSITY}"
fi
if [[ -n "${CLI_PARALLEL_MODE:-}" ]]; then
    PARALLEL_MODE="${CLI_PARALLEL_MODE}"
fi
```

When adding a new CLI flag: add `CLI_NEWFLAG` during the `getopt` parsing loop, then add a re-apply block after config sourcing.

## Output / UI Conventions

**Status badges:** `OK`, `WARN`, `FAIL`, `SKIP`, `CACHE`, `INFO`, `RUN`

**Status line format** (dot-fill, right-aligned timing):
```
OK    sub_passive                    12s
WARN  sub_crt                        3s  (no results)
SKIP  sub_brute                      0s
FAIL  sub_dns                        7s  (see debug.log)
```
Produced by `_print_status STATE "func_name" "duration"` in `lib/common.sh`.

**Section headers** use thin `───` rules:
```bash
_print_section "OSINT"    # calls _print_module_start()
```
Output: `── OSINT ────────────────────────────────────────────────────────────────`

**Verbosity gating:**
- `OUTPUT_VERBOSITY=0` (quiet): only errors/FAIL printed
- `OUTPUT_VERBOSITY=1` (normal, default): OK/WARN/FAIL/SKIP status lines
- `OUTPUT_VERBOSITY=2` (verbose): all of the above + INFO messages + start_func messages

The `notification()` function in `modules/core.sh` gates by level:
```bash
# error → always visible
# warn  → OUTPUT_VERBOSITY >= 1
# info|good → OUTPUT_VERBOSITY >= 2
```

## Function Lifecycle (start_func / end_func)

Every recon function that runs a tool wraps its body with the lifecycle pair:

```bash
function sub_passive() {
    if { [[ ! -f "$called_fn_dir/.${FUNCNAME[0]}" ]] || [[ $DIFF == true ]]; } && [[ $SUBPASSIVE == true ]]; then
        start_func "${FUNCNAME[0]}" "Passive subdomain enum"
        # ... tool invocations ...
        end_func "Passive subdomain enum" "${FUNCNAME[0]}"
    else
        skip_notification "processed"   # or "disabled"
    fi
}
```

- `start_func name desc` — logs to LOGFILE, sets per-function start timestamp, emits INFO at verbosity >= 2
- `end_func message name [status]` — touches checkpoint file, calculates elapsed time, calls `_print_status`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [six2dez/reconftw](https://github.com/six2dez/reconftw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
