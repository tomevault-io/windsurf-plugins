---
trigger: always_on
description: Build one local HTML home server for generated or repo-local HTML artifacts.
---

# HTML Home Server

## Mission

Build one local HTML home server for generated or repo-local HTML artifacts.

This project replaces scattered temporary HTTP servers with one process, one port, and one searchable start page. It should make it obvious which artifacts exist, which project owns them, where their source lives, when they changed, and where to open them.

## Scope

- Serve static HTML artifacts from many GitHub repos through a single HTTP process.
- Provide a lightweight home page for browsing, searching, and opening artifacts.
- Define a small artifact manifest spec that projects can keep in their own repos.
- Index repo-declared artifacts without moving their source of truth.
- Keep project ownership explicit through a first-level `project` namespace.
- Prefer rescans and hot index updates to server restarts.

Out of scope until explicitly needed:

- Running one server per artifact or project.
- Proxying arbitrary app dev servers.
- User accounts, multi-tenant auth, or public hosting.
- Build, rebuild, watch, or lifecycle management for source projects.
- Owning or relocating artifacts away from their home repos.
- Backward-compatible migration layers for early prototypes.

## Vocabulary

- **Project**: A stable owner namespace chosen by the user, usually a repo, tool, client, or workstream.
- **Artifact**: A routable static HTML directory under a project.
- **Manifest Root**: The directory that contains `.html-home.json`. It may be a git repo root, but git is not required.
- **Manifest**: A repo-local `.html-home.json` file that declares projects and artifacts.
- **Artifact Path**: The manifest-root-relative directory path declared by an artifact.
- **Artifact Base Directory**: The absolute directory resolved from `manifestRoot + artifact.path`.
- **Entry File**: The HTML file opened for an artifact, usually `index.html`.
- **Local State**: The persistent `state.json` file containing registrations, the last-known index, and diagnostics.
- **Server**: The single HTTP process that serves the start page, artifacts, assets, and read API.

## Hard Requirements

- Use one HTTP process and one configured port.
- Do not introduce per-project temporary servers.
- Bind to localhost by default. Add explicit security before LAN or public exposure.
- Keep artifacts filesystem-backed, repo-local, and easy to inspect.
- Treat home repos as the source of truth for live artifact content and manifest metadata.
- Treat `state.json` as authoritative only for registered manifest roots; indexed artifact metadata is last-known scan output.
- Do not copy artifacts into a central store by default.
- Use symlinks or direct path mapping as implementation details only; the manifest is the contract.
- Make local state writes atomic.
- Use stable, human-readable slugs for projects and artifacts.
- Justify every dependency before adding it.
- Replace obsolete code instead of keeping compatibility shims.

## Artifact Manifest Spec

Each source repo or local workspace can expose artifacts with `.html-home.json` at the manifest root.

```json
{
  "version": 1,
  "project": {
    "slug": "pdb",
    "title": "PDB"
  },
  "artifacts": [
    {
      "slug": "dashboard",
      "title": "Dashboard",
      "path": "artifacts/dashboard",
      "entry": "index.html",
      "tags": ["debug", "local"]
    }
  ]
}
```

Manifest rules:

- `version` is required and starts at `1`.
- `project.slug` is the stable URL namespace.
- Artifact `slug` values are stable URL IDs within a project.
- Artifact `path` is relative to the manifest root.
- Artifact `path` must resolve to a directory in v1.
- Artifact `entry` defaults to `index.html`.
- `tags` are metadata for filtering only.
- No SPA fallback or implicit nested `index.html` serving in v1.
- v1 artifacts are directories containing the entry file. A single-file artifact should use a directory path and set `entry` to the file name.
- Runtime assets should live inside the artifact base directory.
- v1 supports relative asset URLs. Root-relative URLs such as `/assets/app.js` are not rewritten.
- If an artifact needs repo data, place or symlink that data into the artifact base directory. Symlinks must resolve inside the artifact base directory in v1.
- Unknown top-level and artifact fields are rejected in v1.
- Project, artifact, and tag slugs use `^[a-z0-9]+(?:-[a-z0-9]+)*$`, maximum 64 characters.

## Source And Sync Model

Live mode serves repo-local artifacts. The server indexes manifests into `state.json` and resolves artifact paths at request time.

Initial commands:

- `register <path>` records a manifest root containing `.html-home.json`.
- `rescan` rereads registered manifests and updates local state.
- `unregister <path>` removes a manifest root registration.
- `list` prints indexed projects and artifacts.
- `doctor` reports missing manifests, missing entry files, duplicate slugs, and broken paths.
- `serve` runs the one foreground localhost HTTP process.

The server loads `state.json` fresh per request in v1. Missing, malformed, or unreadable state should not prevent startup; show an empty index with diagnostics instead. Use `Cache-Control: no-store` for v1 responses.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ratacat/html-home](https://github.com/ratacat/html-home) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
