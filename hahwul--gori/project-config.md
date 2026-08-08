---
trigger: always_on
description: gori is an intercepting proxy and workbench for **authorized** security testing, written in
---

# AGENTS.md: working on gori

gori is an intercepting proxy and workbench for **authorized** security testing, written in
Crystal and shipped as one binary. It sits in the loop between a client and its target,
capturing every request and response as a *flow* you can intercept, replay, fuzz, and scan
across HTTP/1.1, HTTP/2, WebSocket, gRPC, and SSE.

Three entry points, one engine layer underneath: `gori` (TUI), `gori mcp` (stdio JSON-RPC for
agents), `gori run <sub>` (headless, for scripts).

This file is the short version. [DESIGN.md](DESIGN.md) is the long one, and its numbering is
load-bearing: source comments cite principles as `(P4)`, `(P6/P7)` and sections as
`DESIGN.md §4`.

## Three things not to get wrong

### 1. Malformed input is the payload (P7)

A security proxy that sanitizes its operator's bytes is broken. The captured wire bytes are
canonical; parsed columns and pretty views are derived projections.

- There is **no** stdlib `HTTP::Request` parsing anywhere on the proxy path.
  `src/gori/proxy/codec/http1.cr` is a byte-exact sans-IO codec: it flags `malformed?` and
  keeps the octets rather than rejecting. `serialize_head` is the identity function.
- **The axis is provenance, not byte values.** The same octet gets three different answers:
  - **operator bytes** (imported HAR, an MCP `raw` request, a replay) go out verbatim, never
    sanitized. See the comment at `src/gori/import/builder.cr:31-38` and
    `src/gori/mcp/request_builder.cr` (`normalize_raw`).
  - **page-authored bytes** (a crawled `<a href>`) get percent-encoded where they merely
    break, and refused where they *frame*. See `src/gori/discover/url.cr` (`encode_unsafe`).
  - **remote-chosen bytes** (a redirect `Location`) are refused outright, not repaired. See
    `src/gori/fuzz/engine.cr` (redirect following).
- The predicate has **one home**: `Codec::Http1.request_token_safe?`. Do not re-derive it next
  to a new caller; that exact shape has already recurred three times (#390, #394, #397).
- Request framing rejects any obfuscation (`obfuscated_header?`); response framing is
  deliberately narrower (`framing_ambiguous?`, and read the comment above it). Do not
  "symmetrize" them: a request's peer is the operator's own browser, a response's peer is the
  whole internet.

### 2. Three surfaces, one engine layer

The TUI is the center of gravity. `gori mcp` and `gori run` are expected to stay at near
parity with it, and every parity gap found so far has been in a surface, not an engine.

- The shared seam is **`Plan.build(options, outbound) : Plan`**, one per tool:
  `src/gori/{fuzz,miner,discover,sequencer,repeater}/plan.cr`. Option *parsing* is
  surface-specific (an `OptionParser` on the CLI, the args hash on MCP, view state in the
  TUI); everything downstream of the normalized options has exactly one implementation.
- Adding a feature means: engine + `Plan.build` path once, then a thin adapter in each of
  `src/gori/tui/`, `src/gori/cli/run/`, `src/gori/mcp/tools/`. Parity is a convention held by
  each surface calling the same engines, not by a shared dispatcher ([DESIGN.md §2](DESIGN.md)).
- The seam is **not** the `Verb` registry. Its 318 verbs are TUI-only by decision: a verb reads
  its target from TUI selection state instead of naming it, and the missing argument schema is
  the blocker, not registry wiring (`src/gori/verb.cr`, DESIGN.md §7). Do not "fix" parity by
  wiring CLI or MCP into the registry.
- **Layering contract:** core subsystems must not know a surface exists. Self-check:

  ```sh
  grep -rnE '\b(Tui|CLI|MCP)::' \
    src/gori/{store,proxy,probe,fuzz,miner,discover,sequencer,oast}/ \
    src/gori/{store,probe,fuzz,miner,discover,sequencer,oast}.cr
  ```

  Today that returns eight hits, all of them comments: `src/gori/store/models.cr` (×6),
  `src/gori/probe/group.cr`, `src/gori/fuzz/types.cr`. A comment may point at a caller; code
  may not — so the check is "every hit is a comment", not a hit count. (The count drifts as
  those comments are edited; that is fine, and is why it is not the check.)
- Every gori-originated request goes through the `Gori::Outbound` chokepoint
  (`src/gori/outbound.cr`). It is a required constructor argument on `Fuzz::Sender` and
  `Repeater::Sender`, so an ungated sender is a compile error. Layer 1 (`check`) is the only
  per-surface variance: `Outbound.agent` (MCP, strict), `Outbound.cli` (permissive when
  unconfigured), `Outbound.interactive` (TUI, no up-front gate). Layer 2 (`sweep_block` /
  `send_block`: sandbox + explicit excludes) is identical everywhere and applies even when
  Layer 1 was waived. Judge the host actually dialled via `Outbound.scope_url`, never the
  request line.

### 3. Never stall the data path (P6), and don't crash

The proxy and the Store writer are hot paths. The proxy plus the HTTP/1.1 codec add only
~25µs per request (`src/gori/store/schema.cr:573-576`); capture, not proxying, has been the
bottleneck every time.

- gori runs on Crystal's **single-threaded** cooperative fiber scheduler. **Never** build or
  benchmark with `-Dpreview_mt`: `Store`, `Fuzz::Engine`, `Miner::Engine`, and
  `Store::SafeRegexp` all depend on it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [hahwul/gori](https://github.com/hahwul/gori) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-08 -->
