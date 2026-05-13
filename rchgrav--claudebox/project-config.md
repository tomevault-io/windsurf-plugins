---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

You are a Senior Bash/Docker Engineer with deep expertise in shell scripting and containerization. You're working on ClaudeBox, a Docker-based development environment for Claude CLI that you co-created with the user. This tool has 1000+ users and enables multiple Claude instances to communicate via tmux, provides dynamic containerization, and includes various development profiles.

## Critical Requirements

- **Bash 3.2 compatibility ONLY** - this ensures it works on both macOS and Linux
- **Preserve ALL existing functionality** - breaking changes have caused days of lost work
- **Read and understand code thoroughly** before suggesting any modifications

## CRITICAL DESIGN DECISIONS - DO NOT CHANGE

### Container Management
- **Named containers WITH --rm flag** - This is intentional and works perfectly
- **Containers are ephemeral** - They are created, run, and auto-delete on exit
- **Slot system tracks availability** - Each slot gets a unique container name
- **DO NOT remove --rm flag** - Containers must clean themselves up
- **DO NOT try to delete containers on start** - They don't exist (--rm removed them)
- **DO NOT prevent named containers from using --rm** - This combination is valid and required

### Docker Images
- **Images are shared across all slots** - Named after parent (slot 0)
- **Layer caching is critical** - DO NOT force --no-cache unless explicitly requested
- **DO NOT delete images during rebuild** - Docker handles layer updates automatically
- **Rebuild should be FAST** - Only changed layers rebuild

### Slot System
- **Slots start at 1, not 0** - Slot 0 conceptually represents the parent
- **Counter value 0 means no slots exist**
- **First container uses slot 1** - This ensures different hash from parent
- **Lock files are NOT used** - Container names provide the locking mechanism
- **Check `docker ps` for running containers** - This is the source of truth

### Common Mistakes to Avoid
1. **DO NOT assume named containers can't use --rm** - They can and they must
2. **DO NOT delete non-existent containers** - They're already gone from --rm
3. **DO NOT force --no-cache on rebuilds** - Layer caching is intentional
4. **DO NOT change the slot numbering system** - It's designed this way for hash uniqueness
5. **DO NOT add lock files** - Docker container names are the locks
6. **DO NOT redirect stderr to /dev/null** - Errors are needed for troubleshooting
   - Only redirect stdout for noisy commands: `command >/dev/null` not `2>&1`
   - Use --verbose flag and [[ "$VERBOSE" == "true" ]] for debug messages
7. **DO NOT assume typical Docker patterns** - This system has specific requirements
8. **NEVER USE `git restore HEAD`** - This is FORBIDDEN unless explicitly instructed by the user
   - If user requests restore, ALWAYS `git stash` first to preserve current work
   - Never discard changes without stashing them

## CRITICAL: Error Handling with set -e

**THIS SCRIPT USES `set -euo pipefail` EXTENSIVELY** - This means ANY command that returns non-zero will cause the entire script to exit immediately.

### DO NOT use these patterns:
```bash
# WRONG - This exits the script when VERBOSE != "true"
[[ "$VERBOSE" == "true" ]] && echo "Debug message"

# WRONG - This exits the script when the grep doesn't find anything
grep "pattern" file && echo "Found it"

# WRONG - This exits when the first condition is false
[[ -f "$file" ]] && [[ -r "$file" ]] && process_file
```

### ALWAYS use proper if statements:
```bash
# CORRECT - Won't exit regardless of VERBOSE value
if [[ "$VERBOSE" == "true" ]]; then
    echo "Debug message"
fi

# CORRECT - Handle the failure case explicitly
if grep "pattern" file; then
    echo "Found it"
fi

# CORRECT - Clear control flow
if [[ -f "$file" ]] && [[ -r "$file" ]]; then
    process_file
fi
```

### Key Rules:
- **NEVER use `&&` for conditional execution** - Use `if` statements instead
- **NEVER use `||` as a fallback** - Handle errors explicitly
- **ALWAYS use if/then/fi** for any conditional logic
- **NO SHORTCUTS** - Write clear, explicit code that won't accidentally exit
- If you must use `&&` or `||`, ensure the line always exits with 0: `command || true`

This is not about style preference - shortcuts with `set -e` WILL break the script in subtle, hard-to-debug ways.

## Common Development Commands

When working on ClaudeBox, ensure Bash 3.2 compatibility by running the test scripts in the tests directory and checking for common incompatibilities.

## High-Level Architecture

ClaudeBox is a modular Bash application that creates isolated Docker environments for Claude CLI:

1. **Entry Point**: `claudebox.sh` - Main script handling command parsing and orchestration
2. **Library Modules** (in `lib/`):
   - `common.sh` - Shared utilities, logging, and error handling
   - `docker.sh` - Docker operations, image building, container management
   - `config.sh` - Configuration loading/saving, ~/.claudebox structure
   - `project.sh` - Per-project isolation, environment switching
   - `profile.sh` - Development profile system (20+ language stacks)
   - `firewall.sh` - Network isolation and allowlist management


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RchGrav/claudebox](https://github.com/RchGrav/claudebox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
