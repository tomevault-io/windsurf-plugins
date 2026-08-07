---
trigger: always_on
description: These rules apply to the entire repository. More specific guidance lives in [`src/AGENTS.md`](src/AGENTS.md) and [`src-tauri/AGENTS.md`](src-tauri/AGENTS.md); a child file adds only rules for its own subtree and does not replace this file.
---

# MangoDisk Contribution Guidelines

These rules apply to the entire repository. More specific guidance lives in [`src/AGENTS.md`](src/AGENTS.md) and [`src-tauri/AGENTS.md`](src-tauri/AGENTS.md); a child file adds only rules for its own subtree and does not replace this file.

## Product and architecture

MangoDisk is a Tauri 2 desktop application for safe disk analysis and cleanup. The Vue frontend is an adapter for user interaction. Rust owns filesystem access, platform safety, scan orchestration, cleanup execution, and persisted operational data.

Keep these domain boundaries stable:

- `cleanup`: cleanup rules, previews, preflight, execution, and verification;
- `storage`: disk analysis, large-file discovery, and exact duplicate detection;
- `applications`: installed application identity and application-scoped capabilities;
- `filesystem`: paths, traversal, permanent deletion, identity, and protection primitives;
- `history` and `reporting`: operation records and machine-readable evidence;
- `platform`: OS contracts and implementations, not product orchestration;
- GUI, CLI, and Tauri commands: sibling adapters over the same Core use cases.

`duplicates` is the exact-content duplicate domain. Use `duplicate_files` only for a file-specific entity, use case, command, event, or page. A future similar-photo feature belongs to a separate similarity domain and must not weaken exact duplicate semantics.

Do not create broad modules such as `common`, `misc`, `manager`, `optimization`, or a new service that aggregates unrelated domains. Product pages may combine domain results without moving that coordination into a giant Core service.

## Implementation principles

1. Prefer the smallest practical design that preserves a clear boundary. Do not add abstractions for hypothetical reuse.
2. Keep side effects at adapters and domain boundaries. Pure classification, formatting, and calculation must remain deterministic.
3. Make behavior configurable when contributors can safely extend it as data. Cleanup rules should normally be declarative TOML validated at build time, not Rust branches.
4. Do not couple rule resources to UI localization. Rules expose stable IDs, categories, risks, capabilities, and diagnostics; locale files own user-facing text.
5. Preserve safety invariants: dry-run, protected paths, link/reparse-point handling, preflight, explicit confirmation, verification, and safe fallback when capabilities are unavailable.
6. Never hide a compatibility or performance regression with an allow attribute, a global machine setting, or an undocumented fallback.

## Naming and text

- Rust files and modules use `snake_case`; frontend, documentation, and resource file names use `kebab-case` unless an external tool requires another format.
- Source comments, logs, diagnostic codes, test names, and assertions must be clear and consistent. Comments explain reasons, risks, and non-obvious boundaries rather than restating code.
- User-facing text belongs in locale resources. Update every supported locale when contributor-visible behavior changes.
- Use stable typed enums or codes across process and persistence boundaries. Do not make UI logic infer behavior from free-form messages.

## Repository hygiene

- Existing and untracked changes belong to the user unless proven otherwise. Do not overwrite, delete, reformat, or include unrelated changes in a commit.
- Do not edit generated or third-party source to implement application-wide behavior.
- Import project-owned business modules from concrete files. Do not add business barrel files.
- Keep generated reports out of production source directories and use short, stable paths that can be checked out on Windows.
- Do not commit credentials, personal file contents, raw private paths in Markdown, build outputs, or local dependency directories.
- Logs and diagnostics must not expose raw filesystem paths, file names, file contents, installation identifiers, or other user-specific metadata. Prefer operation IDs, counts, timings, typed reason codes, error digests, and explicitly redacted labels.
- Do not push unless the user explicitly requests it. A request to commit does not authorize push.

## Workflow

1. Read the nearest `AGENTS.md` before changing code.
2. Inspect the worktree and establish a behavior or performance baseline proportional to risk.
3. Write a concise implementation plan for multi-step work and keep it updated when scope changes.
4. Keep structural moves separate from behavior changes whenever Review would otherwise become ambiguous.
5. Validate cross-platform behavior in an applicable macOS and Windows environment. If a platform is unavailable, document the unvalidated scope instead of claiming support from local results.
6. For behavior or performance changes, describe the tested workload, environment, and observable result in the change description without committing raw machine reports or private data.
7. Review the final diff for correctness, safety, privacy, platform fallbacks, naming, and stale documentation before committing.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [harry0703/MangoDisk](https://github.com/harry0703/MangoDisk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-07 -->
