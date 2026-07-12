---
trigger: always_on
description: Project conventions for AI-assisted development on vault-cortex — for Claude Code and other AI agents.
---

# AGENTS.md

Project conventions for AI-assisted development on vault-cortex — for Claude Code and other AI agents.

## What this project is

Remote MCP server exposing an Obsidian vault over HTTPS. One two-target
Dockerfile builds the `ghcr.io/aliasunder/vault-cortex` image: the `local`
target (`:latest`, the default stage) is tini + the MCP server alone; the
`remote` target (`:remote`) adds s6-overlay supervising both obsidian-sync
(bidirectional Obsidian Sync via the `obsidian-headless` npm CLI) and the MCP
server in a single container — s6 service definitions live in `rootfs/`, and
the init chain registers the initial Sync device under DEVICE_NAME. Both
processes run as UID 1000 (PUID/PGID-adjustable). Production runs the
`:remote` image on Lightsail as a single Compose service, fronted by API
Gateway with a smart Lambda authorizer (path-aware: OAuth endpoints pass
through, /mcp validates static token or JWT). IaC via SST v4.

The server provides vault CRUD, hybrid search (FTS5 keyword + sqlite-vec
vector + cross-encoder reranking via RRF fusion and position-aware score
blending), and the About Me/ memory layer. The Docker image uses Debian
slim (`node:24-slim`) because `onnxruntime-node` requires glibc.

All solutions must be portable — they can't rely on one-off manual fixes,
hardcoded paths, or user-specific configuration. If it works only on
the author's machine, it's not done.

Design for the Obsidian user. The end user is always an Obsidian user, so
anything that mirrors an Obsidian concept — backlinks, outgoing links, orphans,
the graph, tags, properties, daily notes — must match what Obsidian itself does.
At minimum, recognize every form Obsidian recognizes; behavior that is a strict
subset of Obsidian's is a bug, not a limitation. For link resolution
specifically, that means all of Obsidian's link styles (`[[wikilink]]`,
`[[wikilink|alias]]`, `[[wikilink#heading]]`, `![[embed]]`, `[md](path.md)`),
links in frontmatter properties (e.g. `related:`), and all three "New link
format" modes — shortest path, path from vault folder, and path from current
file (including relative `../` paths).

See [ARCHITECTURE.md](./ARCHITECTURE.md) for the full design.

## Structure

```text
sst.config.ts                          # SST v4 IaC (fully implemented)
package.json                           # single package, all deps
tsconfig.json                          # single config
server.json                            # MCP server registry manifest
Dockerfile                             # Two-target build: local (default) + remote
rootfs/                                # Container filesystem overlay (remote target)
  etc/s6-overlay/                      #   init chain + svc-obsidian-sync + svc-vault-mcp
  usr/local/bin/get-token              #   interactive Obsidian Sync token helper
docker-compose.yml                     # Lightsail: single vault-cortex:remote service
docker-compose.local.yml               # Contributor dev: builds from source
.env.example                           # template for Lightsail .env
templates/                             # Bootstrap templates for new vaults
  memory/                              #   About Me/ memory file templates
deploy/                                # End-user quickstart (no clone needed)
  local/                               #   vault-cortex:latest + bind-mounted vault
    README.md                          #     quickstart walkthrough
    docker-compose.yml                 #     just: docker compose up
    .env.example                       #     MCP_AUTH_TOKEN + VAULT_PATH
  remote/                              #   vault-cortex:remote + named volumes
    README.md                          #     quickstart walkthrough (VPS, HTTPS, etc.)
    docker-compose.yml                 #     just: docker compose up
    .env.example                       #     + OBSIDIAN_AUTH_TOKEN, VAULT_NAME, PUBLIC_URL
scripts/                               # Dev/ops helpers (not shipped in Docker)
  dev.ts                               # Deployment helper (subcommands for SSH, sync, etc.)
  sync-cli-templates.ts                # Copies deploy/ compose files into cli/templates/
cli/                                   # npx vault-cortex CLI (published as vault-cortex npm package)
  src/
    bin.ts                             # Entry point (version injection + run)
    main.ts                            # Top-level wiring (program + init + prompts + docker)
    program.ts                         # Commander program definition
    init.ts                            # Init command orchestration
    prompts.ts                         # Interactive prompt flow (mode, vault path, token)
    scaffold.ts                        # File generation (docker-compose.yml, .env)
    docker.ts                          # Container management (compose up, health-check wait)
    env.ts                             # Environment file handling (.env generation)
    token.ts                           # Secure token generation (openssl rand)
    vault.ts                           # Vault path validation
    node-version.ts                    # Node.js version compatibility check
    messages.ts                        # User-facing output formatting

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aliasunder/vault-cortex](https://github.com/aliasunder/vault-cortex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
