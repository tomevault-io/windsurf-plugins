---
trigger: always_on
description: - Use the `@docs/` directory as the source of truth for project contracts and implementation documents.
---

### Instructions

- Use the `@docs/` directory as the source of truth for project contracts and implementation documents.
- All repository-wide rules must be defined in the appropriate AGENTS.md.
- List files in `docs/` before starting each task, and keep `docs/` up-to-date.
- After completing each task, update the relevant `AGENTS.md` and `docs/` files in the same change when policies, structure, or contracts changed.
- For documentation authoring and editing tasks, do not arbitrarily omit, delete, or simplify requested or source-backed content; if content, scope, or intent is ambiguous, ask the user before deciding what to remove, merge, or reinterpret; if the documentation change affects repository or domain policy boundaries, update or create the relevant `AGENTS.md` file in the same change when needed.
- Write all code and comments in English.
- When introducing a workaround, leave sufficient comments that explain why it exists, its scope, and the conditions for removing it.
- Prefer enum types over strings whenever possible.
- If you modified Rust code, run `cargo test` from the root directory before finishing your task.
- If you modified frontend code, run `pnpm test` from the frontend directory before finishing your task.
- Commit your work as frequent as possible using git. Do NOT use `--no-verify` flag.
- Run `git commit` only after `git add`; once files are staged, commit without unnecessary delay so staged changes are preserved in history.
- Committing may require workspace binaries (for example, git hooks). If required binaries are missing, run `pnpm install` at the repository root and retry the commit.
- After addressing pull request review comments and pushing updates, mark the corresponding review threads as resolved.
- When no explicit scope is specified and you are currently working within a pull request scope, interpret instructions within the current pull request scope.
- Do not guess; rather search for the web.
- Debug by logging. You should write enough logging code.
- Write sufficient logs for debugging and operational troubleshooting.
- Prefer structured logging libraries for business and system logs (Go: `log/slog`, Rust: `tracing`).
- Prioritize Connect RPC-based communication for business flows over Tauri-specific bindings.
- Prefer React Query for frontend server-state management when it is available.
- When using React Query with Connect RPC, use `@connectrpc/connect-query` from `https://github.com/connectrpc/connect-query-es`.
- When accessing `github.com`, use the GitHub CLI (`gh`) instead of browser-based workflows when possible.
- Run GitHub CLI (`gh`) commands outside sandbox restrictions by default; use the required approval flow when escalation is needed.
- When writing shell commands or scripts, treat backticks and command substitution carefully, prefer `$(...)` over legacy backticks, and apply strict escaping for all dynamic values.
- If an operation is blocked by sandbox restrictions, retry it without sandbox restrictions using the required approval flow.

### Monorepo Structure Map

- `docs/`: Source of truth for project contracts and repository documentation.
- `apps/`: User-facing apps (Next.js and React Native).
- `crates/`: Rust crates and Rust-based tooling.
- `protos/`: Shared Connect RPC proto contracts used by multi-runtime projects.
- `cmds/`: Go command tools for workflow orchestration.
- `servers/`: Backend services and APIs.
- `packaging/`: Package-manager template assets for release automation.
- `.agents/skills/`: Workspace-local Codex skills and reusable agent workflows.

### Canonical Directory Map

- `docs/README.md`: Canonical docs catalog and naming rules.
- `docs/project-template.md`: Required structure for `project-<id>` index docs.
- `docs/domain-template.md`: Required structure for domain-level contract docs.
- `docs/project-<id>.md`: Canonical project index docs (ownership + domain-doc index + cross-domain invariants).
- `docs/<domain>-<project-or-component>-<contract>.md`: Canonical domain contract docs (`apps`, `cmds`, `servers`, `crates`, `protos`, `packages`).
- `docs/project-cargo-mono.md`: Cargo subcommand project index.
- `docs/project-nodeup.md`: Node.js version manager project index.
- `docs/project-with-watch.md`: Command rerun watcher CLI project index.
- `docs/project-derun.md`: Derun CLI project index.
- `docs/project-ttl.md`: TTL compiler project index.
- `docs/project-mpapp.md`: Expo mobile app project index.
- `docs/project-devkit.md`: Devkit host platform project index.
- `docs/project-devkit-commit-tracker.md`: Commit Tracker scaffold project index.
- `docs/project-devkit-remote-file-picker.md`: Remote File Picker mini app project index.
- `docs/project-thenv.md`: Thenv multi-component project index.
- `docs/project-public-docs.md`: Public docs app project index.
- `docs/project-serde-feather.md`: Serde Feather multi-crate project index.
- `docs/project-rustia.md`: Rustia multi-crate project index.
- `docs/project-dexdex.md`: DexDex multi-runtime project index.
- `docs/crates-with-watch-foundation.md`: with-watch CLI and watcher foundation contract.
- `docs/crates-rustia-core-foundation.md`: Rustia core runtime LLM data contract.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [delinoio/oss](https://github.com/delinoio/oss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
