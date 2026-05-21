---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Development Commands

**IMPORTANT: This project uses iso for containerized builds and testing.**

### Building
- `make bin/miren` - Build the miren binary using hack/build.sh (includes version info)
- `make bin/miren-debug` - Build with debug symbols for debugging
- `make release` - Build release version using hack/build-release.sh

### Testing

- `make test` - Run all tests using iso (runs hack/test.sh in isolated container)
- `make test-serial` - Run all tests serially with `-p 1` (for debugging test interference)
- `make test-shell` - Run tests with interactive shell (set USESHELL=1)
- `make test-blackbox` - Run blackbox CLI tests (requires `make dev` running)
- `hack/it <gopkg>` - Run all tests in a package using iso
- `hack/run <gopkg> <testname>` - Run a single focused test using iso

### Development Environment

The dev environment uses **standalone mode** where miren manages its own containerd and buildkit internally, matching how it runs in production.

**Initial setup (once per worktree):**
- `make dev` - Start persistent dev environment, launch server, and open a shell (recommended)
- `make dev-start` - Start environment only (no server, no shell)

The dev environment automatically:
- Builds the miren binary and creates `/bin/m` symlink
- Generates auth config in `~/.config/miren/clientconfig.yaml`
- Prepares release directory with required binaries

When you run `make dev`, the server starts automatically in the background, so commands like `m app list` work immediately.

**Server lifecycle management:**

The miren server runs independently from your shell session:
- `make dev-server-start` - Start miren server (standalone mode)
- `make dev-server-stop` - Stop miren server
- `make dev-server-restart` - Restart server (useful after rebuilding)
- `make dev-server-status` - Check if server is running
- `make dev-server-logs` - Watch server logs

**Working in the persistent dev environment:**

The dev environment uses persistent containers, which means:
- The container and all services stay running between commands
- Each worktree gets its own isolated dev environment
- You can run commands from different terminals or LLM sessions
- The miren server runs independently and survives shell exits

Running commands:
- `make dev-shell` - Open an interactive shell
- `./hack/dev-exec <command>` - Run any command in the dev container
- Examples:
  - `./hack/dev-exec go test ./pkg/entity/...` - Run tests
  - `./hack/dev-exec m app list` - Use miren CLI
  - `./hack/dev-exec make bin/miren` - Rebuild binary inside dev container (then `make dev-server-restart`)

**Important**: The miren binary must be built **inside** the dev container (not on the host) so it has the correct architecture. Use `./hack/dev-exec make bin/miren` instead of `make bin/miren`.

**Exception**: `go generate` and doc generation (`hack/gen-command-docs`) need host tools like `jq` that aren't in the dev container. For these, build the binary on the host with `make bin/miren` and run `go generate` on the host as well.

**Managing the dev environment:**
- `make dev-stop` - Stop and remove the persistent dev container
- `make dev-restart` - Restart the dev environment (stop + start)
- `make dev-status` - Check the status of the dev environment

**Typical workflow:**
```bash
# Initial setup (once per worktree)
make dev                      # Starts environment, server, and gives you a shell

# Now you're in a shell with server running - try it:
m app list                    # Works immediately!

# Development iteration
vim path/to/code.go           # Edit code
./hack/dev-exec make bin/miren # Rebuild inside container
make dev-server-restart       # Bounce server with new code

# Debugging
make dev-server-logs          # Watch logs
make dev-server-status        # Check if running

# Multiple shells
make dev-shell                # Open another shell (from host)
./hack/dev-exec m app list    # One-off commands

# Cleanup
make dev-stop                 # Tear down environment
```

**Connecting to a local Miren Cloud instance:**

The dev environment can connect to a local copy of Miren Cloud for testing the full authentication and registration flow.

Prerequisites:
- Miren Cloud running locally (typically on `http://localhost:3001`)
- The `.iso/config.yml` includes `extra_hosts` configuration for host access:
  ```yaml
  extra_hosts:
    - "miren.host:host-gateway"
  ```
  This allows containers to reach the host machine via `miren.host` instead of `localhost`.

Workflow:
```bash
# 1. Start the dev environment
make dev

# 2. Login to your local cloud (inside dev shell or via dev-exec)
m login --url http://miren.host:3001
# Follow the URL to authenticate in your browser

# 3. Register the cluster with cloud
m register -u http://miren.host:3001 -n my-dev-cluster
# Approve the registration in the browser when prompted

# 4. Restart server to activate registration
make dev-server-restart

# 5. Verify the setup
m cluster list
m app list
```

**Important notes:**
- Use `miren.host:3001` (not `localhost:3001`) from inside the dev container

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mirendev/runtime](https://github.com/mirendev/runtime) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
