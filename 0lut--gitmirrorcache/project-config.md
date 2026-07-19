---
trigger: always_on
description: Keep this file short and current. Prefer the repo's checked-in docs, scripts,
---

# Agent Guidelines - gitmirrorcache

Keep this file short and current. Prefer the repo's checked-in docs, scripts,
and tests over ad-hoc operational steps.

## Agent Docs Layout

- This file is shared guidance for any agent working in this repository. Keep
  repo-wide safety rules and current architecture contracts here.
- Local-only runbooks live under [`.agents/skills/`](.agents/skills/). They
  require no secrets or live-infrastructure access and should be runnable by any
  agent with the local repo, Rust toolchain, and Git.
- Privileged operational runbooks live under [`.devin/skills/`](.devin/skills/).
  They require a suitable VM, explicit credentials, network access to AWS, and
  authorization to mutate live infrastructure.
- When a workflow applies to multiple environments, keep the policy here and
  link to the capability-specific runbook that owns the step-by-step procedure.

## Git Boundaries

- Unvalidated git arguments are production-safety bugs: option-looking values
  can become flag injection, and NUL bytes can truncate what git receives.
- Any public or private boundary that moves external or caller-derived input
  toward `git` must validate at the top, before building refspecs/config/env
  entries or calling `self.run(...)`, `run_upstream(...)`, or `spawn(...)`.
- Treat API paths, query params, request bodies, headers, config, URLs, refs,
  revisions, refspecs, filters, depths, and upload-pack intent as external
  unless the value was created entirely inside the wrapper.
- Keep repo path inputs behind `RepoKey`, `repo_from_git_path`, and
  `validate_host`; never join raw URL/path segments into cache paths.
- Use the narrowest helper: `reject_ref_arg`, `reject_revision_arg`,
  `reject_config_key`, `reject_remote_url`, `reject_refspec`,
  `reject_fetch_filter`, `reject_fetch_depth`, or `reject_nul`.
- External strings must reject empty values, leading `-` when git may parse them
  as args, and NUL bytes. Refs reject `:`; revisions may allow `HEAD:path`;
  refspecs may allow `+` and `:`.
- Put `--` before positional args wherever git accepts it.
- Keep upstream auth out of argv, logs, and manifests. Use `with_upstream_auth`
  and the wrapper's `GIT_CONFIG_*` env plumbing for credentials.
- Remote Git URLs should come from configured upstream roots or validated
  `upstream_url` construction; do not add arbitrary caller-supplied fetch/proxy
  URLs.
- New wrapper checklist: identify external inputs, choose/create the narrowest
  validator, validate before composing git args, add `--` or `--end-of-options`
  where supported, and test rejected leading-dash/NUL input plus any
  helper-specific constraints.

## Test Layout

- Keep tests out of regular module bodies. Put `#[test]` and `#[tokio::test]`
  functions in dedicated test modules: use `#[cfg(test)] mod tests;` with
  sibling `tests.rs`/`tests/` files, or integration tests under `tests/`. Do not
  add inline `mod tests { ... }` blocks to regular source files.

## Current Cache Contract

- `MaterializeRequest` is intentionally small: `repo`, `selector`, and optional
  `upstream_authorization`. Request bodies deny unknown fields; do not revive
  the removed `mode` or session contract.
- HTTP materialize/resolve should call `Materializer::materialize` or
  `Materializer::resolve` directly after rate limiting and auth checks. Keep the
  worker coordinator for cron, event hints, and explicit warming.
- Exact commit materialization should use complete cached generation metadata
  before contacting upstream. Branch and default-branch materialization must
  verify upstream refs.
- Upstream want hydration flows through the shared batched read-through fetch
  core (direct Git read-through and the proxy-on-miss warm); branch
  materialization shares the same `branch_cache_refspec` construction. Exact
  commit cold misses deliberately fetch all heads so descendant exact-commit
  requests reuse the full generation bundle.
- Direct Git uses `/git/{host}/{owner}/{repo}.git`, rejects receive-pack, and
  serves from the shared bare repo under `cache_root/repos/...`.
- Direct Git GET proves repo access via `ls-remote` only. POST read-through uses
  the same request-scoped auth and must preserve shallow/blobless intent
  (`depth`, `blob:none`) when fetching wants.
- Cold-miss proxying defaults to `git_remote.proxy_on_miss_by_default` (on);
  the `git-cache-use-proxy-on-miss` header is the only per-request override
  (falsey values opt out). Proxy only
  HTTP(S) upstreams, enforce streamed byte limits, forward auth only to upstream,
  then queue bounded background cache work. Proxy readiness and local warm paths
  should not hydrate generation manifests inline; after a branch-tip proxy miss
  finishes, queue async materialization so durable generation manifests are
  published outside the client response path.
- IMPORTANT testing caveat: because proxying is on by default, any test or
  benchmark that means to exercise the local read-through (cache-fill) path
  against an HTTP(S) upstream MUST opt out explicitly — set
  `git_remote.proxy_on_miss_by_default = false` in test configs (the shared
  API test support config does this), or send
  `git -c http.extraHeader='git-cache-use-proxy-on-miss: false'` per request.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0lut/gitmirrorcache](https://github.com/0lut/gitmirrorcache) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-18 -->
