---
trigger: always_on
description: This file is the operating contract for every future coding agent, assistant, or maintainer that modifies ARIEC60870.
---

# AGENTS.md — ARIEC60870 Engineering Guardrails

This file is the operating contract for every future coding agent, assistant, or maintainer that modifies ARIEC60870.

## Product Identity

ARIEC60870 is an **Apache-2.0 clean-room IEC 60870-5-103 Active Master Tester + Analyzer**.

It connects to protection relays / IEDs acting as IEC-103 slaves, runs controlled master polling, decodes the response, updates Value Viewer and Relay Event Log, and exports deep engineering evidence.

ARIEC60870 is **not**:

- a passive decoder only
- a vendor-specific relay tester
- a generic fake profile generator
- a dual-redundancy IEC-101 clone
- a wrapper around external, commercial, GPL, or unclear-license protocol stack source

## Product Non-Negotiables

1. **Active master first**
   - The main product is a master tester that actively connects to IEC-103 slave relays.
   - Offline decoder is a supporting mode for troubleshooting traces.

2. **Single connection baseline**
   - Dual-link redundancy and active/standby architecture may be added only when explicitly requested; keep the wording protocol-neutral and avoid customer- or country-specific convention labels in public files.

3. **Controlled polling**
   - Class 2 is normal/background polling.
   - Class 1 is event-drain only, triggered by ACD=1 or bounded GI follow-up.
   - Never bombard Class 1 while the slave keeps returning NO DATA / ACD=0.

4. **Relay timestamp for Event Log**
   - Event Log time must come from the relay ASDU timestamp when present.
   - PC arrival time is forensic metadata only.
   - GI snapshots update Value Viewer; state change / edge events update Relay Event Log.

5. **User mapping, not built-in vendor profile**
   - Do not ship guessed vendor signal profiles.
   - Built-in code may decode protocol fields only: Type, COT, FUN, INF, DPI, timestamp, quality, raw frame.
   - Signal names must come from user-owned mapping profile files.

6. **Raw evidence always visible**
   - Even when mapped signal names are shown, always retain FUN/INF/Type/COT/DPI/raw hex in UI and reports.

7. **Clean-room implementation**
   - No copied code from external protocol stacks, packet dissectors, vendor SDKs, or commercial protocol stacks.
   - Public docs and standards can guide behavior, but code must be independently written.
   - Do not paste vendor manual tables into the repo unless the user explicitly confirms legal permission.

8. **Apache-2.0 repository hygiene**
   - Keep source code, docs, landing page, sanitized samples, workflows, and legal files.
   - Exclude bin/obj/out/dist/node_modules/raw logs/private captures/secrets.

## Required Architecture Boundaries

Keep protocol logic out of WPF.

```text
ARIEC60870.Desktop
  UI cockpit only: setup, commands, live tables, export actions

ARIEC60870.Cli
  Developer/test harness and offline/batch entrypoint

ARIEC60870.Master
  Active master state machine, polling policy, transport orchestration, live evidence

ARIEC60870.Core
  FT1.2 parser, link control decoder, ASDU decoder, offline analyzer, mapping primitives

ARIEC60870.Reports / future
  Markdown/HTML/PDF reporting and evidence bundle generation
```

WPF must never become the owner of protocol state, FCB/FCV handling, Class 1 drain decisions, or ASDU parsing.

## Master Polling Policy

Correct runtime loop:

```text
Startup:
  Open transport
  Optional startup delay
  Optional Reset Remote Link
  Reset FCB
  Optional Clock Sync
  Optional General Interrogation
  Bounded GI Class 1 follow-up

Normal:
  Poll Class 2 at configured interval

If any secondary response indicates ACD=1:
  Enter Class 1 event drain
  Request Class 1 until NO DATA / GI END / ACD clear / DFC busy / max drain / timeout

If DFC=1:
  Back off and record busy evidence

If timeout:
  Retry in controlled way
  Reset FCB only after configured timeout burst
```

Forbidden pattern:

```text
Request Class 1
NO DATA
Request Class 1
NO DATA
Request Class 1
NO DATA
```

## Value Viewer vs Relay Event Log

### Value Viewer

Current snapshot table.

- updated by GI responses, cyclic/background values, and event responses
- uses latest known state/value per signal key
- can use user mapping profile for signal name/group/state map
- unmapped points remain visible as raw FUN/INF

### Relay Event Log

SOE-like relay event list.

- event time = relay timestamp from ASDU time field when available
- logs state change / edge event / spontaneous event
- does not log every repeated GI/status snapshot
- stores PC arrival time only as forensic metadata

## Mapping Profile Rules

Mapping profile is a user/project asset.

Allowed:

- JSON schema/config for user-defined mapping
- import/export profile
- profile validation
- sample profile clearly labeled as example user mapping

Forbidden:

- `vendor-specific basic profile` unless the user supplies validated data and asks for that profile
- guessed FUN/INF labels as final truth
- hiding raw FUN/INF behind friendly names

## UI Direction

Desktop UI should feel like a serious engineering cockpit with the same mature engineering product identity used by the desktop tool:

- clean, modern, calm, professional

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [masarray/ARIEC60870](https://github.com/masarray/ARIEC60870) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-02 -->
