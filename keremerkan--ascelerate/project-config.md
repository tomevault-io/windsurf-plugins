---
trigger: always_on
description: A command-line tool for the App Store Connect API, built with Swift.
---

# ascelerate

A command-line tool for the App Store Connect API, built with Swift.

## Build & Run

```bash
swift build                           # Debug build
swift build -c release                # Release build (slow — AppStoreAPI has ~2500 generated files)
swift run ascelerate <command>        # Run directly
swift run ascelerate --help           # Show all commands
```

Install globally:
```bash
strip .build/release/ascelerate              # Strip debug symbols (~175 MB → ~59 MB)
cp .build/release/ascelerate /usr/local/bin/
```

## Project Structure

```
Package.swift                         # SPM manifest (Swift 6.0, macOS 13+)
Sources/ascelerate/
  Ascelerate.swift                    # @main entry, root AsyncParsableCommand, central error handling
  Config.swift                        # ~/.ascelerate/config.json loader, ConfigError
  ClientFactory.swift                 # Creates authenticated AppStoreConnectClient
  Formatting.swift                    # Shared helpers: Table.print, ANSI colors, formatFieldName/formatState, formatDate, expandPath
  Aliases.swift                        # Alias storage (~/.ascelerate/aliases.json), resolveAlias()
  MediaUpload.swift                   # Media management: upload, download, retry screenshots/previews
  Commands/
    ConfigureCommand.swift            # Interactive credential setup, file permissions
    AppsCommand.swift                 # All app subcommands + findApp/findVersion helpers
    BuildsCommand.swift               # Build subcommands
    IAPCommand.swift                  # In-app purchase subcommands
    SubCommand.swift                 # Subscription subcommands
    DevicesCommand.swift              # Device management subcommands + findDevice helper
    CertsCommand.swift                # Signing certificate subcommands + findCertificate helper
    BundleIDsCommand.swift            # Bundle identifier subcommands + findBundleID helper
    ProfilesCommand.swift             # Provisioning profile subcommands + findProfile helper
    AliasCommand.swift                # Alias management (add, remove, list) for bundle ID shortcuts
    RunWorkflowCommand.swift          # Sequential command runner from workflow files
    InstallCompletionsCommand.swift   # Shell completion installer with post-processing patches
    InstallSkillCommand.swift         # Claude Code skill installer (fetches from GitHub)
    RateLimitCommand.swift            # API rate limit status check
skills/
  ascelerate/SKILL.md                # AI coding skill (single source of truth)
  package.json                        # npm package for npx installer
  bin/install.js                      # npx installer (fetches SKILL.md from GitHub)
```

## Dependencies

- **[asc-swift](https://github.com/aaronsky/asc-swift)** (1.0.0+) — App Store Connect API client
  - Product used: `AppStoreConnect` (bundles both `AppStoreConnect` core and `AppStoreAPI` endpoints)
  - `AppStoreAPI` is a target, NOT a separate product — do not add it to Package.swift dependencies
  - API path pattern: `Resources.v1.apps.get()`, `Resources.v1.apps.id("ID").appStoreVersions.get()`
  - Sub-resource access: `Resources.v1.appStoreVersions.id("ID").appStoreVersionLocalizations.get()`
  - Client is a Swift actor: `AppStoreConnectClient`
  - Pagination: `for try await page in client.pages(request)`
  - Resolved version: 1.5.0 (with swift-crypto, URLQueryEncoder, swift-asn1 as transitive deps)
- **[swift-argument-parser](https://github.com/apple/swift-argument-parser)** (1.3.0+) — CLI framework
- **[swift-certificates](https://github.com/apple/swift-certificates)** (1.0.0+) — X.509 certificate and CSR generation (used by `certs create` auto-CSR flow)
- **[Yams](https://github.com/jpsim/Yams)** (5.0.0+) — YAML parsing (used by `screenshot` command for `ascelerate.yml`)

## Authentication

Config file at `~/.ascelerate/config.json`:
```json
{
    "keyId": "KEY_ID",
    "issuerId": "ISSUER_ID",
    "privateKeyPath": "/Users/.../.ascelerate/AuthKey_XXXXXXXXXX.p8"
}
```

- `configure` command copies the .p8 file into `~/.ascelerate/` and writes the config
- File permissions set to 700 (dir) and 600 (files) — owner-only access
- JWT tokens use ES256 (P256) signing, 20-minute expiry, auto-renewed by asc-swift
- Private key loaded via `JWT.PrivateKey(contentsOf: URL(fileURLWithPath: path))`

## Commands

```
ascelerate configure                                              # Interactive setup
ascelerate apps list                                              # List all apps
ascelerate apps info <bundle-id>                                  # App details
ascelerate apps versions <bundle-id>                              # List App Store versions
ascelerate apps localizations view <bundle-id> [--version X]      # View localizations
ascelerate apps localizations update <bundle-id> [--locale X]     # Update single locale via flags
ascelerate apps localizations import <bundle-id> [--file X]       # Bulk update from JSON file
ascelerate apps localizations export <bundle-id> [--version X]    # Export to JSON file
ascelerate apps review preflight <bundle-id> [--version X]           # Pre-submission checks (includes IAP/sub state and pricing)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keremerkan/ascelerate](https://github.com/keremerkan/ascelerate) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
