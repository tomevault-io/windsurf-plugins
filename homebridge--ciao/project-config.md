---
trigger: always_on
description: Always reference these instructions first when making changes to the ciao repository. This file highlights repository-specific guidance and enforces that all changes which affect mDNS / DNS-SD behavior are reviewed for RFC compliance.
---

# Copilot Instructions for ciao

Always reference these instructions first when making changes to the ciao repository. This file highlights repository-specific guidance and enforces that all changes which affect mDNS / DNS-SD behavior are reviewed for RFC compliance.

## Repository summary

cIAO (Ciao) implements mDNS / DNS‑SD functionality and MDNS-conformance testing tools. Changes in networking, mDNS, DNS-SD, responder logic, name conflict handling, or packet serialization/deserialization can impact interoperability with other implementations and must follow the relevant RFCs.

## Mandatory RFC compliance requirement

All pull requests and changes that touch mDNS, DNS-SD, responder behavior, discovery, or any network-level handling MUST be reviewed for compliance with:

- RFC 6762 (Multicast DNS)
- RFC 6763 (DNS-Based Service Discovery)

This repository contains an additional instructions file (`.github/instructions/*.instructions.md`) that states: "All changes need to be reviewed for compliance with RFC 6762 and RFC 6763" — ensure you follow that requirement for any relevant changes.

## Pull Request guidance

- Target branch: follow the repository's branching strategy as usual.
- PR title and description: include a short summary of behavior changes and explicitly state whether the PR affects mDNS/DNS-SD behavior.
- All pull requests and changes MUST trigger an automatic Copilot RFC 6762/6763 compliance review. Copilot will run on every PR, analyze diffs for mDNS/DNS-SD impact, post findings as review comments, and add the `rfc-review-required` label for `error` severity issues that need human attention. If the PR affects mDNS/DNS-SD, also include a brief summary of protocol-relevant changes and any tests added.

(Checklist removed — use the Copilot automated PR review below to perform an RFC-focused first-pass.)

If you are unsure about RFC compliance after Copilot's automated review, add the `rfc-review-required` label and request a human reviewer with mDNS/DNS-SD expertise for a deeper review.

## Reviewer responsibilities

- When reviewing PRs that affect mDNS/DNS-SD, evaluate Copilot's automated review output (if present), verify findings are addressed, and ensure any `error` severity items are either fixed or labeled `rfc-review-required` and escalated to a human reviewer.
- Run the included tests locally (see "How to run checks" below) and validate that no regressions are introduced.
- Check for potential interoperability or security issues caused by timing, caching, or record handling changes.

## Suggested automated checks and CI

- Add a GitHub Action that triggers Copilot's automated RFC review for all pull requests (or invokes the review bot). The Action should:
	- run on pull_request events,
	- scan changed files for mDNS/DNS-SD-relevant paths,
	- invoke Copilot/bot to post review comments following this guidance, and
	- automatically add the `rfc-review-required` label when Copilot reports an `error` severity.
- If possible, add or update a CI job that runs the MDNS-related test suite (unit and integration tests) for PRs that modify networking code. This improves catch rate for protocol regressions.
- Consider adding a lightweight linter or script that inspects changes to mDNS-related source files and enforces an RFC-focused review; the GitHub Action above can coordinate that enforcement.

## Copilot automated PR review (instructions for Copilot)

When reviewing pull requests, Copilot should actively scan diffs for changes that may affect mDNS/DNS-SD behavior and provide an automated, RFC-focused code review. Aim to surface potential RFC 6762 / 6763 issues, provide concrete guidance, and escalate to a human reviewer when ambiguous or risky.

Primary rule for reviewers invoked on a PR: when asked to review a pull request, run an RFC 6762 and RFC 6763 compliance check over the PR diff and post findings as GitHub review comments. Mark high-severity findings so maintainers know human attention is required.

Detection heuristics (scan PR diffs for):

- File path or filename patterns: `src/**/MDNSServer*`, `src/**/Responder*`, `src/**/CiaoService*`, `src/**/responder/**`, `src/**/mdns/**`, `src/**/dns/**`.
- Keywords in diffs: `mDNS`, `mdns`, `DNS-SD`, `DNS-SD`, `service`, `probe`, `announce`, `SRV`, `PTR`, `TXT`, `A/AAAA`, `TTL`, `unicast`, `suppression`, `conflict`, `probe`, `announce`, `collision`, `name-change`.
- Changes to serialization/packet-building code, message timing, or probe/announce logic.

Automated checks Copilot should perform on matched diffs:

1. Resource record changes
	- Detect additions/changes to PTR/SRV/TXT/A/AAAA records. Check for correct label formatting, escaping and service-type domain (`_service._tcp.local.`) per RFC 6763.
2. Probing/announce behavior
	- If probe/announce code is modified, verify the presence of the 3-probe sequence and conflict handling logic described in RFC 6762. Flag missing or shortened probe sequences.
3. Multicast address and port usage
	- Confirm code still targets 224.0.0.251 / ff02::fb and UDP port 5353 (or documents why a different approach is used).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [homebridge/ciao](https://github.com/homebridge/ciao) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
