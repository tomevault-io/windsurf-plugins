---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

- Build the only binary: `go build -o freesbc ./cmd/freesbc` (`/freesbc` is gitignored). Set the version with `-ldflags "-X main.version=<v>"` (default `dev`).
- Release: `.github/workflows/release.yml` runs on pushed `v*` tags, or by hand with a `tag` input to backfill one. It builds `CGO_ENABLED=0 -trimpath` binaries for linux/darwin x amd64/arm64, packages `freesbc_<tag>_<os>_<arch>.tar.gz` (binary, README, LICENSE, `freesbc.example.yaml`) with `SHA256SUMS`, runs `check` on the linux/amd64 build, and uploads to the release with `gh`.
- Vet: `go vet ./...`. There is no linter config or Makefile; `gofmt -l .` and `go mod tidy -diff` are clean and CI keeps them so.
- CI (`.github/workflows/ci.yml`, on push to main and PRs): gofmt, `go mod tidy -diff`, vet, build, `check` on `freesbc.example.yaml`, `go test -race ./...`, and govulncheck (pinned v1.8.0, `GOTOOLCHAIN=local`). `soak.yml` runs nightly and on demand: the suite under `-race -count=N` and each `Fuzz*` target for `-fuzztime`. A new fuzz target must be added to its matrix.
- Tests (as the README runs them): `go test ./... -race`. One package: `go test -race ./internal/config`. One test: `go test ./internal/app -run '^TestCheckExamples$' -count=1 -v`.
- No build tags. The edge suite is mostly integration tests over real loopback sockets (real sipgo transports and RTP sockets), not mocks.
- CLI: `freesbc run|check [-c file]`. `-c` defaults to `freesbc.yaml`; a positional argument is a usage error (exit 2), so always pass `-c`.
- Example config: `./freesbc check -c freesbc.example.yaml` passes (`TestCheckExamples` guards this). `check` only parses and validates: it does not test that addresses are local, open cert files or bind sockets. `run -c freesbc.example.yaml` exits 1 because `203.0.113.7` and `10.77.0.2` are not local addresses, so adapt them before `run`. The local copy `/freesbc.yaml` is gitignored.
- Opt-in live FreeSWITCH interop. It skips unless `FREESBC_FS_INTEROP=1` and shells out to `/usr/local/freeswitch/bin/fs_cli`, so run it on the FreeSWITCH host:
  `FREESBC_FS_INTEROP=1 FREESBC_FS_ADDR=<fs-ip>:5060 FREESBC_FS_LOCAL=<this-host-ip> FREESBC_FS_USER=1000 FREESBC_FS_PASS=<pw> go test ./internal/edge/ -run TestFreeSWITCH -v`

## Architecture

One process, one YAML file, one SIP plane: the **edge** (`internal/edge`), a stateful SIP proxy (not a B2BUA) between public endpoints and a SIP switch (FreeSWITCH or Asterisk) on a private LAN. There are three kinds of far end: registered clients, carriers, and the switch. Every key is documented in `docs/config.md`.

**Scope rule.** FreeSBC does only what is necessary to make one switch on a private LAN safely reachable from the public internet. Anything the switch already does well, FreeSBC does not do. A feature is in scope only if it passes one of two questions: (1) only the edge can see or do it (the public wire before rewrite, TLS/WSS/WebRTC termination, NAT and latching, topology hiding, media anchoring, public-side admission, or FreeSBC's own state: sockets, ports, bindings, bans, reloads), or (2) the switch cannot do it well from behind the edge. Everything else stays on the switch; FreeSBC's contribution is a documented switch-side example, not a feature. Check a proposed feature against this before building it.

- Addresses are written once: `public.ip` (advertised), `public.bind` (local bind, default `public.ip`; differs only behind 1:1 NAT), `private.ip` (bind = advertised). Wildcards are rejected. `run` checks both bind IPs are assigned to a local interface; `check` does not.
- Public sockets (`edge.listen.udp|ws|wss`) bind on `public.bind`. The private side is one fixed UDP socket, `private.ip:5060` (`config.PrivateSIPPort`), carrying all SIP to and from the switch. The switch is `edge.switch` (literal `IP:port`, one entry or a hash-user pool) and is trusted only on that socket.
- Call-ID, tags and CSeq pass through, and the proxy stays on the path with RFC 5658 double Record-Route. The switch is the registrar: REGISTER and its digest are forwarded verbatim and FreeSBC holds no credentials, for clients or carriers.
- Dependencies run one way: `cmd/freesbc -> app -> {edge, admin}`. `edge` imports only `config`, `media`, `shield`, `sip` and `sip/sdp`. `config`, `media`, `sip` and `sip/sdp` import nothing from this module. `admin` reads the plane only through `admin.Deps` closures built in `app`.

### Signaling and media
- Signaling uses `emiago/sipgo` v1.4.3 for transactions and retransmission, with one user agent, listener set and `shield.Shield` per process. `internal/sip` holds shared RFC 3261 primitives with no policy.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rasonyang/freesbc](https://github.com/rasonyang/freesbc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
