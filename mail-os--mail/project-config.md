---
trigger: always_on
description: A performant, self-hosted mail server written in Zig. Supports SMTP, IMAP, POP3, CalDAV/CardDAV, and more.
---

# Mail Server

A performant, self-hosted mail server written in Zig. Supports SMTP, IMAP, POP3, CalDAV/CardDAV, and more.

## Project Structure

```
mail/
├── packages/
│   ├── zig/          # Core mail server (Zig 0.16.0-dev)
│   │   ├── src/
│   │   │   ├── main.zig              # Server entry point
│   │   │   ├── mail_cli.zig          # CLI entry point (zig-cli)
│   │   │   ├── core/                 # Core: config, logging, protocol, TLS, sockets
│   │   │   ├── protocol/             # IMAP, POP3, CalDAV, ActiveSync, etc.
│   │   │   ├── auth/                 # Auth, password hashing (Argon2id), CSRF
│   │   │   ├── storage/              # SQLite database layer
│   │   │   ├── delivery/             # Queue, bounce handling, TLS-RPT
│   │   │   ├── antispam/             # DKIM, SPF, DMARC, ARC, DNSBL
│   │   │   ├── observability/        # Logging, metrics, tracing, Discord alerts
│   │   │   ├── features/             # Sieve, quotas, templates, autoresponder
│   │   │   ├── infrastructure/       # Clustering, io_uring, connection pooling
│   │   │   └── api/                  # REST API, health checks
│   │   ├── build.zig
│   │   └── pantry/                   # Zig dependencies (zig-tls, zig-cli)
│   ├── cloud/        # AWS infrastructure (ts-cloud / CloudFormation)
│   │   └── cloud.config.ts           # EC2, SES, Route53, IAM config
│   └── ts/           # TypeScript SDK (ts-mail)
├── pantry.jsonc       # Monorepo config (workspaces, scripts)
└── docs/              # Architecture, security, protocol docs
```

## Build & Development

```bash
# Package manager is `pantry` (like npm/bun)
pantry run build          # ReleaseFast build
pantry run build:debug    # Debug build
pantry run dev            # Build + run server locally
pantry run test           # Run all tests
pantry run fmt            # Format Zig source

# Or directly from packages/zig:
cd packages/zig
zig build                         # Debug build
zig build -Doptimize=ReleaseFast  # Release build
zig build -Dtarget=x86_64-linux   # Cross-compile for Linux
zig build test                    # Run tests
zig build run -- serve            # Run server
```

## Releasing

`better-dx` supplies the dev tooling (bumpx, logsmith, pickier, gitlint) — run
`bun install` once at the repo root.

```bash
pantry run release:patch   # or release:minor, release:major
pantry run release         # prompts for the level
pantry run release:dry     # preview, writes nothing
```

They all go through `scripts/release.ts`, which gates the release (clean `main`
in sync with origin, every manifest on the same version) and then runs bumpx
over an **explicit** manifest list — `package.json`, every
`packages/*/package.json`, `packages/zig/build.zig.zon`. The list is explicit
because `--recursive` discovery also reaches the vendored Zig dependencies under
`packages/zig/{vendor,zig-pkg,pantry}/` and would bump those too. bumpx
regenerates `CHANGELOG.md` via logsmith, commits, tags and pushes; the tag push
starts `.github/workflows/release.yml`.

bumpx only rewrites a manifest whose current version matches the one it is
bumping from, so a drifted manifest is skipped **silently**. `bun test scripts`
asserts they are all in lockstep and CI runs it on every PR.

See [docs/RELEASE_PROCESS.md](docs/RELEASE_PROCESS.md).

## Zig 0.16 Specifics

This project uses Zig 0.16.0-dev which has breaking changes from 0.15:

- `std.ArrayList` is unmanaged: init with `.empty`, pass allocator to every method (`append(allocator, ...)`, `deinit(allocator)`)
- `std.time.sleep` removed: use `time_compat.sleep()` (our wrapper using `std.c.nanosleep`)
- `std.fs.cwd()` removed: use `fs_compat` module for file operations
- `std.process.Child.run` removed: use `extern "c" fn system()` or C file I/O
- `std.c.stat/remove/system` removed: declare `extern "c"` functions directly
- Argon2 password hashing uses `p=1` (single-threaded) to avoid async I/O requirements in tests
- Custom compat layers: `src/core/io_compat.zig`, `src/core/fs_compat.zig`, `src/core/time_compat.zig`, `src/core/socket_compat.zig`

## Production Server

- **Instance**: `i-0e365c6bd31da4678` (EC2, Amazon Linux 2023, us-east-1)
- **Domain**: `mail.stacksjs.com`
- **Binary**: `/opt/mail/mail-server` (renamed from `mail` to avoid conflict with `mail/` mailbox dir)
- **Mailboxes**: `/opt/mail/mail/{username}/new/` (Maildir format)
- **Database**: `/opt/mail/smtp.db` (SQLite)
- **Config**: `/etc/mail/mail.env`
- **TLS**: Let's Encrypt at `/etc/letsencrypt/live/mail.stacksjs.com/`
- **Service**: `mail.service` (systemd, runs as `mail-server` user with `CAP_NET_BIND_SERVICE`)
- **Delivery**: Amazon SES (`SMTP_DELIVERY_METHOD=ses`)
- **Ports**: 25 (SMTP), 465 (SMTPS), 587 (Submission), 143 (IMAP), 993 (IMAPS)

## Deployment via AWS SSM

Deploy new builds without SSH. The server has SSM agent running and an IAM role with SSM permissions.

```bash
# 1. Cross-compile for Linux
cd packages/zig
zig build -Dtarget=x86_64-linux

# 2. Upload binary to S3
aws s3 cp zig-out/bin/mail s3://stacks-production-s3-email/deploy/mail-server-new

# 3. Deploy via SSM (stop service, swap binary, restart)
aws ssm send-command \
  --instance-ids i-0e365c6bd31da4678 \
  --document-name "AWS-RunShellScript" \

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mail-os/mail](https://github.com/mail-os/mail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-26 -->
