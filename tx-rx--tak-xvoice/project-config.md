---
trigger: always_on
description: This file is the standing brief for any Claude Code (or other agentic
---

# CLAUDE.md — Operator handoff for Claude Code sessions

This file is the standing brief for any Claude Code (or other agentic
assistant) session working on this repository. Read it before making
edits, opening shells, or composing commit messages.

## Project identity

TAK-XVoice (XV) is a **public, Apache-2.0-licensed ATAK plugin**. The
repository at `https://github.com/TX-RX/TAK-XVoice` is public and the
default branch is `main`. Anything that lands on `main` is, for
practical purposes, world-readable forever — assume git history is a
publication surface, not a private workspace.

## Sensitive-content rules

The following content categories MUST NEVER appear in committed files,
commit messages, PR descriptions, code comments, log output captured
into the repo, or any message text that gets pushed to a remote (this
includes chat replies the assistant emits while pairing with the
operator):

- **Credentials of any kind**
  - Personal access tokens (GitHub PATs, GitLab PATs, anything similar)
  - API keys, service account keys, OAuth client secrets, OAuth tokens
    (access or refresh)
  - Android/JKS signing keys, keystore passwords, key aliases tied to
    a production keystore
  - SSH private keys, GPG private keys
- **Bluetooth MAC addresses tied to a specific operator's hardware.**
  Use the redacted placeholder `XX:XX:XX:XX:XX:XX` in examples,
  docstrings, log lines, and tests. Realistic-looking fake MACs are
  also fine (e.g. `AA:BB:CC:DD:EE:FF`) — the point is never to ship a
  real operator's device address.
- **TAK server hostnames, FQDNs, URLs, or IP addresses.** This
  includes staging/dev servers as well as production. Use
  `tak.example` / `tak.example.com` / RFC 5737 (`192.0.2.x`,
  `198.51.100.x`, `203.0.113.x`) for examples. Multicast group
  literals derived from server cert fingerprints are also off-limits
  even when they "look" generic.
- **Customer, agency, or unit names.** No identifying organization
  strings, no real operator callsigns, no team identifiers from live
  deployments. Use `Alpha`/`Bravo`/`Charlie` or numbered placeholders
  in examples.
- **Operator GPS coordinates** of any precision. If a bug repro
  involves a real location, replace with a coordinate inside a known
  uninhabited region (e.g. `0.0, 0.0`) or a published city centroid
  before pasting.
- **Internal IP ranges.** No RFC 1918 prefixes that identify a real
  internal network. Generic illustrations of `10.0.0.0/8` etc. are
  fine; specific subnets observed in the field are not.

If the assistant is unsure whether a string falls into one of the
above categories, the default is **redact and ask** — never commit
and apologize later. Operator deployments depend on this; the public
history of an ATAK plugin is one of the first things an adversary
will read.

## Commit-message style

- Subject line: terse, conventional-commit-style prefix
  (`feat:` / `fix:` / `chore:` / `docs:` / `refactor:` / `test:`),
  imperative mood, under ~72 characters.
- Body: technical and descriptive. Explain *why* the change was made,
  what subsystem it touches, and any constraints a future reader
  needs to know. The single squashed `Initial public commit —
  TAK-XVoice` commit is the canonical tone reference: dense,
  factual, organized by subsystem, zero operator-identifying detail.
- Never name customers, agencies, units, real operators, or specific
  field deployments in commit messages, even at a high level.
- Do not paste log excerpts, device serials, or BT MACs into commit
  bodies. Summarize ("verified against an AINA V2 puck") instead of
  quoting raw evidence.

## README maintenance

The top-level `README.md` is the project's public-facing description
of what TAK-XVoice does, who it's for, and what hardware and features
are supported. Because this repo is public, the README is often the
first thing an operator, contributor, or downstream packager reads —
keep it current alongside the code.

- When a change lands that affects user-visible behavior — new
  supported hardware, a new UX affordance, a new transport, a new
  build task, a changed default, a retired feature — update the
  relevant README section in the same PR that ships the code.
  Do not treat README updates as a follow-up task; a merged feature
  that isn't in the README is functionally invisible to new
  operators.
- The **curated-hardware policy** is load-bearing: nothing is listed
  under "Hardware tested" as supported until it has been integrated
  and validated end-to-end against real event traffic. Do not add
  speculative, aspirational, or "should work" entries. When a device
  graduates to supported (or is retired), update both "What's
  different" and "Hardware tested" together.
- When shipped work materially reshapes the roadmap
  ("Now / Next / Later"), promote or retire bullets to reflect
  reality rather than intent. A roadmap that reads like a wish list
  loses signal.
- The **status-and-intended-use disclaimer**, the
  **hardware-philosophy** section, and the **reporting-issues**
  guidance are load-bearing legal / policy language. Do not reword
  them unilaterally — flag proposed changes to the operator first.
- Vendor names for downstream / interop targets that the operator has

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TX-RX/TAK-XVoice](https://github.com/TX-RX/TAK-XVoice) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-03 -->
