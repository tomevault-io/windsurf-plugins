---
trigger: always_on
description: `skills-registry` is **two coordinated deliverables** shipped from a single repo:
---

`skills-registry` is **two coordinated deliverables** shipped from a single repo:

| Piece | Language | Distribution | Job |
|---|---|---|---|
| `skills-registry` (Go) | Go 1.24+ | GitHub Releases tarballs, installed by `install.sh` (`curl … \| sh`) | Charmbracelet TUI + headless commands. Bare invocation routes to wizard / hub / help. Subcommands: `bootstrap`, `list`, `search`, `get`, `sync`, `add`, `publish`, `remove`, `update`. All subcommands honor a persistent `--json` flag. |
| `skills-registry-mcp` (Python, hosted) | Python 3.10+ (FastMCP) | Docker image on Railway, served at `https://mcp.skills-registry.dev/mcp` | Streamable HTTP MCP server with **2 read-only tools** (`search_skills`, `get_skill`). All writes (`publish` / `sync` / `remove`) go through the Go CLI — the hosted server never mutates the user's repo. OAuth + GitHub App on first connect. Users never install this. |

- **Build (Python, maintainer-only):** `hatchling` (PEP 517) with a static `version = "0.0.0+server"` in `pyproject.toml`. The server is never published to PyPI, never tagged, and Railway redeploys on every push to `main`, so there's no semver to derive. The wheel exists only to provide the `skills-registry-mcp` entry point inside the Docker image.
- **Package manager (Python):** `uv`
- **Test runner (Python):** `pytest` with `pytest-cov`
- **Lint/Format (Python):** `ruff`
- **Build/Test (Go):** stdlib (`go build`, `go test`, `go vet`) + `staticcheck` + `deadcode` for dead-code / unused-symbol detection
- **TUI library:** Charmbracelet (bubbletea + lipgloss + bubbles + cobra)
- **MCP transport:** Streamable HTTP via FastMCP 3.x (the hosted server). stdio is no longer supported. Codex is supported via its TOML config (`[mcp_servers.skills-registry]` with `url = "…"` in `~/.codex/config.toml`) — it speaks the same Streamable HTTP.
- **Network surface:**
  - **Hosted MCP server (Python):** every GitHub call uses an installation-scoped GitHub App token. No `gh`, no `git`, no SSH, no user shell state. The container has only what its Dockerfile installs.
  - **CLI bootstrap (Go):** the bulk initial import (wizard step 4) uses **`git push` over HTTPS** (single push for the whole tree) because per-file `POST /git/blobs` trips GitHub's secondary rate limit on registries with dozens of skills. Auth wired through `gh auth setup-git`.
  - **CLI reads (Go):** `list`, `get`, `sync` and the hub read from a **local shallow-clone mirror** at `~/.cache/skills-mcp/mirror/<owner>/<repo>/` (see `cli/internal/registry/mirror.go`). Created with `git clone --depth=1`, fast-forwarded with `git fetch --depth=1` + `git reset --hard FETCH_HEAD`. The previous `1 + N` sequential `gh api` walk dropped from ~25 s to ~0.8 s warm on a 91-skill registry. `SKILLS_MIRROR_DISABLE=1` (or no `git` on PATH) forces the original gh-api path.
  - **CLI writes (Go):** single-skill `publish` and `remove` go through `gh api` — 1–10 files, well under the rate limit, and the atomic Git Data API path keeps strict-ordering / null-SHA semantics intact.
  - **Installer (`install.sh`):** the only one-shot `curl … | sh` surface. POSIX `sh`, detects OS/arch, downloads the matching tarball, drops the binary into `~/.local/bin/skills-registry`. Never touches Python.
  - **npm wrapper (`npm/`):** a thin Node launcher published as `skills-registry` (`npx skills-registry`). Ships no binary of its own — its `postinstall` (with a lazy first-run fallback in `run.js` to survive `--ignore-scripts`) detects `process.platform`/`process.arch`, downloads the matching GitHub Release tarball/zip, extracts the single binary via the host's `tar`/`Expand-Archive`, and execs it with inherited stdio. Package version maps 1:1 to the release tag. Never touches Python; never bundles or commits the binary.

---

## Repository Layout

```text
install.sh               # POSIX `curl | sh` installer — the user-facing entry point.
                         # Downloads the matching skills-registry tarball from GitHub Releases.

npm/                     # Thin npm wrapper published as `skills-registry` (`npx skills-registry`).
                         # postinstall/run.js download + exec the matching release binary; ships no binary.
  package.json           # bin → run.js; postinstall → install.js; version stamped to the release tag in CI
  lib/binary.js          # platform→asset mapping, download/extract helpers (no runtime deps)
  install.js             # postinstall hook (non-fatal; defers to first-run fallback)
  run.js                 # bin launcher: ensure binary present, exec with inherited stdio + exit-code passthrough

infa-not-for-users/      # Maintainer-only. Hosted MCP server source + Docker/Railway config.
  skills_mcp/            # Python package (no `src/` layout — packages = ["skills_mcp"] in pyproject.toml)
    __init__.py          # __version__ resolved from installed package metadata
    remote_server.py     # `skills-registry-mcp` — FastMCP build_server() + main(); registers search_skills + get_skill, wires middleware stack + mask_error_details
    middleware.py        # Production middleware stack: ErrorHandling → RateLimiting (per-user `sub` token bucket) → StructuredLogging

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nikships/skills-registry](https://github.com/nikships/skills-registry) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
