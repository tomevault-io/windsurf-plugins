---
trigger: always_on
description: This file provides guidance to AI coding agents (Claude Code, pi, etc.) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents (Claude Code, pi, etc.) when working with code in this repository.

## Project Overview

BashTab is a Bash scripting framework providing intelligent autocompletion, argument parsing, IDE integration, and modern CLI tools. It stays 100% in Bash with no DSL or YAML conversion.

## Common Commands

### Testing

```bash
# Initialize test submodules (first time only)
git submodule update --init

source ./activate -t
```

Do **not** run the full test suite locally — it takes extremely long on a
normal machine (anything less than ~64-way parallelism is impractical).
Leave full-suite runs to CI.

Even a single `.bats` file can be very slow, so **handpick individual test
cases** with `--filter` instead of running a whole file:

```bash
# One test function
bats --filter 'test_bu_query_object_grep_regex_any_field' ./test/out_test.bats

# A few related tests by regex
bats --filter 'grep|query_object' ./test/out_test.bats
```

Only test what your change touches.

### Development Environment

```bash
source ./activate           # Standard activation
source ./activate -e        # Load examples environment
source ./activate -t        # Load test environment
```

### Build Single-File Distribution

```bash
source ./activate --__bu-inline ./inline.sh
```

## Architecture

### Core Modules (`/lib/core/`)

- **bu_core_base.sh** - Core utilities (filesystem, logging, string manipulation, arrays, caching)
- **bu_core_autocomplete.sh** - Autocompletion generation with lazy loading and script parsing
- **bu_core_cli.sh** - CLI command routing
- **bu_core_preinit.sh** - Pre-initialization for registering commands, key bindings, aliases
- **bu_core_tmux.sh** - Tmux orchestration and job management
- **bu_core_var.sh** - Global variable initialization

### Initialization Flow

1. `bu_entrypoint.sh` - Main entry point that orchestrates loading
2. Parses `BU_MODULE_LIST` (the sole module registry)
3. Loads static config (`config/bu_config_static.sh`)
4. Loads dynamic config (`config/bu_config_dynamic.sh`)
5. Sources all core modules
6. Runs pre-init callbacks, then main init, then post-entrypoint callbacks

### ⚠️ Custom `source` Function (read this before writing sourced files)

After activation, `source` is **not the bash builtin** — `bu_custom_source.sh`
replaces it with a function (`bu_def_source`) that adds `--__bu-once`,
autopushd, and inline-build support. One critical consequence:

**Any file sourced through it executes inside that function's scope.** A
top-level `declare` (without `-g`) in a sourced file therefore creates a
**function-local variable that vanishes when `source` returns**. The failure
is silent: the globals you declared are simply missing afterwards.

```bash
# ✗ BROKEN in sourced files — becomes a local of the source() function,
#   lost as soon as source returns
declare -r MY_CONST=$'\e'
declare -A MY_MAP=()
declare -i MY_COUNTER=0

# ✓ CORRECT — global declarations survive
declare -g -r MY_CONST=$'\e'
declare -A -g MY_MAP=()
declare -g -i MY_COUNTER=0

# ✓ ALSO FINE — plain assignments create/modify globals even inside a function
MY_SCALAR=value
MY_ARRAY=(a b c)
# (but associative arrays and readonly REQUIRE declare, so use -g for those)
```

**Symptoms of a missing `-g`:** `declare: VAR: not found` after sourcing;
escape sequences printed literally (`[1m` instead of bold text); unset
variables silently treated as `0`/`""` in arithmetic and expansions.

Escape hatches: `builtin source file.sh` bypasses the wrapper entirely;
`bu_ext_source` temporarily undefines it. Check at runtime with
`[[ "$BU_SOURCE_IS_CUSTOM" == true ]]`.

### Commands (`/commands/`)

Scripts named `bu-*.sh` that can be invoked via:
- `bu verb-noun ...`
- `bu-verb-noun.sh ...` (direct executable)

Command types: `execute` (new process), `source` (current shell), `function` (bash function)

## Creating New Commands

Use `bu new-command --dir commands --name my-command` to generate from template.

### Command Structure

All commands follow this pattern (see `lib/templates/script_template.sh`):

```bash
#!/usr/bin/env bash
function __bu_SCRIPT_NAME_main()
{
# 1. Setup: get script location, pushd to script dir
local -r invocation_dir=$PWD
local script_name script_dir
# ... path parsing ...
pushd "$script_dir" &>/dev/null

# 2. Source entrypoint (executable scripts only, skipped during autocomplete)
if [[ -z "$COMP_CWORD" ]]; then
    source "$BU_DIR"/bu_entrypoint.sh
fi

# 3. Initialize scope management
bu_exit_handler_setup
bu_scope_push_function
bu_scope_add_cleanup bu_popd_silent
bu_run_log_command "$@"

# 4. Declare local variables for options
local my_option=
local is_help=false
local error_msg=
local autocompletion=()
local shift_by=

# 5. Parse arguments in while loop
while (($#)); do
    bu_parse_multiselect $# "$1"
    case "$1" in
    -o|--option)# OPTION_HINT
        # Help text for this option (shown in autohelp)
        bu_parse_positional $# --hint "description"
        my_option=${!shift_by}
        ;;
    -f|--flag)# _FLAG
        # Help text for flag
        is_flag=true
        ;;
    -h|--help)# _FLAG
        is_help=true
        ;;
    *)
        bu_parse_error_enum "$1"
        break
        ;;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sunjc826/BashTab](https://github.com/sunjc826/BashTab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
