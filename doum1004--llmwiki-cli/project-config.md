---
trigger: always_on
description: A CLI tool that helps LLM agents (Claude Code, Codex, etc.) build and maintain personal knowledge bases. The CLI is the hands — it reads, writes, searches, and manages wiki files. The LLM is the brain — it decides what to create, update, and connect.
---

# LLM Wiki CLI

## What This Is

A CLI tool that helps LLM agents (Claude Code, Codex, etc.) build and maintain personal knowledge bases. The CLI is the hands — it reads, writes, searches, and manages wiki files. The LLM is the brain — it decides what to create, update, and connect.

**Key principle: The CLI never calls any LLM API. It is a pure storage tool with pluggable backends (filesystem, git).**

## Tech Stack

- **Runtime**: Bun (development), Node.js (published bundle)
- **Language**: TypeScript
- **Dependencies**: commander (CLI), js-yaml (YAML parsing)
- **Build**: `bun build` bundles to `dist/wiki.js` targeting Node.js
- **Tests**: Bun test runner (`bun test`)

## LLM Agent Skill Guide

Run `wiki skill` to print the full guide, or see [`docs/SKILL.md`](docs/SKILL.md) for the source. Covers workflows, command patterns, and gotchas for LLM agents.

## Project Structure

```
bin/wiki.ts              # CLI entry point (source)
dist/wiki.js             # Built bundle (npm-published artifact)
src/
  types.ts               # Shared TypeScript interfaces
  lib/
    storage.ts           # StorageProvider factory (createProvider)
    wiki.ts              # WikiManager: filesystem StorageProvider
    git-provider.ts      # GitProvider: filesystem + auto-commit + auto-push
    config.ts            # .llmwiki.yaml read/write
    registry.ts          # Global registry (~/.config/llmwiki/registry.yaml)
    resolver.ts          # Wiki resolution chain (--wiki → cwd → walk up → default)
    git.ts               # Git operations via child_process.execFile
    github.ts            # GitHub API: createRepo, getUsername, enablePages
    git-credentials.ts   # resolvedGitToken: env LLMWIKI_GIT_TOKEN / GITHUB_TOKEN / GIT_TOKEN, then YAML
    templates.ts         # Default file content (SCHEMA.md, index.md, log.md, viz workflow/scripts)
    search.ts            # Full-text search with term-frequency ranking
    index-manager.ts     # IndexManager: uses StorageProvider for index.md
    log-manager.ts       # LogManager: uses StorageProvider for log.md
    frontmatter.ts       # YAML frontmatter parse/detect/add
    link-parser.ts       # Wikilink extraction and link graph building
  commands/
    init.ts              # wiki init (--backend, --git-token, --viz)
    registry.ts          # wiki registry
    use.ts               # wiki use
    read.ts              # wiki read
    write.ts             # wiki write
    append.ts            # wiki append
    list.ts              # wiki list
    search.ts            # wiki search
    index-cmd.ts         # wiki index (add/remove/show)
    log-cmd.ts           # wiki log (append/show)
    lint.ts              # wiki lint
    links.ts             # wiki links
    backlinks.ts         # wiki backlinks
    orphans.ts           # wiki orphans
    status.ts            # wiki status
    skill.ts             # wiki skill (print LLM agent guide)
test/
  init.test.ts           # Config, registry, resolver, templates, init integration
  git.test.ts            # Git operations (commit, log, diff, remote, branch)
  read-write.test.ts     # WikiManager page operations
  storage.test.ts        # StorageProvider factory
  filesystem-provider.test.ts # Filesystem provider contract tests
  git-provider.test.ts   # GitProvider auto-commit tests
  git-credentials.test.ts # resolvedGitToken precedence
  github.test.ts         # GitHub API with mocked fetch
  commands.test.ts       # End-to-end CLI command integration tests
docs/
  phase-1.md             # Phase tracking files
  phase-2.md
  phase-3.md
  phase-4.md
  phase-5.md
scripts/
  generate-viz-scripts.ts # Extracts viz build scripts from templates.ts (used by demo workflow)
test-wiki-page/
  wiki/                  # Example wiki pages for live demo on GitHub Pages
    index.md
    log.md
    concepts/
    sources/
    synthesis/
```

## Commands

### Wiki Management
```
wiki init [dir] --name --domain --backend <filesystem|git>
wiki init [dir] --backend git --git-token <pat> [--git-repo owner/repo]
wiki init [dir] --backend git --no-viz              # Skip visualization scaffolding
wiki init [existing-wiki-dir] --viz                 # Add visualization to existing git wiki
wiki registry                       # List all wikis
wiki use [wiki-id]                  # Set active wiki
```

### Reading & Writing
```
wiki read <path>                    # Print page to stdout
wiki write <path>                   # Write stdin to page
wiki append <path>                  # Append stdin to page
wiki list [dir] [--tree] [--json]   # List pages
wiki search <query> [--limit N] [--json]  # Search pages
```

### Index & Log
```
wiki index show                     # Print master index
wiki index add <path> <summary>     # Add entry to index
wiki index remove <path>            # Remove entry
wiki log show [--last N] [--type T] # Print log entries
wiki log append <type> <message>    # Append log entry
```

### Health & Links
```
wiki lint [--json]                  # Check wiki health (broken links, orphans, frontmatter, index)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doum1004/llmwiki-cli](https://github.com/doum1004/llmwiki-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
