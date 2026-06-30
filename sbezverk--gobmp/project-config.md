---
trigger: always_on
description: `gobmp` is a Go implementation of the BGP Monitoring Protocol (BMP, RFC 7854) and related BGP
---

# GitHub Copilot Instructions for gobmp

## Project Overview

`gobmp` is a Go implementation of the BGP Monitoring Protocol (BMP, RFC 7854) and related BGP
message parsing (RFC 4271, RFC 4760, RFC 9072, and many extension RFCs). It produces structured
messages consumed by downstream systems (Kafka, NATS, MongoDB connectors).

---

## Review Scope and Behavior

When reviewing a pull request, **always produce a complete review of every changed file**.
Do not stop after 2–3 comments. Continue until every changed file has been reviewed and all
findings have been reported. Group related findings per file.

### Review Completeness

- Review ALL changed files in a single pass. Do not split findings across multiple reviews.
- Provide ALL comments in one review submission. Do not batch into groups of 4-5.
- For each file, check: correctness, bounds safety, error handling, test coverage, RFC compliance.
- Prioritize findings by severity: security/crash > logic bug > style > nit.

### What to Flag

- Missing bounds checks before slice operations
- Incorrect byte offset tracking (forgetting `p += N` after reading N bytes)
- Type assertions without comma-ok pattern
- Slice aliasing bugs (modifying shared backing arrays)
- Missing test coverage for new code paths
- RFC specification violations (cite the specific RFC section)
- Error messages missing context (need/have byte counts)
- Unreachable dead code (redundant bounds checks, impossible branches)

### Confidence and Quality

- **Only report findings you are highly confident about.** Do not speculate or flag potential
  issues unless you can explain exactly why the code is wrong and how to fix it.
- If you are unsure whether something is a bug or intentional, do not comment on it.
- Every comment must include a concrete fix or actionable suggestion, not just "this looks wrong."
- Avoid false positives. One accurate, high-impact comment is worth more than ten speculative ones.

### RFC Compliance (Mandatory)

This project implements multiple IETF RFCs. **Every review comment about protocol behavior must
reference the specific RFC and section number** that supports the finding.

Examples of good RFC references:
- "RFC 8955 Section 4 requires multiple NLRIs to be concatenated in a single MP_REACH_NLRI"
- "Per RFC 4760 Section 3, the Reserved byte must be consumed after the Next Hop field"
- "RFC 8956 Section 3 defines prefix byte count as ceil((length - offset) / 8)"

Do NOT make protocol claims without citing the RFC. If you cannot find the relevant RFC section
to support your comment, do not post the comment.

Key RFCs for this project:
- RFC 4271: BGP-4 base specification
- RFC 4760: Multiprotocol Extensions (MP_REACH_NLRI / MP_UNREACH_NLRI)
- RFC 7854: BMP (BGP Monitoring Protocol)
- RFC 8955: FlowSpec IPv4
- RFC 8956: FlowSpec IPv6
- RFC 9072: Extended Optional Parameters Length
- RFC 9256: Segment Routing Policy
- RFC 7432: EVPN
- RFC 4364: BGP/MPLS IP VPNs (L3VPN)
- RFC 6513/6514: Multicast VPN
- RFC 4684: Route Target Constraint
- RFC 9252: SRv6 BGP Overlay Services
- RFC 9012: Tunnel Encapsulation Attribute

### What NOT to Flag

- Style preferences that don't affect correctness
- Minor comment wording unless factually wrong
- Import ordering (handled by goimports)
- Line length unless it affects readability
- Speculative issues without concrete evidence of a bug

---

## Go-Specific Guidelines

### Error Handling
- All error messages in binary parsers must include both **"need N bytes"** and **"have M bytes"**
  (`len(b)-p`) at the point of failure — consistent with the pattern used throughout
  `pkg/bgp/mp-reach-nlri.go` and `pkg/bgp/bgp-update.go`:
  ```go
  fmt.Errorf("not enough bytes to unmarshal X: need %d bytes, have %d", needed, len(b)-p)
  ```
- Error messages must refer to the **remaining** bytes (`len(b)-p`), not the total buffer
  size (`len(b)`), which is misleading when `p > 0`.
- Never return a nil error with a nil value without explanation; always wrap with context.

### Bounds Checks
- Bounds checks that can never trigger (because a stricter guard already exists above them)
  are dead code and must be removed. Flag any unreachable `if p+N > len(b)` guards.
- Checks must be placed **before** the slice operation they protect, not after.

### Code Structure
- Table-driven tests are required for all new functionality. Each table entry must cover at
  least one happy path and one error/edge-case path.
- Test functions must not shadow package-level identifiers (e.g., naming a local variable
  `base` when the package imports `pkg/base`).
- Helper functions duplicating standard library functions (e.g., reimplementing
  `strings.Contains`) must be replaced with the standard library equivalent.

### Naming & Comments
- RFC references must cite the specific section, e.g., `// RFC 4760 §3` not just `// RFC 4760`.
- Exported symbols must have doc comments.
- Internal/unexported helpers used only in tests should be defined in `_test.go` files.

---

## Protocol-Specific Checks

### BGP OPEN (`pkg/bgp/bgp-open.go`)
- `HoldTime` must be validated: only `0` or `>= 3` are valid (RFC 4271 §4.2).
- `BGPID` of `0.0.0.0` is invalid (RFC 6286).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sbezverk/gobmp](https://github.com/sbezverk/gobmp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
