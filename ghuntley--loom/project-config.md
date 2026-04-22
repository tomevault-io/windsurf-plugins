---
trigger: always_on
description: **IMPORTANT:** Before implementing any feature, consult the specifications in `specs/README.md`.
---

# Loom Agent Guidelines

## Specifications

**IMPORTANT:** Before implementing any feature, consult the specifications in `specs/README.md`.

- **Assume NOT implemented.** Many specs describe planned features that may not yet exist in the codebase.
- **Check the codebase first.** Before concluding something is or isn't implemented, search the actual code. Specs describe intent; code describes reality.
- **Use specs as guidance.** When implementing a feature, follow the design patterns, types, and architecture defined in the relevant spec.
- **Spec index:** `specs/README.md` lists all specifications organized by category (core, LLM, security, etc.).

## Commands

### Building with Nix (Preferred)
Use cargo2nix for reproducible builds with per-crate caching. Much faster on incremental changes.

- **Build CLI:** `nix build .#loom-cli-c2n`
- **Build server:** `nix build .#loom-server-c2n`
- **Build any crate:** `nix build .#<crate-name>-c2n` (e.g., `nix build .#loom-common-http-c2n`)
- **Build images:** `nix build .#weaver-image` or `nix build .#loom-server-image`
- **Update Cargo.nix:** `cargo2nix-update` (after modifying Cargo.lock)

### Building with Cargo (Development)
Use cargo for quick iteration during development. Slower than nix on clean builds.

- **Build:** `cargo build --workspace`
- **Test all:** `cargo test --workspace`
- **Test single:** `cargo test -p loom-<crate> <test_name>` (e.g., `cargo test -p loom-core test_agent`)
- **Lint:** `cargo clippy --workspace -- -D warnings`
- **Format:** `cargo fmt --all`
- **Check all:** `make check` (format + lint + build + test)

### Web
- **Web dev:** `cd web/loom-web && pnpm dev`
- **Web test:** `pnpm test`

### cargo2nix Workflow
When you modify `Cargo.toml` or `Cargo.lock`:
1. Run `cargo2nix-update` to regenerate `Cargo.nix`
2. Commit `Cargo.nix` along with your changes
3. The nix build will use the updated dependency graph

## Deployment
Deployments happen automatically via `git push` to the `trunk` branch. The production server runs NixOS with auto-update enabled. The update service runs every 10 seconds, checks for new commits, and rebuilds if needed.

- **Deploy:** `git push origin trunk`
- **Check status:** `sudo systemctl status nixos-auto-update.service`
- **View logs:** `sudo journalctl -u nixos-auto-update.service -f` (follow) or `-n 100` (last 100 lines)
- **Check deployed revision:** `cat /var/lib/nixos-auto-update/deployed-revision`
- **Force rebuild:** Delete the deployed revision file and restart: `sudo rm /var/lib/nixos-auto-update/deployed-revision && sudo systemctl start nixos-auto-update.service`
- **Service state:** `activating` = deploying, `active (exited)` = completed successfully
- **Repo location on server:** `/var/lib/depot`

### Verifying Deployment
0. IMPORTANT: You are running on the machine and can check status without ssh (just use sudo)
1. Check the deployed revision matches your commit: `cat /var/lib/nixos-auto-update/deployed-revision`
2. Check loom-server was restarted: `sudo systemctl status loom-server` (look at start time)
3. Check health endpoint: `curl -s https://loom.ghuntley.com/health | jq .`

## Database Migrations

**All database migrations go in `crates/loom-server/migrations/`** as numbered SQL files.

- **Convention:** `NNN_description.sql` (e.g., `020_scm_repos.sql`)
- **DO NOT** put inline SQL migrations in other crates (loom-server-scm, loom-thread, etc.)
- Migrations run automatically on server startup via `db/mod.rs`
- Check existing migrations for the next available number before creating new ones

### IMPORTANT: Force Rebuild After Adding Migrations

**cargo2nix doesn't track `include_str!` file changes.** When you add or modify migration files:

1. Run `cargo2nix-update` to regenerate `Cargo.nix` (this changes the hash and forces rebuild)
2. Commit `Cargo.nix` along with your migration changes
3. Without this step, the deployed binary will NOT include the new migration!

## Local Testing
Before deploying, test changes locally to verify behavior:

- **Run server on alternate port:** `LOOM_SERVER_PORT=9090 LOOM_SERVER_DB_PATH=/tmp/loom-test.db ./target/release/loom-server`
- **Dev mode (auto-auth):** Add `LOOM_SERVER_AUTH_DEV_MODE=1` for testing without real auth
- **Test against local:** `curl http://localhost:9090/health`
- **Run integration tests:** `cargo test -p loom-server <test_name>`

## Weaver Troubleshooting
Weavers are ephemeral K8s pods for running remote Loom REPL sessions.

### CLI Commands
- **Login first:** `loom --server-url https://loom.ghuntley.com login`
- **List weavers:** `loom --server-url https://loom.ghuntley.com weaver ps`
- **Create weaver:** `loom --server-url https://loom.ghuntley.com new --image <image>`
- **Attach:** `loom --server-url https://loom.ghuntley.com attach <weaver-id>`
- **Delete:** `loom --server-url https://loom.ghuntley.com weaver delete <weaver-id>`

### Kubernetes Debugging
Weavers run in the `loom-weavers` namespace:

- **List pods:** `sudo kubectl get pods -n loom-weavers`
- **Describe pod:** `sudo kubectl describe pod <pod-name> -n loom-weavers`
- **Pod logs:** `sudo kubectl logs <pod-name> -n loom-weavers`
- **Delete stuck pod:** `sudo kubectl delete pod <pod-name> -n loom-weavers`

### Server Logs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ghuntley/loom](https://github.com/ghuntley/loom) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
