---
trigger: always_on
description: - Dobby is a compact developer client for Minecraft Bedrock on mcpelauncher.
---

# Dobby contributor guidance

## Mission

- Dobby is a compact developer client for Minecraft Bedrock on mcpelauncher.
- Prioritize exact evidence over generic interpretations or guesses.
- Keep the in-game interface small even when reports are comprehensive.
- Never alter gameplay packets; observe and report only.
- Fail closed when a target address, signature, layout, or ABI is uncertain.
- Preserve useful diagnostics locally in text and JSONL formats.
- Treat repeat occurrences as new events that may require a new popup.
- Design every subsystem so future developer features can be added independently.
## Architecture

- `src/dobby.cpp` owns only exported lifecycle entrypoints.
- `src/core/` owns configuration, constants, and shared runtime state.
- `src/diagnostics/` owns decoded records, stream traces, and reports.
- `src/network/` owns the packet ID-to-name catalog.
- `src/hooks/` owns image discovery, validation, detours, and hook installation.
- `src/platform/` owns files, logs, clipboard, and launcher ABI bridges.
- `src/ui/` owns the in-game menu, windows, actions, and presentation.
- `tests/` exercises portable behavior without launching Minecraft.
## Core modules

- Put immutable target metadata in `core/constants.hpp`.
- Put environment parsing and output-path selection in `core/config.*`.
- Put session counters, toggles, history, and hook status in `core/runtime_state.*`.
- Do not access environment variables outside the configuration module.
- Do not duplicate target versions, build IDs, offsets, or limits.
- Keep runtime state synchronized and return snapshots to presentation code.
- Bound every collection or capture controlled by untrusted network input.
- Keep configuration parsing deterministic and directly unit tested.
## Diagnostic modules

- `types.hpp` contains passive data structures and no platform behavior.
- `violation_decoder.*` understands the warning-packet object layout only.
- `stream_probe.*` understands the ReadOnlyBinaryStream layout and read traces only.
- `report_builder.*` converts evidence into UI summaries, text, and JSON.
- Keep raw evidence distinct from interpretations in every report.
- Persist only fields and boundaries observed directly from the client.
- Never include live process pointers in persisted reports.
- Add a unit test whenever a decoded field or layout offset changes.
## Network catalog

- `packet_names.hpp` is the canonical packet-ID-to-name table.
- Regenerate packet names from the matching LeviLamina generated header.
- Client field names come from the runtime PacketSchemaReader trace.
- Do not claim a field boundary unless instrumentation proves it.
- Unknown packet IDs must remain printable as numeric and hexadecimal values.
- Unknown schemas must still expose byte-level stream evidence.
- Keep catalog data independent from hooks and launcher APIs.
- Document the Bedrock target used when catalog entries are refreshed.
## Hook modules

- All target addresses are image-relative constants.
- Validate that function addresses fall inside the executable segment.
- Validate instruction signatures before touching a vtable.
- Validate the current vtable target before replacing a slot.
- Use `mcpelauncher_patch` for launcher-compatible writes.
- Keep warning capture operational when the optional stream probe cannot install.
- Preserve all ABI-required registers in assembly detours.
- Never broaden a signature match merely to make an unsupported build start.
## Platform modules

- `files.*` owns directory creation, timestamps, escaping, hex, and file writes.
- `log.*` owns human logs and lifecycle JSONL events.
- `launcher.*` owns all mcpelauncher menu, window, and clipboard ABI declarations.
- Keep weak launcher imports isolated from portable code.
- Use the launcher's ImGui clipboard bridge when available.
- Always write a local clipboard fallback before reporting clipboard failure.
- Do not add GUI frameworks or runtime dependencies without a clear need.
- Keep Android-only declarations behind `__ANDROID__` guards.
## UI modules

- The root menu name is `Dobby`.
- The root action opens developer status, not an empty violation window.
- Violation windows remain non-modal to avoid persistent black dim layers.
- Show only packet, status, exact reason, and decode boundary by default.
- Put full structure, trace, paths, and JSON behind copy actions.
- Automatic popup is independent from history deduplication.
- Every captured violation may show after the previous window is closed.
- New controls must remain useful during a disconnected game state.
## Safety

- Treat server packet data as untrusted input.
- Cap context length, raw capture length, trace length, and history length.
- Validate pointers and lengths before copying stream data.
- Do not patch unsupported Minecraft builds.
- Do not send packets, bypass checks, or suppress Bedrock disconnects.
- Do not publish runtime logs, packet captures, credentials, or local profiles.
- Keep previous installed builds recoverable during deployment.
- Prefer partial diagnostics over an unsafe hook.
## Testing

- Host tests must cover short and long Android libc++ strings.
- Host tests must cover stream boundaries and predicted overflow.
- Host tests must cover repeated identical violations.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [evc24004/dobby](https://github.com/evc24004/dobby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-11 -->
