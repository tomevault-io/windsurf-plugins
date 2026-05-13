---
trigger: always_on
description: Go CLI for the RETYC platform. Module: `github.com/retyc/retyc-cli`
---

# CLAUDE.md — retyc-cli

## Project

Go CLI for the RETYC platform. Module: `github.com/retyc/retyc-cli`

## Structure

```
main.go                        # Entry point — calls cmd.Execute()
cmd/
  root.go                      # cobra root, --config / --insecure / --debug flags, viper init
  auth.go                      # auth login / logout / status + newHTTPClient + debugTransport
  common.go                    # Shared helpers: constants, newAPIClient, resolveUserIdentity,
                               #   newTransferBar, uploadChunks, downloadChunks
  transfer.go                  # transfer ls/info/create/download/enable/disable
  dataroom.go                  # dataroom commands (ls, cp, mv, rm, mkdir, create, info, user)
  version.go                   # version command (shows version + build mode)
internal/
  auth/oidc.go                  # DeviceFlow, Refresh, GetValidToken
  api/
    client.go                   # Authenticated REST client (oauth2 transport)
                                #   Get, Post, Put, Patch, Delete, PostMultipartChunk, GetBytes
    login.go                    # FetchOIDCConfig (unauthenticated, GET /login/config/public)
    transfer.go                 # Transfer types + ListTransfers, GetTransferDetails,
                                #   ListFiles, CreateShare, CreateFile, UploadChunk,
                                #   CompleteTransfer, DisableTransfer, EnableTransfer
    dataroom.go                 # Dataroom types + all dataroom API methods
    user.go                     # UserKey type + GetActiveKey
  config/
    config.go                   # Structs, SetDefaults(), Load(), token persistence
    paths_dev.go                # configDir() + defaultAPIBaseURL for dev
    paths_prod.go               # configDir() + defaultAPIBaseURL for prod
  crypto/age.go                 # AGE encrypt/decrypt helpers (PQ-only, see below)
  keyring/keyring.go            # Linux kernel session keyring cache (TTL-based)
Dockerfile                      # Multi-stage scratch image (golang:1.24 builder → scratch)
.dockerignore
.github/workflows/ci.yml        # CI + release workflow
```

## Build modes

Two mutually exclusive build tags control config location, `BuildMode` constant, and default API URL.

| Command | Tag | Config dir | API default | `retyc version` |
|---|---|---|---|---|
| `go build .` | `!prod` (default) | `.retyc/` (CWD) | `https://api.triplesfer.traefik.me` | `x.y.z (dev build)` |
| `go build -tags prod .` | `prod` | `~/.config/retyc/` | `https://api.retyc.com` | `x.y.z (prod build)` |

Override config dir at runtime: `RETYC_CONFIG_DIR=/some/path retyc ...`

`defaultAPIBaseURL` is defined in `paths_dev.go` / `paths_prod.go` (not in `config.go`).

## Version injection

`cmd.Version` is set via ldflags at build time:

```bash
go build -tags prod -ldflags "-X github.com/retyc/retyc-cli/cmd.Version=v1.2.3" .
```

Default value is `"dev"`. CI injects `github.ref_name` on tag pushes.

## Docker

Multi-stage scratch image — builder `golang:1.24`, final `scratch`:
- `CGO_ENABLED=0` static binary, `-tags prod`, ldflags version injection via `ARG VERSION`
- Copies only: binary, CA certs, `/etc/passwd`, `/home/retyc` (with `.config/retyc/` pre-created)
- Non-root user `retyc` (uid 1000), `VOLUME ["/home/retyc/.config/retyc"]`

```bash
docker build --build-arg VERSION=v1.2.3 -t retyc-cli:v1.2.3 .
docker run -it --rm -v retyc-config:/home/retyc/.config/retyc retyc-cli:v1.2.3 auth login
```

## Key defaults (API)

Registered via `viper.SetDefault` in `SetDefaults()` in `internal/config/config.go`.
All overridable from `~/.config/retyc/config.yaml` (prod) or `.retyc/config.yaml` (dev).

- Device auth URL: `.../protocol/openid-connect/auth/device`
- API base URL: per build mode (see above)
- Keyring: enabled by default, TTL 3600s (configurable via `keyring.enabled` / `keyring.ttl`)

## Persistent flags (root)

| Flag | Short | Default | Description |
|---|---|---|---|
| `--insecure` | `-k` | false | Skip TLS verification (self-signed certs) |
| `--debug` | `-d` | false | Print all HTTP requests + raw responses to stderr |
| `--config` | | auto | Override config file path |

`--debug` covers **all** HTTP traffic: API calls (via `api.Client.do` / `GetBytes`) and unauthenticated calls (`FetchOIDCConfig`, device flow, token refresh) via `debugTransport` wrapping the `*http.Client` RoundTripper in `newHTTPClient`.

Format: `> METHOD URL` then `< STATUS` + pretty-printed JSON body (or `(N bytes, binary)` for binary).

## TLS self-signed certificates

Use `--insecure` / `-k` (persistent flag on root) to skip TLS verification.
Applies to both the OIDC device flow HTTP client and the API REST client.
`InsecureSkipVerify` is annotated `#nosec G402` where used.

## Auth flow

`GetValidToken(ctx, cfg, httpClient)` in `internal/auth/oidc.go` is the single
entry point for obtaining a valid token:
1. Loads stored token → return if valid
2. If expired + refresh token present → `Refresh()` → save → return
3. Otherwise → `ErrNoToken` or `ErrNoRefreshToken` (callers re-run device flow)

Tokens are stored in `<configDir>/token.json` with permissions `0600`.

## Crypto — AGE post-quantum only

All keys use **MLKEM768-X25519 hybrid** (post-quantum). No legacy X25519 support.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [retyc/retyc-cli](https://github.com/retyc/retyc-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
