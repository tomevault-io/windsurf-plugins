---
trigger: always_on
description: handles. The Parquet + metadata sidecars on disk make repeat
---

# VeloQ — Contributor Guidelines

This file is for agents/contributors **developing VeloQ itself**
(adding verbs, profile sources, fixes, refactors). User-facing
agents that use `veloq` as a black-box CLI to analyze profiles
should read the skills under `.claude/skills/`:

- `.claude/skills/nsys-profile-analysis/` — Nsight Systems timelines
- `.claude/skills/ncu-profile-analysis/` — Nsight Compute kernel reports

VeloQ (velo-query) is a profile-query CLI family. Pure CLI in /
JSON contract out by default, CSV/table projections for row-shaped
views, no GUI, no MCP server in v1. Today it covers Nsight Systems
(timeline traces) and Nsight Compute (kernel reports) through a
single binary with a shared envelope and pluggable `ProfileSource`
trait. Perfetto and Perfsim are planned along the same shape.

## Wire-format invariants (do not break casually)

These constrain how every new verb/source must emit data. The
user-facing contract description (with examples) lives in each
skill's `SKILL.md`; this section is the maintainer-side rule set.

The JSON envelope and the per-source `version`s are VeloQ's public
contract; the crate's `0.x` Cargo version is independent of the wire
version (breaking shape changes bump `ENVELOPE_VERSION`/`source.version`
plus a CHANGELOG entry — see invariant 1; additive fields keep the
version).

1. **Envelope shape**: `veloq_core::Envelope<T>` is the only success
   payload VeloQ writes on stdout, and `veloq_core::EnvelopeError`
   is the only error payload. Both carry `schema` / `source` /
   `command` / `trace?` / `trace_span?` / `data | error`. Bump
   `ENVELOPE_VERSION` only on a breaking shape change; additive
   fields keep the same version.
2. **Canonical list contract**: every list-shaped response uses
   `data: { count, total_matched, rows: Vec<Row>, auxiliary? }`.
   Each `Row` carries a `pub key: String` composed from the row's
   identifying axes — see the per-verb format below. Non-primary
   data (per-mode common blocks, bucket histograms, …) goes under
   `auxiliary`. New verbs MUST conform — don't add a parallel list
   field with a different name. `wire_format_smoke::every_primary_rows_item_carries_key`
   structurally enforces the `key` presence across every Response
   type.

   Per-verb `key` formats (the substrate for `INDEX(.rows; .key)`
   cross-trace joins):

   | Verb                        | Row key format                                                                                                                                                                                                |
   | --------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
   | `stats`                     | `kind\|<name?>\|dev:<n?>\|stream:<n?>\|ctx:<n?>\|graph:<n?>\|graph_node:<n?>\|style:<push_pop\|start_end\|unknown?>\|nvtx:<rowid-or-none?>\|nvtx-path:<path-or-none?>\|grid:<x>x<y>x<z>?\|block:<x>x<y>x<z>?` |
   | `search`                    | `<row_id>` (e.g. `kernel:1234`)                                                                                                                                                                               |
   | `inspect`                   | `<row_id>` (matches the requested row_id; `NotFound` same)                                                                                                                                                    |
   | `timeline`                  | `bucket\|<start_ns>..<end_ns>`                                                                                                                                                                                |
   | `concurrency`               | `concurrency\|dev:<n>` (per device; nested `streams[]` carry `stream_id`, no key)                                                                                                                             |
   | `slices` instance           | `slice\|<name>\|@<cpu_start_ns>`                                                                                                                                                                              |
   | `slices` aggregate          | `scope\|<name>` / `scope\|path:<path>` per `--group-by`                                                                                                                                                       |
   | `gaps` device               | `gap\|dev:<n>\|@<start_ns>` (default; cross-stream sweep)                                                                                                                                                     |
   | `gaps` stream               | `gap\|dev:<n>\|stream:<n>\|@<start_ns>` (`--scope stream`)                                                                                                                                                    |
   | `gaps` trace                | `gap\|@<start_ns>` (`--scope trace`; multi-GPU)                                                                                                                                                               |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lucifer1004/VeloQ](https://github.com/lucifer1004/VeloQ) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
