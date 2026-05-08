---
trigger: always_on
description: Work autonomously and continuously across all pillars. No mocking, no placeholders, no heuristics, no summaries.
---

# API Security Researcher — Development Guide

Work autonomously and continuously across all pillars. No mocking, no placeholders, no heuristics, no summaries.

## Project Overview

A Chrome Extension (MV3) for API discovery, protocol reverse-engineering (Protobuf/JSPB/JSON/gRPC-Web/GraphQL/SSE/NDJSON), JavaScript security code review, and security testing across all websites.

## Pillars

Every change belongs to one pillar. After a landed change, the next change must be in a different pillar until all pillars have moved once; then rotate back. Don't stay in one pillar because it's easier.

1. **Schema learning** — map request/response schemas and URL params from observed traffic + AST call sites. Every field gets a real example value, never a default.
2. **Usable example values** — pull real captured values for every param/field; no synthesized defaults.
3. **Security issue detection** — AST-traced taint from known sources to known sinks. No name matching, no regex — only scope-resolved analysis.
4. **Service grouping quality** — group methods under the right host/path prefix. No cross-origin misattribution from concat tricks.
5. **API vs static asset classification** — magic-byte sniffing + content-type. No URL-suffix heuristics.
6. **Exploit probe** — prove execution end-to-end: NOT REPRODUCED / TAINT REACH ONLY / REAL EXPLOIT. Active payloads (img onerror) set a per-marker flag the probe checks via chrome.scripting.
7. **Real server interaction** — send learned requests through the active page (credentialed) via `schema.verify`, diff response against learned schema.
8. **Website JS review** — follow each finding back to the original JS (sourcemap-resolved). Every per-hop code snippet is captured at analysis time.
9. **Verifying & reviewing output quality** — findings are re-read against real JS; schema.review shows every leaf's example + provenance.
10. **UI review** — the popup and viewer render the same taint-path and tree-shake the reviewer would want; both inspected end-to-end with the harness before claims are made.
11. **Per-finding drill-down** — `finding`, `finding.func`, `finding.callers`, `finding.view`, `finding.exploit` together produce enough context that no guesses are needed.
12. **Harness reviewer context** — every harness command gives ground-truth AST data, not summaries: source snippets per hop, dim transitions, sanitizer report, preconditions, receiver type.
13. **Performance on real sites** — measure on a real 5 MB+ bundle. No change ships with regression > 10 % vs the previous real-site baseline.
14. **Facts not guesses** — every value the analyzer emits is traced structurally. No hardcoded strings, no depth caps, no magic numbers that paper over resolver gaps.
15. **Tooling context improvement** — when a review needed a manual step, the harness learns that step (hop snippets, receiverType, finding.view, etc.).

## Operational Rules

**Output rule.** Output text only when progress is blocked by something external (missing credential, ambiguous irreversible choice). Otherwise call tools. Never write "tests pass, the fix is in" style summaries. The diff is the record.

**Rotation cadence.** After any non-trivial landed change, the next non-trivial change must be in a different pillar. Don't cluster.

**Verification protocol.** A change is landed only after it has been:
1. Exercised on the real site where it applies — the finding/fix seen, diffed, or measured end-to-end.
2. Exercised on at least one site where it should NOT apply — confirming no regression.
3. Snapshot comparisons preferred: `finding.snapshot <name>` before, `finding.diff <name>` after — the diff is the proof.

Tests are for detector polarity (positive + negative, pinned to similar-shape inputs). Tests are never a substitute for real-site verification.

**Resolver-gap ownership.** Any `resolverError` a batch produces is a P1. The next task after the batch is to add the missing resolution path — inter-procedural, scope-aware, whatever's needed. `_resolveAllValues` throws when a concat term can't be traced; the caller catches and records the gap. Don't paper over with placeholders, don't raise an excuse, don't use the word "unresolvable". Close the gap.

**Timeout root-cause.** A command that hangs past 30 s means the service worker or offscreen worker is stuck. Inspect `_scriptBuffers`, `_tabMeta`, the offscreen document via harness. Don't poll-retry. Don't raise the timeout. Find what's blocked.

**Decision log.** When a choice is ambiguous but reversible, pick the biggest-impact path, record the reason in the code comment or commit message, and proceed. Only pause on irreversible choices (destructive ops, shipping).

**Phase discipline.** Each task has three phases: investigate → change → verify on real site. Finish phase N before starting N+1. Don't interleave; don't claim done mid-phase.

**Comments policy.** No preachy comments, no self-congratulation, no "carefully removed the heuristic" narration. A comment explains WHY a non-obvious decision was made, not what the code does or what rule it follows.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NDevTK/APIClient](https://github.com/NDevTK/APIClient) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
