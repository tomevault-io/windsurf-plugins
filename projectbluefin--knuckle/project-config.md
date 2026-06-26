---
trigger: always_on
description: > **Audience:** AI coding agents (Claude Code, pi, Copilot, Cursor). Humans want `README.md`.
---

# knuckle — Agent Context

> **Audience:** AI coding agents (Claude Code, pi, Copilot, Cursor). Humans want `README.md`.
>
> **Bar:** CNCF-incubating rigor. Every change keeps `just ci` green, respects package
> boundaries, and preserves the safety invariants below.

**Read [`docs/SKILL.md`](docs/SKILL.md) before starting any task.** It routes you to the right skill file for PR review, releases, testlab, and CI work.

> **Before using any tool or library: look up its docs via Context7 first. Always.**
> Go stdlib, charm.sh (Bubble Tea, Lip Gloss, Huh), Flatcar Butane, cosign, govulncheck — every tool has live docs.
> Pattern: `resolve-library-id` → `get-library-docs` → implement → cite the section.
> Guessing, flag-hunting, and trial-and-error are banned. The docs exist. Read them.

---

## What This Repo Is

A TUI installer for [Flatcar Container Linux](https://www.flatcar.org/) and [FCOS](https://fedoraproject.org/coreos/), targeting bare-metal.
Built in Go on charm.sh (Bubble Tea v2, Lip Gloss v2, Huh v2). Assembles an Ignition config
and dispatches to `flatcar-install` (Flatcar) or `coreos-installer` (FCOS) via `DispatchingInstaller` — knuckle never writes partitions itself.

- **Module:** `github.com/projectbluefin/knuckle` (Go 1.26+)
- **License:** Apache-2.0
- **Status:** v0.7.0; full install → reboot → SSH verified end-to-end in QEMU; dual-arch ISOs (amd64 + arm64); cosign keyless signing on releases.

## v1 Supported Scope

- Architecture: x86_64 + ARM64
- Storage: single target disk (no RAID, LVM, LUKS)
- Networking: DHCP + simple static IPv4 only
- Sysexts: official Flatcar Bakery entries only (via GitHub Releases API)
- Config: guided TUI OR external Ignition URL passthrough (`Ctrl+A`) — mutually exclusive

Anything outside this list belongs in an issue, not a PR.

---

## PR Comment Policy

**One comment per PR event, max.** Combine all findings into a single comment. Never post a follow-up comment for a new observation — edit the existing one instead.

**Never duplicate GitHub UI state.** Do not post approval counts, merge queue status, or CI pass/fail summaries — GitHub already surfaces these natively in the PR timeline.

**Test reports: minimal.** Report what ran, pass/fail, and blockers only. No diff summaries. No tables unless comparing ≥3 divergent approaches that require a human decision.

**@ mentions in context only.** Only ping someone if asking them to do something specific. Always inside the combined comment — never as a standalone comment.

**When in doubt, don't post.** If the only thing to report is "tests pass", post nothing.

---

## Build / Test / Lint

```bash
just              # list all recipes
just ci           # tidy + gofmt + vet + lint + vuln + test-race + cover-check + build
just build        # GOOS=linux GOARCH=amd64 CGO_ENABLED=0 → bin/knuckle
just test         # go test ./...
just cover-check  # per-package coverage threshold gate
just headless-test       # config-gen e2e (CI gate, runs on host)
just vm                  # install in QEMU → auto-boots installed system after
just vm-e2e              # automated 4-pass: DHCP · static · sysext · NVIDIA
just boot-iso            # build ISO → boot in QEMU GTK window (requires -cpu host)
just e2e                 # build ISO → boot in QEMU GTK window → interactive install
```

`just ci` is the pre-push gate. Never `--no-verify`.

**Pre-commit guard:** `no-floating-action-tags` blocks third-party `@main`/`@v*` floating action tags at commit time. `projectbluefin/` refs (`@v1`, `@main`) are intentional managed tags and are exempted.

---

## Safety Invariants ⛔

1. **Never run `flatcar-install` on the host.** `just headless-test` only validates config generation. Real installs run only inside QEMU.
2. **All system commands route through `internal/runner`.** No `exec.Command` outside that package. Reboot wired via `rebootFn func(context.Context) error` injected from `cmd/knuckle/main.go`.
3. **Disk identity is `/dev/disk/by-id`.** Never trust `/dev/sdX` enumeration order.
4. **Never log to stdout.** Bubble Tea owns it. Use `log/slog` with a file handler (`/tmp/knuckle.log` default).
5. **Ignition contains secrets.** Write with `os.CreateTemp` (O_EXCL), `chmod 0600`, `defer os.Remove`. See `internal/install/install.go:WriteIgnitionFile`.

---

## Package Boundaries

Coverage gates are authoritative in [`docs/CI-AND-TESTING.md`](docs/CI-AND-TESTING.md#coverage-gate).

| Package             | Responsibility                                                    |
| ------------------- | ----------------------------------------------------------------- |
| `cmd/knuckle`       | CLI entrypoint, flag parsing, runner wiring                       |
| `internal/model`    | Pure data types — `InstallConfig`, `DiskInfo`, `NetworkInterface` |
| `internal/runner`   | `Runner` interface: `RealRunner`, `DryRunner`, `SpyRunner`        |
| `internal/probe`    | `lsblk` + `ip addr` JSON parsing, `/dev/disk/by-id` resolution   |
| `internal/validate` | Hostname, CIDR, gateway, SSH key, timezone, disk path validators  |
| `internal/bakery`   | `DispatchingClient` routing to Flatcar or FCOS bakery clients; sysext catalog + release/SBOM fetchers, SHA512 + GPG check|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [projectbluefin/knuckle](https://github.com/projectbluefin/knuckle) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
