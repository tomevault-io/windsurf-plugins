---
trigger: always_on
description: These rules apply to every file in this repository. They are intended to be safe for a public repository and may be made stricter by rules in a nearer directory.
---

# WakeGPT Repository Rules

These rules apply to every file in this repository. They are intended to be safe for a public repository and may be made stricter by rules in a nearer directory.

## Product Boundary

- WakeGPT is independently designed and implemented. External products may inform user needs and observable behavior, but their source, styles, assets, configuration, schemas, private protocols, tests, data, and Git history must not be copied or mechanically rewritten.
- Restate referenced behavior as a neutral WakeGPT requirement before designing or implementing it.
- Do not imply an official partnership, certification, or endorsement by a model or tool provider.
- Use `构想`, `已定义`, `原型`, `已验证`, and `已交付` consistently. Planned or unverified behavior must not be described as supported, secure, or compatible.

## Cockpit Replacement Boundary

- WakeGPT is a replacement for Cockpit, not an extension, plugin, companion, or dependent service for Cockpit.
- WakeGPT source, runtime, development, tests, builds, packaging, instance discovery, configuration, persistence, migration, and verification must not depend on the Cockpit app, processes, services, registries, databases, configuration, private paths, APIs, source code, or build artifacts.
- WakeGPT must create and manage its own local state and required execution surfaces through independently implemented, documented platform or provider interfaces.
- A Codex process, debugging endpoint, configuration, or other environment prepared by Cockpit is not valid proof that a WakeGPT feature works. Reproduce the required setup through WakeGPT before marking the feature verified.
- Cockpit may be mentioned only as historical context or neutral evidence of a user need. If a WakeGPT path works only while Cockpit is installed or running, treat that path as blocked rather than supported.
- Current explicit user instructions may change WakeGPT's replacement scope. A request to reproduce an observable outcome does not authorize a Cockpit runtime or implementation dependency.

## Public Repository Boundary

- Treat every Git-tracked file and every committed history entry as public disclosure.
- Never commit personal names, home-directory paths, private hostnames, internal URLs, signing identities, backup locations, account inventories, or machine-specific initialization records.
- Never commit credentials, API keys, access tokens, refresh tokens, cookies, sessions, private keys, certificates containing private material, account exports, production databases, or unredacted operational logs.
- Test fixtures, screenshots, recordings, crash reports, and examples must use synthetic or irreversibly sanitized data.
- Personal working preferences belong in the user's global agent configuration, not this repository. A local root `AGENTS.override.md`, when used, must remain untracked and must preserve every privacy, security, provenance, and release rule in this file because it replaces the root `AGENTS.md` for Codex discovery.

## Privacy Hard Requirements

- Collect and retain the minimum data required for an explicitly defined user-facing purpose.
- Before adding a new data category or external transfer, document its purpose, source, destination, access boundary, retention period, deletion path, and user control.
- Telemetry and crash upload remain off by default until there is a public privacy notice, explicit user choice, redaction, retention limits, and a tested disable path.
- Users must have documented export and deletion paths for locally stored account metadata, prompts, outputs, attachments, remote-operation history, and audit records.
- Logs must redact secrets and minimize prompts, outputs, account identifiers, file contents, network addresses, and remote command results.
- Security or diagnostic collection must not silently become product analytics.

## Credentials And Accounts

- Store secrets only through an approved platform secure-storage mechanism. Do not add a plaintext fallback.
- Keep credentials isolated from ordinary application data, exports, backups, logs, and model context.
- Design rotation, revocation, expiry, deletion, and recovery before a credential format is considered stable.
- Manage only accounts and service credentials the user is authorized to use.
- Do not use account selection, pooling, or failover to evade access controls, provider limits, suspensions, pricing, or terms.
- Consumer login sharing, browser-cookie import, private endpoints, and UI automation are not acceptable production integration paths unless the provider explicitly authorizes that exact use.

## Live Account Test Gate

- Synthetic, local, and contract checks must run before any live-account test. A real provider account may be used only after the user explicitly authorizes that provider, account, purpose, and bounded test for the current task.
- ChatGPT/Codex official login must use the provider's official web authorization flow. WakeGPT may open that flow and observe its completion, but the user enters passwords, passkeys, one-time codes, and other sign-in factors directly on the provider page; WakeGPT must not request, type, read, store, or relay them.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Awaker-OTE/WakeGPT](https://github.com/Awaker-OTE/WakeGPT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
