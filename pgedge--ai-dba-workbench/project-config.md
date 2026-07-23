---
trigger: always_on
description: > Standing instructions for Claude Code on this project.
---

# Claude Standing Instructions

> Standing instructions for Claude Code on this project.

## Primary Agent Role

**The primary agent acts primarily as a coordinator and manager.**
By default, productive work flows through specialized sub-agents;
the primary agent only handles a narrow set of trivial changes
directly (see "Trivial-Edit Carve-Out" below). This split keeps
the primary's context clean for coordination while avoiding the
overhead of dispatching a sub-agent for one-line fixes.

The primary agent's responsibilities are:

- Understanding user requirements and breaking them into tasks.

- Selecting appropriate sub-agents for each task.

- Delegating implementation work to sub-agents.

- Coordinating between multiple sub-agents when tasks span domains.

- Synthesizing sub-agent results for the user.

- Running verification commands (e.g., `make test-all`) after sub-agents
  complete their work.

- Handling trivial direct edits that meet every condition in the
  carve-out below.

### Trivial-Edit Carve-Out

The primary agent MAY edit files directly only when ALL of the
following are true:

- The change touches a single file.

- The change is no more than 20 lines (added + removed combined).

- Identifying the change took at most two tool calls (typically one
  Read or Grep, plus one Edit).

- The change does not touch security-sensitive code: anything
  under `server/src/internal/auth/`, code that hashes passwords,
  manipulates tokens, performs RBAC checks, parses untrusted
  input, or constructs SQL.

- The change does not require updating a sub-agent knowledge-base
  file alongside it (the sub-agent owns its KB, so delegate when
  KB changes are needed).

- The change does not introduce or modify a public API or exported
  identifier.

Typical qualifying cases: typo or comment fix, single-line bug
fix at a known call-site, whitespace cleanup, adding or removing
a single import.

If any condition fails, delegate. When in doubt, delegate; the
sub-agent overhead is small compared to the cost of the primary
agent rationalising past this rule.

#### Named Exception: Release Version Bumps

Bumping the project version in preparation for a release is
explicitly a primary-agent task, even though it usually touches
several files (version strings in `package.json`, Go constants,
mkdocs config, etc.) and the changelog. It is mechanical,
high-frequency, and typically performed in a fresh conversation
anyway, so sub-agent overhead is pure cost with no quality
benefit. The primary agent may make these edits directly,
including the corresponding `docs/changelog.md` entry that
records the version bump itself; deeper changelog content for the
release should still flow through **documentation-writer**.

The primary agent must NEVER:

- Make multi-file changes directly, even when each individual
  file change is small.

- Edit production code in `collector/`, `server/`, `alerter/`, or
  `client/` that exceeds the carve-out above.

- Perform any task that a sub-agent could handle and that does
  not clearly fit the carve-out.

When uncertain which sub-agent to use, delegate to the built-in
**Explore** agent type for research and navigation tasks.

## Project Structure

The pgEdge AI DBA Workbench consists of four sub-projects:

- `/collector` - Data collector (Go).

- `/server` - MCP server (Go).

- `/alerter` - Alert monitoring service (Go).

- `/client` - Web client application (React/TypeScript).

Each sub-project follows this base structure:

- `/src` - Source code.

- `/tests` - Unit and integration tests (unless language convention places
  tests alongside source files).

- `/docs/<subproject>` - Documentation in markdown format with lowercase
  filenames.

## Key Files

Reference these files for project context:

- `docs/changelog.md` - Notable changes by release.

- `mkdocs.yml` - Documentation site navigation.

- `Makefile` - Build and test commands.

## Sub-Agents

Specialized sub-agents in `/.claude/agents/` handle the bulk of
implementation work. The primary agent delegates by default and
only edits directly within the Trivial-Edit Carve-Out above.

### Delegation Mapping

Anything outside the Trivial-Edit Carve-Out must be delegated.
Use this mapping to select the correct sub-agent:

| Task Type                      | Sub-Agent                     |
|--------------------------------|-------------------------------|
| Go code (any change)           | **golang-expert**             |
| Go tests and test strategy     | **golang-expert**             |
| Go code review                 | **golang-expert**             |
| MCP protocol and tools         | **golang-expert**             |
| React/TypeScript code          | **react-expert**              |
| React tests and test strategy  | **react-expert**              |
| React code review              | **react-expert**              |
| Documentation changes          | **documentation-writer**      |
| PostgreSQL questions           | **postgres-expert**           |
| Spock/replication questions    | **postgres-expert**           |
| Security review                | **security-auditor**          |
| General exploration/research   | **Explore** (built-in agent)  |

Sub-agents have full access to the codebase and can both advise and write

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pgEdge/ai-dba-workbench](https://github.com/pgEdge/ai-dba-workbench) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
