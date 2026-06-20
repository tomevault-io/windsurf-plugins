---
trigger: always_on
description: Generate a complete UVM testbench scaffold for a single IP from its spec documents (docx/pdf/md) and register table (xlsx/csv/md/IP-XACT). Use whenever the user asks to "generate a UVM tb", "build a verification environment", "scaffold a testbench", "create a UVC", "set up DV for this IP", or anything equivalent in Chinese ("帮我生成验证环境", "搭一个 tb", "做 UVM 环境", "做这个 IP 的验证"). Also trigger when the user points at a directory containing an IP spec and asks Claude to "verify it" / "test it" without nam
---


# gen-tb — UVM Testbench Scaffold Generator

## Purpose

Generate a directory-aligned UVM testbench for one IP from:

Built-in first-class buses: APB slave, AHB-Lite (slave or master),
AXI4-Lite (slave or master). Other single-channel request/response
buses (I2C, SPI, Wishbone, OBI, custom register buses, …) are handled
by a generic fallback that reuses the same layout, sim flow, and
`tb_api` surface — see §Scope below.

Inputs:

- behavioral spec: `*.docx`, `*.pdf`, `*.md`
- register table: `*.xlsx`, `*.csv`, markdown table, IP-XACT XML
- RTL in place, an external RTL path, or a generated stub

The result must compile, run `<ip>_sanity_test`, run
`<ip>_reg_access_test`, and provide two usage surfaces:

- DV persona: UVM tests, sequences, agent, monitor, RAL hooks
- DE persona: `tb_api::write/read/expect_reg` task-style BFM

## Scope

| Dimension | Built-in (high-quality scaffold) | Generic fallback | Planned |
|---|---|---|---|
| Bus | APB slave, AHB-Lite (slave or master), AXI4-Lite (slave or master) | any single-channel request/response bus the scaffold sub-agent can infer from spec + the three built-in exemplars (e.g. I2C, SPI, Wishbone, OBI, custom register buses) | AXI4 full |
| Simulator | VCS, Questa (vlog/vsim — static-only, see note), xrun (Cadence Xcelium) | VCS, Questa, xrun | — |
| Ref model | none, SV, C-DPI | none, SV, C-DPI | Python-DPI |
| Spec input | docx, pdf, md | docx, pdf, md | — |
| Reg input | xlsx, csv, md, IP-XACT | xlsx, csv, md, IP-XACT (optional — set `register_semantics: no` for non-register buses) | — |

Generic fallback is a best-effort path: directory layout and sim flow
stay authoritative, but the bus-shaped pieces (`<bus>_if.sv`, agent,
sequence library, `tb_api` body) are inferred per-IP by a constrained
scaffold sub-agent that pattern-matches on the three built-in
references. Tell the user up-front that scaffold correctness is lower
and the chance of an `unresolved.md` delivery is higher than in
built-in mode. Load `references/generic_bus.md` only when entering
this path.

AXI4 full (bursts/IDs/outstanding) is out of scope in both modes — it
must never be routed to generic. If the DUT has AXI ports, follow the
AXI4-full detector in Phase 1 and the mandatory question in Phase 2.
For DUTs that expose AXI signals but use only single-beat transfers,
generate the AXI4-Lite environment with a monitor-side assertion that
`AWLEN == 0 && ARLEN == 0`, so any later burst use fails loud at sim
time rather than relying on user review.

Do not use this skill for debugging an existing UVM environment, formal
verification, gate-level sim, DFT, or a multi-master SoC subsystem.

## Pipeline

Each phase writes audit artifacts under `<ip>/work/_gen_audit/`.
Do not collapse phases; resumability and auditability are part of the
deliverable.

1. Discover: identify IP, spec, regs, RTL, optional external VIP
2. Intake: ask only unresolved questions
3. Normalize: produce `registers.yaml`, `behavior.md`, `parse_report.md`
4. Scaffold: write the generated tree
5. Compile-fix: run compile and use a constrained sub-agent if needed
6. Sanity: run mandatory tests and runtime-fix loop if needed
7. Hand-off: write `CLAUDE.md`, `unresolved.md`, and concise summary

### Pipeline Hooks (Optional)

After each phase finishes writing its audit artifacts, call:

```bash
python3 scripts/hooks.py <phase> <ip_root>
```

with `<phase>` ∈ `{discover, intake, normalize, scaffold, compile_fix,
sanity, handoff}`. The script looks for
`<ip_root>/.gen_tb_hooks/<phase>[.strict].{py,sh}` and is a no-op when
absent (zero cost in the common case). Non-strict hook failures are
logged to `work/_gen_audit/hooks/<phase>.log` and do not block the
pipeline; `.strict.` hooks abort the phase on non-zero exit. Hooks
receive a JSON payload on stdin and `GEN_TB_PHASE` / `GEN_TB_IP_ROOT` /
`GEN_TB_AUDIT_DIR` in env. Do not invoke hooks before audit artifacts
for the phase are on disk.

## Phase 1: Discover

### IP Selection

Pick the IP root in this order:

1. Explicit IP name/path from the user
2. Current directory if its basename looks like an IP name
3. The only child directory containing spec files
4. Otherwise ask the user to choose

Show the selected IP name and path once so the user can correct it.

### Spec And Registers

Search `spec/`, `doc/`, `docs/`, then IP root. Classify:

- register table: `*reg*table*.xlsx/csv`, `*regs*.xlsx/csv`, IP-XACT XML
- behavior spec: large `*.pdf`, `*.docx`, `*spec*.md`
- markdown tables that look like regs: confirm if ambiguous

Do not ingest README, license, unrelated datasheets, draft/old/bak files.

### RTL

Prefer an existing filelist. Otherwise scan `rtl/`, `src/`, `design/`,
`hdl/`, and `<ip_name>/` for RTL, excluding tb/test/sim/bench paths.
If no RTL exists, generate a stub only when the user confirms.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [gokeshenzhen/gen-tb-skill](https://github.com/gokeshenzhen/gen-tb-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
