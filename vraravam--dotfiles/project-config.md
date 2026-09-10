---
trigger: always_on
description: This dotfiles repository uses a comprehensive instruction system in `.ai/`. This file highlights the non-obvious patterns agents commonly miss.
---

# Agent Quick Reference

This dotfiles repository uses a comprehensive instruction system in `.ai/`. This file highlights the non-obvious patterns agents commonly miss.

## Instruction Architecture

**Primary source of truth**: `.ai/instructions.md` → domain-specific files in `.ai/domains/`

Tool-specific configs (`.cursorrules`, `.windsurfrules`, `.github/copilot-instructions.md`) are minimal redirects. Load `.ai/instructions.md` first.

Key instruction files (apply to code in `${DOTFILES_DIR}`, `${PERSONAL_BIN_DIR}`, and `${PERSONAL_CONFIGS_DIR}`):
- `shell-scripting.md` — Shell patterns, utilities, startup optimization
- `ruby-scripting.md` — Ruby patterns, dual-mode scripts, GitProcessor
- `zsh-startup.md` — Performance optimization, caching, plugin loading
- `fresh-install.md` — Bootstrap idempotency, vanilla OS mode
- `git-config.md` — Git aliases, hooks, per-repo customization
- `edit-checklist.md` — Post-edit verification workflow (syntax, format, whitespace)

**CRITICAL**: Any refactoring, formatting, or pattern fixes identified in one location MUST be applied to **all matching files** across all three directories. For example:
- If you fix a `&&` pattern in `${DOTFILES_DIR}/scripts/file.sh`, scan and fix the same pattern in `${PERSONAL_BIN_DIR}/*.sh`
- If you update UTF-8 file reading in a Ruby script, search for all instances of `File.readlines` / `.each_line` across all three locations
- Use grep/glob to find all matching files before starting edits: `grep -r "pattern" ${DOTFILES_DIR} ${PERSONAL_BIN_DIR} ${PERSONAL_CONFIGS_DIR}`

## Git State Management — NEVER Modify Without Permission

**DEFAULT WORKFLOW: The user reviews and stages changes manually.**

After making edits:
1. ✅ Make the requested edits to files
2. ✅ Show `git status` or `git diff` to display what changed
3. ❌ **STOP** - Do NOT stage, commit, or modify git state
4. ✅ Let the user review changes and stage manually

**Prohibited without explicit permission:**
- `git add` / `git add -A` / `git add .` — staging files
- `git commit` / `git commit -a` / `git commit --amend` — creating/modifying commits
- `git reset` / `git restore --staged` — unstaging files
- `git checkout` / `git switch` — changing branches
- `git push` / `git push --force` — pushing to remote
- `git rebase` / `git merge` — modifying history
- `git stash` / `git rm` — any other state-modifying command

**If the user explicitly grants permission for ONE operation:**
- Permission is ONE-TIME ONLY for that specific instruction
- NOT a blanket approval for the rest of the session
- NOT carried over to future sessions
- Each git state change requires fresh explicit permission

**Why this matters:**
- Users manually review all changes before staging
- Users may stage specific hunks or files for different commits
- Automatically staging destroys the user's deliberate staging intent
- The staging area reflects the user's workflow decisions

See: `.ai/instructions.md` § Git State Management Rules

## Critical Commands (Run After Every Edit)

**Shell scripts**:
```zsh
zsh -n file.sh                    # Syntax check
shfmt -w file.sh                  # Format
rm -f file.sh.zwc                 # Delete bytecode cache
chmod +x file.sh                  # Ensure executable
```

**Ruby scripts**:
```zsh
/usr/bin/ruby -c file.rb          # Syntax (Ruby 2.6 compat required)
rufo file.rb                      # Format
```

**All files** (except `.md`):
```zsh
# Verify whitespace rules (see .ai/instructions.md § Whitespace Rules)
tail -c 1 file | od -An -tx1 | grep -q '0a' || echo "Missing final newline"
tail -n 1 file | grep -q '^$' && echo "Has trailing blank lines"
grep -q '[[:space:]]$' file && echo "Has trailing whitespace"
```

## Decision-Making Priority Order

When choices conflict, this order wins:

1. **Startup speed** (for zsh hot path)
2. **Maintainability** (readability, DRY, clear intent)
3. **POSIX compatibility** (when scripts run in bash/direnv)
4. **Zsh built-ins** (when they don't conflict with #1-3)

Document tradeoffs in comments when priorities conflict.

## Common Agent Mistakes

### 1. `&&` Under `set -e` Triggers ERR Traps

**Problem**: Standalone `A && B` where A returning false is *expected* propagates exit code 1, triggering ERR trap or aborting the script.

```zsh
# BAD -- file not existing is normal, but fires ERR trap
is_file "${optional}" && process_file

# Good -- explicit if/then never propagates predicate exit code
if is_file "${optional}"; then process_file; fi
```

**Safe exception**: `A && B || C` where C returns 0 (overall expression = 0, no trap).

See: `shell-scripting.md` § `&&` as Conditional

### 2. Dual-Mode Ruby Scripts (Module + CLI)

**ALL standalone Ruby scripts MUST follow this pattern** to enable both CLI usage and direct module calls (no subprocess overhead):

```ruby
module MyScript
  extend self
  def run(param:)
    # ... logic ...
    true  # Return boolean, NEVER call exit()
  end
end

if __FILE__ == $PROGRAM_NAME
  include Logging  # Only in CLI block
  # ... option parsing ...
  success = MyScript.run(param: value)
  exit(success ? 0 : 1)
end
```

**When calling from another Ruby script**:
```ruby
require_relative 'my-script'

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vraravam/dotfiles](https://github.com/vraravam/dotfiles) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
