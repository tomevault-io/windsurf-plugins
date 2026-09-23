---
trigger: always_on
description: Mine Mail is a cross-platform desktop mail client built with Tauri 2, React, Rust,
---

# Mine Mail — Agent Contract

Mine Mail is a cross-platform desktop mail client built with Tauri 2, React, Rust,
and SQLite. This file contains repository-wide rules for coding agents. Keep it
short and durable.

## Source of truth

- Read this file before changing the repository.
- Read `DESIGN.md` before changing the reusable visual system: visual identity,
  theme or semantic tokens, typography, spatial hierarchy, shell geometry,
  component appearance, motion, visual assets, or accessibility presentation. It
  is the only visual specification. Adding a feature or changing product copy
  with existing visual patterns does not by itself require a `DESIGN.md` change.
- Read the relevant section of `docs/PRODUCT.md` before changing user-visible
  behavior, workflows, feature availability, durable product copy,
  synchronization, accounts, drafts, sending, notifications, contacts, or
  identity rules.
- Read `docs/MAIL_RENDERING.md` before changing MIME parsing, HTML sanitization,
  body render modes, remote images, or reply-history parsing.
- Read `docs/MCP.md` before changing or configuring Mine Mail's local MCP
  service, permissions, tools, transport, or supported agent clients.
- `README.md` is the human setup guide. `docs/RELEASE.md` is a mutable release
  checklist, not a product specification.
- Do not create nested `AGENTS.md` files that repeat these rules. Add a scoped
  instruction only when a subproject has a genuinely different build or safety
  constraint.

When documents conflict, this file controls agent behavior; the domain document
controls its named domain. Existing code or an old screenshot does not silently
override a documented decision. Durable product, architecture, or visual-system
changes require the user's approval and an update to the corresponding canonical
document. Product behavior and durable product copy belong in `docs/PRODUCT.md`;
only reusable visual rules belong in `DESIGN.md`.

## Architecture and safety invariants

- The product is a desktop application. The Vite browser build is only an
  explicit, no-network UI demo and test surface; never create a parallel Web mail
  runtime.
- Rust and SQLite own credentials, IMAP/SMTP, MIME processing, synchronization,
  drafts, Outbox state, and notification decisions. React calls narrow Tauri
  commands and renders local state.
- Preserve offline-first startup: render cached SQLite state immediately, then
  synchronize in Rust without replacing usable content with loading placeholders.
- Keep account data, caches, notification baselines, queued mutations, and
  synchronization state scoped by stable account ID.
- Never expose authorization secrets, passwords, OAuth tokens, complete RFC822
  messages, or unrestricted database/file/network access to React. The exact
  active or user-selected product-data directory is the sole complete-path
  exception, limited to the About and confirmed storage-migration flow required
  by `docs/PRODUCT.md`; mail, attachment, diagnostic, and managed-cache paths
  remain Rust-only.
- Never write mailbox addresses, subjects, message bodies, raw HTML/RFC822,
  credentials, tokens, or complete local paths to logs. Keep errors useful but
  privacy-safe.
- End-user mailbox passwords, authorization secrets, and OAuth access/refresh
  tokens belong in the OS credential store and Rust runtime, not SQLite,
  frontend state, repository files, or build configuration.
- Provider-issued desktop OAuth client metadata is a separate, ignored Rust-only
  build input. Never treat it as an end-user credential, expose it to React or
  logs, or commit production configuration.
- Treat mail content as untrusted. Sanitize in Rust, preserve low-confidence
  content, and isolate sender-controlled complex HTML without scripts.
- Operations that can be retried must be idempotent or explicitly model an
  uncertain outcome. Never turn an unknown SMTP result into an automatic resend.

## Product invariants

- Support at most three connected accounts. The interface has one active account,
  while startup, scheduled, tray, and manual synchronization cover every account.
- A missing, expired, or revoked credential stops network work only for that
  account; its cached mail remains readable and the UI offers reauthentication
  without repeated background-error notifications.
- Message stars are the IMAP `\Flagged` system flag. Local changes are immediate
  and remain queued until the server confirms the requested state.
- Draft edits use a stable draft ID plus SQLite `local_version`. Stale writes
  create conflict copies, stale deletes cannot remove a newer draft, and sending
  binds recipient confirmation and Outbox state to one exact version.
- First historical import establishes the notification baseline. Later unread
  arrivals may notify with sender identity/address, subject, and receiving account
  identity/address, but never body text.
- Contacts, favorites, remarks, account remarks, and avatar overrides are
  Mine Mail-local metadata. IMAP does not own them. A local remark or avatar
  override wins over sender/provider presentation without hiding the real address
  where identity must be clear.
- Never query a remote avatar service. Runtime avatar assets come from local

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Tantless/mine-mail](https://github.com/Tantless/mine-mail) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
