---
trigger: always_on
description: **The consumer is the mock STS** (`rcbj/iya-sts`, the `sts/` submodule here).
---

# embedded/ — the debugger as the mock STS embeds it

**The consumer is the mock STS** (`rcbj/iya-sts`, the `sts/` submodule here).
It serves this debugger's UI as STATIC FILES from a listener of its own — its
own origin, e.g. `https://host:8444/` — behind an OIDC sign-in, and proxies
everything under `https://host:8444/api/*`, after checking an access token and
with **`/api` stripped**, to this debugger's api, which it runs as a **forked
child process listening on a unix socket, plain HTTP**. All authentication
lives in the mock STS; the api has none and needs none, because its only peer
is that proxy.

**Nothing here changes the standalone debugger.** client:3000 / api:4000, the
compose stacks and the idptools.com static build behave exactly as before:
every change is additive and off unless `DEPLOYMENT=embedded` or one of the
`DEBUGGER_*` variables below is set. `tests/embedded_deployment.js` and the
allow-list section of `tests/api_ssrf_guard.js` hold this side; a static build
with and without these changes was compared file by file when they landed and
was identical.

This file is the CONTRACT between the two repositories. Changing any name,
path or variable in it is a change to the mock STS too.

| File | What it is |
|---|---|
| `build.sh` | `embedded/build.sh --out <dir>` — writes the tree below, WITHOUT docker |
| `Dockerfile` | the same tree in a `FROM scratch` image, by running `build.sh` |

## A. The tree

```
<OUT>/ui/            client/build.js, DEPLOYMENT=embedded, CONFIG_FILE=./env/embedded.js
<OUT>/api/           a runnable api: `cd <OUT>/api && node server.js`
<OUT>/common/        tls_listener.js and spiffe/ — what the api requires as ../common/...
<OUT>/version.json   the build's M.N.O (the same record as api/version.json)
```

`<OUT>/api` is what `api/Dockerfile` stages at `/usr/src/app`: every file under
`api/` (its `env/embedded.js` included), production `node_modules` with the
`ldapjs` link to `node-ldapjs` inside the package root, `data.js` and
`xmldsig.js` from `common/`, the repo-root `VERSION`, the client's `version.js`
and the stamped `version.json`. `<OUT>/common` is that Dockerfile's two COPYs
into `/usr/src/common` and nothing more. **When `api/Dockerfile` stages
something new beside the api, `build.sh` needs the same line**, or the image
works and the embedded api dies at startup with `Cannot find module`.

**`build.sh` touches nothing in the checkout.** It copies `VERSION`, `client/`,
`api/` and `common/` to a temporary directory (minus `node_modules`, build
output and the per-build files the images write), runs `npm ci`, `build.js`,
`npm install --omit=dev` and the version stamp THERE, copies the result to
`<OUT>` and deletes the temporary directory however it exits. The reason is
that all four of those steps write into the tree they run in, and several
stacks run from one checkout concurrently. It copies what is ON DISK, so
uncommitted work is in the build, as it would be in a `docker build`. It sets
`BUILD_NUMBER` once so the UI's and the api's `version.json` agree.

**`Dockerfile`** — build context is the repo root, classic builder only (no
BuildKit on the machines this runs on, so no `# syntax`, `--mount` or
`--build-context`), Node 24.16.0 installed as the other images do, and a final
`FROM scratch` stage holding exactly `/debugger/ui`, `/debugger/api`,
`/debugger/common` and `/debugger/version.json`. Tag it
`rcbj/id-proto-debugger-embedded:<tag>`; the mock STS's Dockerfile does
`COPY --from=rcbj/id-proto-debugger-embedded:<tag> /debugger/ …`. Pass
`--build-arg GIT_COMMIT=…` — there is no `.git` in the context to ask.

## B. The api child's environment

The mock STS forks `<OUT>/api/server.js` with cwd `<OUT>/api` and sets:

| Variable | Read by | Effect |
|---|---|---|
| `CONFIG_FILE` | `api/server.js` | absolute path of `<OUT>/api/env/embedded.js` |
| `DEBUGGER_LISTEN_SOCKET` | `common/tls_listener.js` | bind plain HTTP on this unix socket (see below) |
| `DEBUGGER_UI_URL` | `api/env/embedded.js` | the debugger origin, no trailing slash; `uiUrl` = it, `apiUrl` = it + `/api`, `spEntityId` = it + `/saml/sp`, `acsUrl`/`sloUrl`/`wsfedAcsUrl` = apiUrl + `/samlacs`, `/samlslo`, `/wsfed` |
| `DEBUGGER_ALLOWED_ADDRESS_RANGES` | `api/env/embedded.js` → `api/ssrf_guard.js` | a JSON array of ranges; non-empty = allow-list mode |
| `DEBUGGER_BLOCK_PRIVATE_NETWORK_CALLS` | `api/env/embedded.js` | `"true"`/`"false"`, only without an allow-list; default true |
| `DEBUGGER_LOG_LEVEL` | `api/env/embedded.js` | default `info` |
| `NODE_EXTRA_CA_CERTS` | node | the anchor for the mock STS's own certificate |

**The socket.** `tls_listener.listen()` takes `options.socketPath` or
`DEBUGGER_LISTEN_SOCKET`, and a socket OUTRANKS `https` and `TLS_ENABLED` —
it is always plain HTTP, and `materialFor()` is never asked, so a config left
at `https: true` with no certificate cannot stop it starting. It removes a
stale SOCKET at the path first (and refuses anything else there — a regular
file is somebody's), binds with the umask narrowed, chmods 0600, and when
`process.send` exists sends `{ type: 'debugger-api-listening', socket }` once.
`serverCertificate()` is null. The filesystem mode IS the access control, there
being no address to firewall.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rcbj/id-proto-debugger](https://github.com/rcbj/id-proto-debugger) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-01 -->
