---
trigger: always_on
description: `objgitd` is a single-binary Git server that stores repositories as objects in a Tigris bucket instead of on a local filesystem. It speaks three transports against the same backend:
---

# Repository guidelines

`objgitd` is a single-binary Git server that stores repositories as objects in a Tigris bucket instead of on a local filesystem. It speaks three transports against the same backend:

- **Smart HTTP** (`-http-bind`, default `:8080`) — primary transport. Carries an HTTP Basic credential into the auth seam.
- **git://** (`-git-bind`, default `:9418`) — unauthenticated TCP, opt-in.
- **SSH** (`-ssh-bind`, default off) — public-key transport, opt-in. Host key persisted in the bucket.

All three funnel authorization through one pluggable `internal/auth.Authorizer` (see [The auth seam](#the-auth-seam-internalauth)).

A fourth listener serves **Prometheus metrics** at `/metrics` (`-metrics-bind`, default `:9090`, empty disables) — see [Metrics](#metrics-internalmetrics).

Module path: `tangled.org/xeiaso.net/objgit`. Go 1.26.

## Commands

```text
go build ./...                  # build everything
go build -o objgitd ./cmd/objgitd
go test ./...                   # full test suite
go test ./cmd/objgitd/...       # protocol tests (require `git` on PATH; skipped otherwise)
go test -run TestSmartHTTP ./cmd/objgitd/...

# Run locally. Flags can also come from env via flagenv (UPPER_SNAKE of the flag name).
# A .env file in CWD is auto-loaded by godotenv.
./objgitd -bucket $BUCKET -http-bind :8080 -allow-push
./objgitd -bucket $BUCKET -ssh-bind :2222 -allow-push   # git clone ssh://git@host:2222/repo.git
```

SSH tests additionally need `ssh` and `ssh-keygen` on PATH (skipped otherwise);
run them with `go test -run TestSSH ./cmd/objgitd/...`.

`flagenv` maps `-allow-push` → `ALLOW_PUSH`, `-bucket` → `BUCKET`, etc. Tigris client credentials come from the standard AWS SDK chain (`AWS_PROFILE` etc.).

## Architecture

### The `daemon` is the shared backend

`cmd/objgitd/main.go` constructs one `*daemon` holding `(fs billy.Filesystem, loader transport.Loader, authz auth.Authorizer)` (plus the hooks fields) and serves it through all three transports concurrently under an `errgroup`. Repository resolution, authorization, and **create-on-first-push** all live on `*daemon` (`loadOrInit` in `git_protocol.go`) so every transport behaves identically.

- `cmd/objgitd/git_protocol.go` — git:// TCP server: `Serve` → `handle` decodes a `packp.GitProtoRequest`, then dispatches to `transport.UploadPack` / `UploadArchive` / `ReceivePack`. Also holds the shared `operationFor(service)` helper (receive-pack → `auth.Write`, else `auth.Read`).
- `cmd/objgitd/http.go` — `*daemon` implements `http.Handler` directly. Dispatch is by **URL suffix** (`/info/refs`, `/git-upload-pack`, `/git-receive-pack`) because repo paths are variable-depth and `http.ServeMux` wildcards can't capture a prefix before a fixed suffix. Smart-HTTP uses the same go-git server commands with `StatelessRPC: true` (and `AdvertiseRefs: true` for `GET /info/refs`).
- `cmd/objgitd/ssh.go` — SSH server (gliderlabs/ssh): `newSSHServer` builds the server and host key; `handleSSH` is the per-session dispatcher, a sibling of `handle` (see [Git over SSH](#git-over-ssh-sshgo)).

### Two subtle protocol points

1. **`writePack` in `receivepack.go`** stores the incoming pack whole on every transport. go-git's default `PackfileWriter` path (`WritePackfileToObjectStorage` → `io.CopyBufferPool`) copies until `io.EOF`, which deadlocks over a persistent git:// / SSH socket (the client holds the connection open awaiting report-status). Instead of relying on EOF, `writePack` drives a `packfile.Scanner` over an `io.TeeReader(rd, packWriter)`: the scanner knows the pack's end from its own framing (header object count + trailer checksum) and stops there, while the tee mirrors exactly those bytes into the `PackfileWriter`. The result is one `.pack` + one `.idx` per push — no per-object loose writes (which on S3 cost a `HeadObject` dedup `Lstat` **and** a `PutObject` each). This replaced the old `streamingStorer` hack (which hid `PackfileWriter` to force the loose-object `Parser.Parse` path on git:// / SSH); all three transports now share `writePack`.

2. **No-op closers everywhere.** `transport.UploadPack`/`ReceivePack` call `Close` on the reader (and sometimes the writer) between negotiation rounds. The git:// socket can't survive that, and the HTTP `ResponseWriter` doesn't implement `Close`. Wrap with `io.NopCloser` (reader) and `ioutil.WriteNopCloser` from `go-git/v6/utils/ioutil` (writer).

**SSH shares both gotchas with git://**, not HTTP: an `ssh.Session` is a persistent bidirectional stream, so `handleSSH` wraps the session in the same no-op closers and relies on the same Scanner-bounded `writePack` (gotcha #1) instead of the request-body EOF that HTTP enjoys.

> **s3fs `ReadAt` contract.** Because packs are now read back via go-git's `packfile.FSObject` (which probes a packed object's handle with a 1-byte `ReadAt` and reopens the pack on `os.ErrClosed`), `s3fs`'s read file (`internal/s3fs/file.go`) returns `os.ErrClosed` from `Read`/`ReadAt`/`Seek` once closed rather than dereferencing its niled `*bytes.Reader`. A post-receive hook reading the just-pushed commit hits exactly this path.

### The auth seam (`internal/auth`)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tigrisdata/objgit](https://github.com/tigrisdata/objgit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
