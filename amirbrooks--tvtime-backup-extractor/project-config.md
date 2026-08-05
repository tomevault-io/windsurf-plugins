---
trigger: always_on
description: - Never add a real iOS backup, TV Time output, viewing history, title list, account/profile data,
---

# Repository agent notes

## Privacy boundary

- Never add a real iOS backup, TV Time output, viewing history, title list, account/profile data,
  cookie, database, manifest, completion marker, stable ID, device ID, hash, private URL, screenshot,
  screen recording, PDF preview, or local path.
- Tests and documentation use obviously synthetic values only. Redacting one field does not make a
  recovered artifact safe.
- Native decrypted output belongs in the owner-only app-managed local container, outside every Git
  repository, cloud-sync root, share, and source backup. FileVault remains recommended because the
  sandbox and permissions do not encrypt recovered reports. `.gitignore` and the release scanner
  are final defenses, not storage policy.
- Preserve secure defaults: completed encrypted backup, disconnected phone, read-only source,
  owner-only app-managed local output, sensitive-output confirmation, hidden or secure password
  entry, fresh output, Git/overlap/link/traversal checks, atomic completion markers, and opt-in
  raw-cache or decrypted-manifest retention.
- Never ask a user to upload a backup, output tree, report, database, marker, or screenshot of
  recovered content. Reproduce failures with synthetic fixtures.
- Native diagnostics use only the fixed `RecoveryDiagnostics` event vocabulary. Never log paths,
  identifiers, filenames, titles, counts, passwords, free-form errors, helper stderr, or recovered
  content, and never add network telemetry. Unknown failures remain `unrecognized_failure`.

## Product contract

- macOS 14+ users receive the native SwiftUI workflow through the architecture-specific signed and
  notarized DMGs published in the official v0.2.0 release. End users must not need Python, iMazing,
  Homebrew, Git, or developer tools.
- `script/build_local_app.sh` remains an ad-hoc, local-only acceptance path. The controlled release
  pipeline produced and published Developer ID-signed, notarized, stapled v0.2.0 DMGs for both
  architectures. Do not describe later local candidates as published until their own release gates
  are completed.
- The Python CLI remains the free fallback with Python 3.10 through 3.13. Full encrypted-backup
  recovery is supported on macOS and Linux; Windows supports analysis and report rebuilding from an
  existing completed extraction only in this baseline.
- The canonical readable output is Markdown. Self-contained offline HTML is always produced by a
  successful full report. PDF is optional and must be omitted rather than lose or reshape recovered
  text incorrectly.
- Opening private reports can add filenames to browser/viewer history or Recent Items; user guidance
  must retain that warning.

## Python code map

- `tvtime_extractor/cli.py`: public commands, hidden password prompt, readable summaries, and exit
  handling.
- `tvtime_extractor/models.py`: UI-neutral requests, preflight results, progress events, results, and
  cooperative cancellation token.
- `tvtime_extractor/service.py`: shared preflight and recovery orchestration for CLI and native app.
- `tvtime_extractor/protocol.py`: bounded framed JSON request/control primitives, strict destination
  identity binding, separate secret-channel handling, and bounded sequenced JSON Lines events used
  by the native helper.
- `tvtime_extractor/helper_main.py`: bundled-helper handshake, request validation, progress, terminal
  events, cancellation, and safe public error mapping.
- `tvtime_extractor/extract.py`: encrypted-backup access, selected-domain copying, source
  revalidation, inventory, and extraction completion marker.
- `tvtime_extractor/analyze.py`: schema/integrity checks and normalized private CSV tables.
- `tvtime_extractor/report.py`: readable report, sanitized media tables, report staging, recovery
  marker, and atomic promotion.
- `tvtime_extractor/visual_report.py`: shared visual model plus offline HTML and fidelity-gated PDF.
- `tvtime_extractor/safety.py`: path, permissions, no-follow I/O, portable names, private writes, and
  completion-marker validation.
- `scripts/macos_helper_entry.py`: minimal PyInstaller entry point; package behavior remains in
  `tvtime_extractor/`.

The required primary domain is `AppDomain-com.tozelabs.tvshowtime`. The current analyzer expects
`Documents/DioCache.db`; optional image-cache reporting reads
`Library/Application Support/libCachedImageData.db`. Treat both schemas as version-sensitive.

## Native macOS code map

- `macos/Sources/TVTimeRecoveryCore/`: helper client and protocol decoding, no-link inherited
  destination-handle binding, recovery state machine, strict summary invariants, scoped-resource
  leases, and output validation.
- `macos/Sources/TVTimeRecoveryApp/`: SwiftUI step flow, secure password confirmation, system folder
  pickers, cancellation/quit guards, result chart, report actions, and window behavior.
- `macos/Tests/TVTimeRecoveryCoreTests/`: deterministic Swift tests with fake helpers and synthetic
  output trees.
- `macos/Bundle/`: app/helper plists, sandbox entitlements, and third-party notice.
- `macos/Package.swift` and `macos/Package.resolved`: macOS 14 SwiftPM product and locked test
  dependency.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [amirbrooks/tvtime-backup-extractor](https://github.com/amirbrooks/tvtime-backup-extractor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
