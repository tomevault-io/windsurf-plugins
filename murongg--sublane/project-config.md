---
trigger: always_on
description: Before significant changes, read `PRODUCT.md` for scope and status, `DESIGN.md` for interface rules, and [the architecture guide](https://sublane.dev/docs/architecture) for module ownership and implementation boundaries.
---

# Agent instructions

## Context

Before significant changes, read `PRODUCT.md` for scope and status, `DESIGN.md` for interface rules, and [the architecture guide](https://sublane.dev/docs/architecture) for module ownership and implementation boundaries.

Use English for code comments, `PRODUCT.md`, and primary developer documentation. Keep the English and Simplified Chinese UI dictionaries complete. English is the default interface language.

## Work style

Usage, operations, and architecture guides are maintained in [the documentation repository](https://github.com/murongg/sublane-website). Update guides there instead of duplicating them here; see [the documentation index](docs/README.md) for references retained locally.

- Make the smallest complete, scoped change; reuse existing mechanisms instead of speculative frameworks, packages, or services. Report assumptions and actual verification results concisely; live subscription and desktop compatibility require separate evidence.
- Do not create commits, publish a repository, or deploy without a user request.

## Releases

- Publishing a version is not complete until `CHANGELOG.md` includes that tag. The tag workflow generates GitHub Release notes but does not update the repository file.
- After the release workflow succeeds, use `git-cliff 2.14.2` to run `make changelog-check` and `make changelog`. Review the new version section and comparison link, then commit the changelog and merge it into `main`.

## Boundaries

- Follow the architecture guide's ownership boundaries. Keep allocation accounting separate from provider IO, browser authorization, and billing; membership, policy, storage, and version policy must not depend on SDK types.
- Backups use read-only SQLite snapshots, bounded archives, integrity/key validation, and atomic publication to new paths. Never overwrite live data or expose backups to unauthenticated users or members.
- New pools grant no member access until configured. Gateway keys have immutable pool bindings and never authenticate browser sessions. Secret disclosure is owner-only and audited; revocation removes encrypted values while legacy hash-only keys remain usable without disclosure.
- Keep one-to-one metadata on its owner (`api_keys`, `memberships`, `settings`); separate counters, relations, and history when lifecycle or cardinality requires it.
- Production embeds the client-rendered React app in Go; it must not require a Node.js server.
- Use pinned public CLIProxyAPI SDK executors through `internal/upstream`. Keep the SDK credential store empty, watcher inert, loopback routes blocked, and automatic refresh disabled. Never register live credentials or pass refresh tokens to execution auth. Only the explicit Antigravity refresh in `accounts.Prepare` may receive a refresh token; persist its result before execution. Never import upstream `internal` packages.

## Backend

- Use standard Go conventions, explicit dependency injection at test seams, and `net/http`-compatible handlers; domain services must not depend on chi.
- Default to loopback listening. Protect workspace management routes with administrator authentication; platform settings and backups require the platform owner. Enforce roles on frontend routes and backend requests without bypasses. Browser roles come from active membership; gateway workspace identity comes from the key, never the browser selection header. See [authentication](https://sublane.dev/docs/authentication).
- Use chi `Route`, method registrations, and router-level `Use` for session/role/gateway authentication, including 404/405 responses. Reserve `With` for endpoint-specific checks. Never dispatch methods or gateway paths inside handlers or serve HTML for API errors.
- First-run username/password setup atomically creates the initial workspace owner with a single-administrator guard. Password changes atomically revoke browser sessions; session creation rechecks the verified password hash. OAuth attempts remain session-bound, with PKCE where supported.
- Write application queries in `internal/storage/queries/` and run `make generate`. Never hand-edit `internal/storage/db/`; keep generated code with its SQL changes. Migration bootstrap SQL remains in storage.
- Domain services own transactions; use `queries.WithTx(tx)` throughout, never hold transactions during network IO or generation, and keep database rows separate from public responses. Successful mutation audit events share the domain transaction; automatic refresh is not manual authorization.
- Preserve existing SQLite migrations; use ordered additive migrations for subsequent schema changes.
- Serialize account credential changes and preserve upstream identity on reauthorization. Persist rotated credentials, model/quota snapshots, and version policy before publishing them. Official release checks stay bounded, respect manual pins, and never install executables. Quota cache reads check account enablement; stale values retain their observation time.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [murongg/SubLane](https://github.com/murongg/SubLane) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
