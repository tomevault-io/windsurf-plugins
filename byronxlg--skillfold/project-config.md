---
trigger: always_on
description: Configuration language and compiler for multi-agent AI pipelines. Compiles YAML config into standard SKILL.md files.
---

# Skillfold

Configuration language and compiler for multi-agent AI pipelines. Compiles YAML config into standard SKILL.md files.

## GitHub

- **Repo**: https://github.com/byronxlg/skillfold
- Issues, PRs, releases, actions, and all other GitHub features are available for use

## Quick Reference

- **Run compiler**: `npx tsx src/cli.ts`
- **Run pipeline**: `npx tsx src/cli.ts run --target claude-code`
- **Resume pipeline**: `npx tsx src/cli.ts run --target claude-code --resume`
- **Dry run**: `npx tsx src/cli.ts run --target claude-code --dry-run`
- **Compile to Claude Code agents**: `npx tsx src/cli.ts --target claude-code`
- **Package as plugin**: `npx tsx src/cli.ts plugin`
- **Watch mode**: `npx tsx src/cli.ts watch`
- **Validate config**: `npx tsx src/cli.ts validate`
- **Check output is current**: `npx tsx src/cli.ts --check`
- **List pipeline**: `npx tsx src/cli.ts list`
- **Interactive graph**: `npx tsx src/cli.ts graph --html > pipeline.html`
- **Search npm skills**: `npx tsx src/cli.ts search [query]`
- **Run with custom config**: `npx tsx src/cli.ts --config path.yaml --out-dir out/`
- **Build**: `npm run build`
- **Build plugin**: `npm run build:plugin`
- **Type check**: `npx tsc --noEmit`

## Project Structure

```
src/
  cli.ts          - CLI entry point, arg parsing
  config.ts       - YAML parsing, config types, validation (cycles, references)
  state.ts        - State schema parsing, type system, location validation
  resolver.ts     - Reads SKILL.md files from skill directories (local + remote)
  remote.ts       - GitHub URL parsing and remote skill fetching
  agent.ts        - Claude Code agent markdown generation from composed skills
  compiler.ts     - Recursive composition, body concatenation, orchestrator integration
  graph.ts        - Graph parsing, validation (skills, state, conflicts, cycles)
  orchestrator.ts - Orchestrator SKILL.md generation from graph definition
  plugin.ts       - Claude Code plugin packaging (skillfold plugin)
  list.ts         - Pipeline introspection (skillfold list)
  npm.ts          - npm package resolution (npm: prefix in skill refs and imports)
  search.ts       - npm registry search for skillfold-skill packages (skillfold search)
  run.ts          - Pipeline runner with conditionals, loops, and map (skillfold run)
  backends.ts     - State backend interface and GitHub integration implementations
  watch.ts        - File watching and auto-recompile (skillfold watch)
  init.ts         - skillfold init scaffolding
  integrations.ts - Built-in state integrations (GitHub issues, discussions, pull requests)
  errors.ts       - ConfigError, ResolveError, CompileError, GraphError
skills/           - Atomic skill definitions (each has a SKILL.md)
library/          - Shared skills library (11 generic skills + 3 example configs)
plugin/           - Claude Code plugin (library skills + /skillfold command)
docs/             - Getting-started tutorial and documentation
dist/             - tsc compiled JS (npm package, gitignored)
build/            - Compiled skill output (default --out-dir, gitignored)
skillfold.yaml         - Pipeline config for the dev team itself
skillfold.schema.json  - JSON Schema for config validation and IDE autocompletion
action.yml             - Reusable GitHub Action for CI integration (skillfold --check)
BRIEF.md               - Full design brief
```

## Workflow

- Push commits to GitHub frequently to maintain visibility of progress. Don't let work accumulate locally.

## Code Conventions

- TypeScript, strict mode, ESM modules
- Node stdlib imports use `node:` prefix
- Imports: node stdlib -> third-party -> local, alpha within groups
- File extensions in imports: `.js` (NodeNext module resolution)
- Custom errors extend Error with descriptive messages including skill name context
- No `any`, no unnecessary type assertions

## Config Structure

Four top-level sections: `resources`, `skills`, `state`, `team`.

- **resources**: Resource declarations mapping group names to namespace URLs for state location validation and orchestrator URL resolution
- **skills.atomic**: Path references to atomic skill directories
- **skills.composed**: Composition declarations combining atomic skills into agents
- **state**: Typed state schema (top-level, importable independently)
- **team.orchestrator**: Optional skill name to append generated plan to
- **team.flow**: Directed execution graph with conditional routing, loops, parallel map, and sub-flow imports

Imports pull in `skills`, `state`, and `resources`, ignore `team`.

## Design Brief Summary

Read BRIEF.md for full context. Key points:

- **Skill composition**: Atomic skills define reusable fragments. Composed skills concatenate atomic skill bodies in declared order. Composition is recursive.
- **Team flow**: Agents wired into typed execution graphs with conditional routing, loops, and parallel map. Parsed and validated.
- **State schema**: Typed state schema with custom types, primitives, lists, and external locations. Reads/writes validated against team flow. Location paths validated against skill resource declarations at compile time.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [byronxlg/skillfold](https://github.com/byronxlg/skillfold) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-04 -->
