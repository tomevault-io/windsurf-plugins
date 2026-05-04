---
trigger: always_on
description: - Do what has been asked; nothing more, nothing less
---

# Claude Code Configuration - AQE & Claude Flow V3

## Behavioral Rules (Always Enforced)

- Do what has been asked; nothing more, nothing less
- NEVER create files unless they're absolutely necessary for achieving your goal
- ALWAYS prefer editing an existing file to creating a new one
- NEVER proactively create documentation files (*.md) or README files unless explicitly requested
- NEVER save working files, text/mds, or tests to the root folder
- Never continuously check status after spawning a swarm — wait for results
- ALWAYS read a file before editing it
- NEVER commit secrets, credentials, or .env files

## Data Protection (ABSOLUTE — Data Loss = Unrecoverable)

- NEVER overwrite, replace, recreate, or `rm` any `.db` file without explicit user confirmation
- NEVER run DROP TABLE, DELETE FROM, or TRUNCATE on `.agentic-qe/memory.db` or any learning database
- NEVER consolidate or migrate databases without verifying row counts before AND after
- NEVER claim a sync or migration succeeded without querying the destination to verify actual row counts
- ALWAYS backup before any database operation: `cp file.db file.db.bak-$(date +%s)`
- ALWAYS verify integrity after operations: `sqlite3 file.db "PRAGMA integrity_check; SELECT COUNT(*) FROM qe_patterns;"`
- ALWAYS remove stale WAL/SHM files when restoring: `rm -f file.db-wal file.db-shm`
- When fixing sync/migration code, test against a COPY of the database, never the original
- The `.agentic-qe/memory.db` contains 1K+ irreplaceable learning records — treat it like production data

## File Organization

- NEVER save to root folder — use the directories below
- Use `/src` for source code files
- Use `/tests` for test files
- Use `/docs` for documentation and markdown files
- Use `/config` for configuration files
- Use `/scripts` for utility scripts
- Use `/examples` for example code

## Project Architecture

- Follow Domain-Driven Design with bounded contexts
- Keep files under 500 lines
- Use typed interfaces for all public APIs
- Prefer TDD London School (mock-first) for new code
- Use event sourcing for state changes
- Ensure input validation at system boundaries

### Project Config

- **Topology**: hierarchical-mesh
- **Max Agents**: 15
- **Memory**: hybrid
- **HNSW**: Enabled
- **Neural**: Enabled

## AQE Project Scope

- This project contains ~84 AQE/QE skills and separate Claude Flow platform skills
- When working with skills, ALWAYS distinguish between AQE/QE skills and Claude Flow platform skills
- Only count/modify AQE skills unless explicitly told otherwise — do NOT include Claude Flow platform skills
- AQE skills live under `.claude/skills/` but exclude platform infrastructure skills (v3-*, flow-nexus-*, agentdb-*, reasoningbank-*, swarm-*)
- `.claude/agents/v3/` contains BOTH QE agents (qe-*.md, shipped to users) AND project-internal agents (v3-*, adr-*, security-*, sparc-*, etc., NOT shipped)
- Only `qe-*.md` agents are part of the AQE fleet for users — non-qe agents are Claude Flow platform or project-specific agents
- The `assets/agents/v3/` directory contains ONLY qe-*.md agents for npm distribution — do NOT copy non-QE agents there
- Memory namespaces like `aqe/v3/domains/*` are database identifiers, NOT filesystem paths — never change them during structural refactors

## Database Architecture

- Unified persistence system: all data goes through SQLite (better-sqlite3) — one DB, one schema

## Build & Test

```bash
# Build
npm run build

# Test
npm test

# Lint
npm run lint
```

- ALWAYS run tests after making code changes
- ALWAYS verify build succeeds before committing
- NEVER simulate or mock tests when asked to run tests — always run real commands against the actual codebase unless explicitly told to simulate
- When debugging, always reproduce with real commands first — do not guess at root causes
- Use `/debug-loop` skill for hypothesis-driven autonomous debugging

## Production Safety

- Before modifying adapter code or any module used in production, explain the change and its production impact before applying it
- Wait for user confirmation on changes that could affect live users or published packages
- When fixing bugs, grep for ALL instances of the problematic pattern across the entire codebase before patching — never assume a value only appears in one place

## Bug Fix Verification (Mandatory)

- **Reproduction-First**: Before closing any bug, run the **exact reproduction steps** from the issue on a real project — not just unit tests
- **MCP-CLI Parity**: Every fix that touches a CLI code path MUST also be verified via MCP (and vice versa). The two paths diverge frequently.
- **No Batch-Close Without Per-Issue Verification**: A single commit can fix multiple issues, but each issue needs its own reproduction test
- **Smoke Test Before Release**: Run the top MCP tools and CLI commands against a fixture project before tagging a release. If any crash or return empty/fabricated data, block the release.
- **Never Claim Fixed Without Evidence**: Post the actual output (command + result) in the issue/PR before marking fixed
- **Integration Tests Required for MCP**: Unit tests of handler functions are insufficient. MCP fixes must be verified by making real MCP tool calls through the protocol server.

## Releases & Publishing


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [proffesor-for-testing/agentic-qe](https://github.com/proffesor-for-testing/agentic-qe) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
