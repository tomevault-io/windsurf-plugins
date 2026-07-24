---
trigger: always_on
description: provides:
---

## Local Scope First (`nebulex_redis_adapter`)

This repository is `nebulex_redis_adapter` (the Redis adapter for Nebulex),
not Nebulex core. When imported Nebulex sections reference missing
`usage-rules/*.md` paths or Nebulex-core files, treat them as upstream
guidance and prioritize this repository's local files and modules.

### Local Rule Precedence (for this repo)

1. This local preface.
2. `nebulex:workflow` section in this file.
3. `nebulex:nebulex` section in this file (as framework guidance).
4. `nebulex:elixir-style` and `nebulex:elixir` sections in this file.

### Local Key Files

- `lib/nebulex/adapters/redis.ex` - Main Redis adapter implementation.
- `lib/nebulex/adapters/redis/options.ex` - Adapter option definitions/docs.
- `lib/nebulex/adapters/redis/client.ex` - Redis client abstraction.
- `lib/nebulex/adapters/redis/connection.ex` - Connection management.
- `lib/nebulex/adapters/redis/pool.ex` - Connection pooling.
- `lib/nebulex/adapters/redis/supervisor.ex` - Adapter supervision tree.
- `lib/nebulex/adapters/redis/cluster.ex` - Redis Cluster mode support.
- `lib/nebulex/adapters/redis/cluster/` - Cluster internals (config manager, keyslot, pools).
- `lib/nebulex/adapters/redis/client_side_cluster.ex` - Client-side cluster mode support.
- `lib/nebulex/adapters/redis/client_side_cluster/` - Client-side cluster internals (hash ring, pools).
- `lib/nebulex/adapters/redis/serializer.ex` - Serialization behaviour.
- `lib/nebulex/adapters/redis/helpers.ex` - Shared adapter utilities.
- `lib/nebulex/adapters/redis/error_formatter.ex` - Error formatting.
- `test/nebulex/adapters/redis/standalone_test.exs` - Standalone mode tests.
- `test/nebulex/adapters/redis/cluster_test.exs` - Redis Cluster mode tests.
- `test/nebulex/adapters/redis/client_side_cluster_test.exs` - Client-side cluster tests.
- `test/nebulex/adapters/redis/client_test.exs` - Client abstraction tests.
- `test/shared/` - Shared test cases (cache, queryable, info, command errors).
- `README.md` - Public usage/configuration for this adapter.
- `CHANGELOG.md` - Adapter release history.

<!-- usage-rules-start -->
<!-- nebulex:workflow-start -->
## nebulex:workflow usage
# Agent Workflow

## Rule Index

Start here, then read these at session start and refer back while coding:

- `usage-rules/nebulex.md` - Nebulex-specific rules
- `usage-rules/elixir-style.md` - Style guidelines
- `usage-rules/elixir.md` - Core Elixir rules

> If these files are not found, check `AGENTS.md` or the local
> `usage-rules/` folder instead.

## Rule Precedence

When rules conflict, prioritize them in this order:

1. `usage-rules/workflow.md`
2. `usage-rules/nebulex.md`
3. `usage-rules/elixir-style.md`
4. `usage-rules/elixir.md`

> If these files are not found, apply the same precedence to the
> corresponding sections in `AGENTS.md`.

## Session Bootstrap

At the start of each session, quickly establish context:

1. Run `git status --short` and `git diff --name-only` to check
   local modifications and currently touched files.
2. Run `git log --oneline -20` to see recent changes.
3. Run `git branch -a` to see active branches and current branch.
4. Read `README.md` and the latest section of `CHANGELOG.md`.
5. Check `.tool-versions` or the `elixir` version in `mix.exs` for
   supported Elixir/OTP versions.

If on a feature branch, also run:

6. `git log --oneline main..HEAD` to see the branch's commits.
7. `git diff main...HEAD` to understand the branch's full scope.

When relevant to the task:

8. Check open issues and PRs with `gh issue list` and `gh pr list`.
   If `gh` is unavailable or unauthenticated, skip this step.

## Current Project Status

- **Latest release**: check the latest section in `CHANGELOG.md`.
- Read `CHANGELOG.md` for recent features, breaking changes, and
  the project's direction.
- Changelog policy: user-visible behavior changes should be documented;
  internal refactors may be omitted before a release.

## PR Workflow

### Reviewing PRs

1. Read the PR description and all comments:
   `gh pr view <number>` and `gh pr view <number> --comments`.
2. Review the diff: `gh pr diff <number>`.
3. Check `CHANGELOG.md` to understand if the change aligns with the
   project's direction.
4. Verify code follows `usage-rules/` conventions (Elixir patterns,
   Nebulex-specific rules, style guidelines).
5. Run the validation commands (see below) before approving.
6. Provide constructive feedback referencing specific lines and
   conventions.
7. Structure review feedback as:
   - findings first (ordered by severity, with file:line references),
   - open questions/assumptions,
   - brief summary last.

### Opening PRs

1. Branch from `main` with a descriptive branch name
   (e.g., `fix/some-bug`, `feat/cache-warming-support`).
2. Update `CHANGELOG.md` under the appropriate section
   (Enhancements, Bug fixes, Backward-incompatible changes).
3. Run all validation commands before pushing.
4. Reference related GitHub issues in the PR description
   (e.g., "Closes #123").
5. Use `gh pr create` with a clear title and description.

## Commit Messages

Commit messages must follow the
[Conventional Commits](https://www.conventionalcommits.org/) format:

```text
type(scope): short summary
```

### Allowed Types

- `feat`
- `fix`
- `refactor`
- `docs`
- `test`
- `chore`
- `perf`
- `ci`
- `build`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [elixir-nebulex/nebulex_redis_adapter](https://github.com/elixir-nebulex/nebulex_redis_adapter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
