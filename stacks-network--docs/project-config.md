---
trigger: always_on
description: This repository monitors merged PRs across all Stacks Network and Hiro Systems projects and ensures documentation stays in sync with code changes.
---

# Documentation Sync System

## Purpose
This repository monitors merged PRs across all Stacks Network and Hiro Systems projects and ensures documentation stays in sync with code changes.

## Monitored Repositories

### Stacks Network (21 repositories)
- **stacks-core** - Core Stacks blockchain implementation
- **clarity-wasm** - Clarity smart contract WASM implementation
- **docs** - This documentation repository
- **stacks-blockchain-docker** - Docker configurations for Stacks blockchain
- **clarunit** - Unit testing framework for Clarity
- **bitcoin** - Bitcoin integration components
- **c32check** - C32check encoding library
- **stacks** - Stacks utilities and tools
- **send-many-stx-cli** - CLI for batch STX transfers
- And 12 additional supporting repositories

### Hiro Systems (8 repositories)
- **clarinet** - Clarity development environment and testing framework
- **stacks.js** - JavaScript libraries for Stacks
- **connect** - Wallet connection library (formerly stacks-connect)
- **stacks.js-starters** - Starter templates for Stacks.js
- **vitest-environment-clarinet** - Vitest environment for Clarity testing
- And 3 additional supporting repositories

## Documentation Structure

### Primary Documentation Sections
- `/concepts/` - Conceptual documentation about Stacks, PoX, Clarity, sBTC
- `/guides-and-tutorials/` - Hands-on tutorials and how-to guides
- `/reference/` - API reference and technical specifications
- `README.md` - Quick start and overview

### Key Documentation Areas
- **Stacks blockchain concepts** - `/concepts/stacks-101/`
- **sBTC documentation** - `/concepts/sbtc/`
- **Clarity language** - `/concepts/clarity/`
- **Node operations** - `/guides-and-tutorials/nodes-and-miners/`
- **Signer operations** - `/guides-and-tutorials/running-a-signer/`
- **Stacking guides** - `/guides-and-tutorials/stack-stx/`
- **Developer quickstart** - `/guides-and-tutorials/hello-stacks-quickstart-tutorial.md`

## Review Process

### When Analyzing Merged PRs

1. **Fetch the full diff** - Don't just read the PR summary
   - Use `curl [diff_url]` or `gh api /repos/OWNER/REPO/pulls/NUMBER` to get complete changes
   - Look at actual code changes, not just commit messages

2. **Identify user impact** - Ask these questions:
   - Does this change what developers see or do?
   - Does this affect APIs, CLIs, or SDKs?
   - Does this change configuration or setup steps?
   - Does this introduce new features or capabilities?
   - Does this break existing functionality?

3. **Check documentation** - Search our docs for related content:
   - Use `grep -r "relevant_term" concepts/ guides-and-tutorials/ reference/`
   - Look for mentions of changed functions, APIs, or concepts
   - Check if tutorials reference affected code

4. **Be specific** - Point to exact files and sections that need updates
   - Don't say "update the Clarity docs"
   - Say "update `/concepts/clarity/language-functions.md` line 145 to reflect new `map-set` signature"

### What Requires Documentation

✅ **Always document:**
- New API endpoints or methods
- Changed function signatures (especially in public APIs)
- New Clarity functions or keywords
- New CLI commands or flags
- Modified configuration options
- Changed setup/installation steps
- Breaking changes (HIGH PRIORITY)
- New features or capabilities
- Changes to sBTC, stacking, or signing operations
- Updates to Stacks.js APIs
- Changes to Clarinet commands or configuration

❌ **Skip documentation for:**
- Internal refactoring with no external impact
- Test-only changes
- CI/CD pipeline updates
- Dependency updates (unless breaking or adding new capabilities)
- Bug fixes that restore intended behavior without changing usage
- Performance optimizations that don't change APIs
- Internal code comments or documentation in code

### Detection Scenarios

When analyzing PRs, classify documentation impact into three categories:

#### 1. UPDATE Existing Docs
**When:** Code changes make existing documentation incorrect or outdated.

**Detection signals:**
- Modified function signatures in public APIs
- Changed CLI flag behavior or defaults
- Updated configuration schema
- Renamed types, methods, or modules
- Changed setup/installation steps
- Breaking changes that alter existing functionality

**Example:**
```
PR changes `map-set` to require a third parameter `ttl`
→ Search docs for "map-set" 
→ Found in /concepts/clarity/language-functions.md
→ Issue: UPDATE the signature and examples to include ttl parameter
```

#### 2. CREATE New Docs
**When:** New functionality has zero documentation coverage.

**Detection signals:**
- New public functions, methods, or classes
- New API endpoints
- New CLI commands or subcommands
- New configuration options
- New features or capabilities
- Keywords in diff: "add", "new", "introduce", "implement"

**Example:**
```
PR adds new Clarity function `map-insert`
→ Search docs for "map-insert", "map_insert"
→ NOT FOUND in any documentation
→ Issue: CREATE new section in /concepts/clarity/language-functions.md with signature, usage, examples
```

#### 3. REMOVE or DEPRECATE Docs
**When:** Features are removed or deprecated but docs still reference them.

**Detection signals:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stacks-network/docs](https://github.com/stacks-network/docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
