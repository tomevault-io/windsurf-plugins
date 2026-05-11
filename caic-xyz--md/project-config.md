---
trigger: always_on
description: A file to [guide coding agents](https://agents.md/).
---

# Agent development guide

A file to [guide coding agents](https://agents.md/).

## Requirements

- Make sure the code passes shellcheck after every change. Then format with `shfmt -l -w $script_name`
- Update this file (AGENTS.md) everytime you make a change that affects this project's requirements.
- Update rsc/user/home/user/src/AGENTS.md everytime you make a change that affects the agent inside the container.
- **Glob/find tools may skip dotfiles by default.** The `rsc/` tree contains important config under dot-directories (e.g. `rsc/user/home/user/.config/git/config`). Use Grep or explicit dot-inclusive patterns when searching for files under `rsc/`.
- When adding a new setup script in `rsc/root/root/setup/`, add a corresponding `RUN` command to `rsc/root/Dockerfile`. When adding a new setup script in `rsc/user/home/user/setup/`, add a corresponding `RUN` command to `rsc/user/Dockerfile`.
- No tests should be written for Python or shell script changes.
- **NEVER run `go build ./cmd/md/` without `-o`** — the repo root contains a Python script named `md` and `go build` will overwrite it. Always use `go build -o /tmp/md-test ./cmd/md/` or similar.
- For Go code changes, ensure code passes `go test ./...`, `go vet ./...`, and `golangci-lint run ./...`.
- For Python code changes, ensure code passes `pylint` and `ruff` checks as defined in `.github/workflows/docker-build-user.yml`
- When adding new tools to the system, they must also be added to `rsc/user/home/user/setup/generate_version_report.sh` to ensure they appear in version reports. The script generates `/home/user/src/tool_versions.md` which is used in release notes and build reports

## md Tool: Image Build and Cache Injection

### Image hierarchy

- **`md-root-local`** — root image built locally from `rsc/root/Dockerfile` via `md build-image` (first step).
- **`md-user-local`** — user image built locally from `rsc/user/Dockerfile` on top of `md-root-local` via `md build-image` (second step). Used as base when `--image md-user-local` is passed.
- **`ghcr.io/caic-xyz/md-root:latest`** — remote root image with system packages. Rebuilt infrequently (when root setup scripts change). Built by `docker-build-root.yml`.
- **`ghcr.io/caic-xyz/md-user:latest`** (default) or any `--image`/`--tag` variant — remote user image with Go, Node, Rust, etc. Rebuilt weekly. Built by `docker-build-user.yml` on top of `md-root`.
- **`md-specialized-<hash>`** — specialized per-user image built on top of the chosen base via a generated Dockerfile + `docker build`. A Dockerfile is created at runtime with `COPY --chown` for SSH keys and `COPY --from=<named-context> --chown` for cache directories, then built with `--no-cache --pull=never --build-context cache-<name>=<hostpath>`. This approach was chosen over `docker create`/`cp`/`commit` (slower: `docker cp` uses API round-trips vs COPY's storage-driver-level tar streaming, and requires starting the container for permission fixes) and over a static Dockerfile (cannot adapt to dynamic cache sets). Built automatically by `md start` and `md run` when needed. The image name includes a 32-hex-char hash of (base image, active cache key) so that different base images or cache sets get distinct images without clobbering each other. Computed by `userImageName()` in `docker.go`.

### When the user image is rebuilt

`imageBuildNeeded` (`docker.go`) returns `true` (triggering a rebuild) when any of the following change:
1. `md.base_digest` label missing/empty, or differs from the current base image digest.
2. For remote base images: registry has a newer version than the local copy.
3. `md.context_sha` label differs from the SHA of the SSH keys.
4. `md.cache_key` label differs from `cacheSpecKey` of the **active** caches (those whose host directories currently exist).

### Cache injection

`md start` and `md run` bake host cache directories into the user image at build time via `COPY --from=<name>` in the Dockerfile. This avoids slow cold-start downloads inside the container.

**Default behaviour**: all `WellKnownCaches` entries are included. Caches whose host directory does not exist are silently skipped (no rebuild triggered for missing dirs).

**CLI flags** (on both `md start` and `md run`):
- `--no-cache <name>` — exclude a specific well-known cache (repeatable).
- `--no-caches` — disable all default caches; use `--cache` to add back specific ones.
- `--cache <spec>` — add a well-known name (re-adds when used with `--no-caches`) or a custom `host:container[:ro]` path.

**Well-known cache names** (defined in `WellKnownCaches`, `client.go`): android-keys, bun, cargo, go-mod, gradle, maven, npm, pip, pnpm, uv.

**Shallow caches**: setting `Shallow: true` on a `CacheMount` copies only top-level files from the host directory, ignoring subdirectories. This is useful for directories like `~/.android` where only a few files (debug.keystore, adbkey) are needed but subdirectories (avd/, cache/) are large and unwanted. The generated Dockerfile emits one `COPY` per file instead of `COPY . <dest>/`. If no top-level files exist, the cache is skipped.

**Adding a new well-known cache**: add an entry to `WellKnownCaches` in `client.go`. No other changes needed — it is automatically picked up by `resolveCaches` and the flag help text.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caic-xyz/md](https://github.com/caic-xyz/md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
