---
trigger: always_on
description: **kobo.koplugin** is a KOReader plugin that extends Kobo device functionality with virtual library
---

# Copilot Instructions for kobo.koplugin

## Project Overview

**kobo.koplugin** is a KOReader plugin that extends Kobo device functionality with virtual library
management and reading state synchronization.

- **Language:** Lua (primary), Markdown (docs), Shell (build scripts)
- **Testing Framework:** Busted
- **Current Version:** 0.1.0
- **Plugin ID:** kobo.koplugin

## Project Structure

```
/
├── _meta.lua                              # Plugin metadata (version, description, author)
├── main.lua                               # Plugin entry point
├── src/                                   # Core plugin modules (extensions, managers, handlers)
│   ├── bookinfomanager_ext.lua
│   ├── docsettings_ext.lua
│   ├── document_ext.lua
│   ├── filechooser_ext.lua
│   ├── filesystem_ext.lua
│   ├── metadata_parser.lua
│   ├── readerpagemap_ext.lua
│   ├── readerui_ext.lua
│   ├── reading_state_sync.lua
│   ├── session_flags.lua
│   ├── showreader_ext.lua
│   └── lib/                               # Shared libraries and utilities
│       ├── kobo_state_reader.lua
│       ├── kobo_state_writer.lua
│       ├── status_converter.lua
│       └── sync_decision_maker.lua
├── patches/                               # KoReader patches
│   ├── 2-filemanager-menu-separator.lua
│   ├── 2-kobo-virtual-library-startup.lua
│   └── 2-readerrolling-position-fix.lua
├── spec/                                  # Busted test files
│   ├── helper.lua
│   └── *_spec.lua
├── docs/                                  # Markdown documentation
├── book/                                  # mdBook generated HTML (output directory)
└── package.sh                             # Build script for creating distributable ZIP
```

## Language-Specific Guidelines

This project uses multiple languages with specific coding standards:

- **Lua:** See `.github/instructions/lua.md` for detailed Lua coding guidelines
- **Shell Scripts:** See `.github/instructions/shell.md` for shell script best practices
- **Markdown:** See `.github/instructions/markdown.md` for documentation formatting

## General Coding Principles

### 1. Code Quality

- Write clear, maintainable code
- Prefer simplicity over cleverness
- Document complex logic with comments
- Keep functions focused and single-purpose
- Use descriptive variable and function names

### 2. Testing Requirements

- All new features must include tests
- Tests must be placed in `spec/` directory with `_spec.lua` suffix
- Use Busted testing framework DSL
- Aim for minimum 70% code coverage
- Mock external dependencies, not code under test

### 3. Error Handling

- Always validate inputs to functions
- Return early on error conditions
- Provide meaningful error messages
- Handle edge cases explicitly

### 4. Performance Considerations

- Avoid unnecessary file I/O operations
- Cache computed values when appropriate
- Be mindful of memory usage on embedded devices
- Profile code for performance-critical sections

## CI/CD Workflows

This project uses GitHub Actions for continuous integration:

- **Linting:** Runs luacheck, stylua, prettier, and shellcheck
- **Testing:** Runs busted tests with coverage reporting
- **Build:** Creates distributable artifacts (kobo.koplugin.zip, kobo-patches.zip)
- **Documentation:** Builds mdBook documentation and deploys to GitHub Pages
- **Release:** Automated releases using release-please

All workflows use intelligent change detection to run only when relevant files are modified.

## Development Workflow

### Local Development

```bash
# Format Lua code
stylua --sort-requires --indent-type Spaces --indent-width 4 *.lua src/ spec/

# Lint Lua code
luacheck *.lua src/ spec/

# Check shell scripts
shellcheck *.sh

# Format markdown
prettier --write "docs/**/*.md" "*.md"

# Run tests
busted spec/
```

### Creating Releases

This project uses conventional commits for automated releases:

- `feat: description` → Minor version bump
- `fix: description` → Patch version bump
- `docs: description` → No version bump
- `chore: description` → No version bump
- `BREAKING CHANGE:` in body → Major version bump

When you push to main with conventional commits, release-please automatically creates a PR with
version bump and changelog.

## Common Patterns

### Module Structure

```lua
local Module = {}

-- Private function (underscore prefix)
local function _internalHelper()
    -- implementation
end

-- Public function
function Module.publicFunction()
    return _internalHelper()
end

return Module
```

### Early Return Pattern

```lua
-- GOOD: Use early returns
function process(data)
    if not data then
        return nil
    end

    if not data.isValid then
        return nil
    end

    return transformData(data)
end

-- BAD: Avoid nested if-else
function process(data)
    if data then
        if data.isValid then
            return transformData(data)
        else
            return nil
        end
    else
        return nil
    end
end
```

### Test Structure

```lua
describe("Module", function()
    local instance

    before_each(function()
        instance = Module.new()
    end)

    describe("method", function()
        it("should handle valid input", function()
            local result = instance:method("valid")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OGKevin/kobo.koplugin](https://github.com/OGKevin/kobo.koplugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
