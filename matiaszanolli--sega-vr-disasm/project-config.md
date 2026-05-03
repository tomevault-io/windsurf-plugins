---
trigger: always_on
description: Agent briefing for Virtua Racing Deluxe 32X disassembly/reassembly project.
---

# CLAUDE.md

Agent briefing for Virtua Racing Deluxe 32X disassembly/reassembly project.

**Last Updated**: March 16, 2026

## Agent Team (v3)

Two agents. See [agents/README.md](agents/README.md) for details.

**Worker** (Sonnet or Opus) does all technical work: research, code, build, test. Spawn directly — no intermediary. Use Opus for hard problems (B-004, B-006, novel COMM work), Sonnet for routine tasks.

**Auditor** (Opus) is a focused safety reviewer. Spawned fresh per concrete COMM/SH2/expansion proposal only. Returns APPROVED or BLOCKED. Not needed for 68K-only, profiling, or doc work.

**You are the task manager.** Pick a task from BACKLOG.md, spawn the Worker, review findings, spawn Auditor if flagged, approve/commit.

**Research-First Principle:** Before implementing any fix, read the relevant docs and build a mental model with citations. If a second attempt fails for related reasons, stop coding and read. Named anti-patterns: address shopping, circular investigation, modern platform assumptions, undocumented guessing — all banned.

**index.md maintenance rule:** After any session where a new pitfall is discovered or a new architectural fact is established, update `analysis/agent-scratch/oracle/index.md` before closing.

## Build & Test

```bash
make all          # Build 4MB ROM from disassembly
make clean        # Remove build artifacts
picodrive build/vr_rebuild.32x  # Test in emulator (PicoDrive only — BlastEm has NO 32X support)
```

Build produces `build/vr_rebuild.32x`. Binary compatibility with the original ROM is no longer maintained — the codebase is now actively modified for optimization and correctness.

## Ground Rules — STRICTLY ENFORCED

1. **Do Not Guess** — Use `docs/` (hardware manuals) and `analysis/` (architecture). Use `analysis/agent-scratch/oracle/index.md` as a topic lookup table. Read the primary source. Research first.
2. **Understand Before Modifying** — Never patch `dc.w` without understanding it. Disassemble and document first.
3. **Use Available Tools** — Profiler at `tools/libretro-profiling/`, disassemblers `tools/m68k_disasm.py` and `tools/sh2_disasm.py`. Measure, don't assume.
4. **Proper Assembly** — Modify assembly source, not raw binary. Convert `dc.w` to mnemonics when possible (see [KNOWN_ISSUES.md](KNOWN_ISSUES.md) for SH2 exceptions).
5. **Clean Commits** — No stale comments, no partial changes. Revert completely if something doesn't work.
6. **Verify Changes** — `make clean && make all` after every modification. Confirm build succeeds.
7. **DRY** — Never create duplicate files. Fix in place. Use git branches for experiments.
8. **COMM Register Safety** — Before modifying any 68K↔SH2 communication code, read [analysis/COMM_REGISTERS_HARDWARE_ANALYSIS.md](analysis/COMM_REGISTERS_HARDWARE_ANALYSIS.md). Critical rules:
   - **Read-during-write = undefined** — not just write-write. Both CPUs must use handshakes.
   - **SH2 writes are buffered** — value may not reach the register immediately. Dummy-read the same address to force synchronization.
   - **COMM1 is a system signal register** — func_084 manages COMM1_LO bit 0 ("command done"); V-INT, scene init, frame swap all poll it. Never write arbitrary data to COMM1 without save/restore + interrupt disable.
   - **COMM7 is the Slave doorbell** — never broadcast game command bytes to it (proven crash, B-006).
   - **Always use cache-through** (`$20004020`) for SH2 COMM access, never `$00004020`.
9. **Memory Boundaries** — Always check the [hardware manual memory map](docs/32x-hardware-manual.md) before assuming an address is accessible from another CPU.
   - **SH2 CANNOT access 68K Work RAM** (`$FF0000`) at ANY address. The region between SDRAM (`$0203FFFF`) and Frame Buffer (`$04000000`) is unmapped. Three failed B-003 attempts proved this.
   - **SDRAM mapping**: SH2 `$0600xxxx` = ROM file offset `$20000 + xxxx`, NOT `$xxxx`. Getting this wrong produces hex dumps of 68K code instead of SH2 SDRAM.
   - **Shared memory options** (exhaustive): COMM registers (16 bytes), SDRAM ($02000000-$0203FFFF), Frame Buffer ($04000000, FM-controlled). That's all.
10. **SH2 Patching Discipline** — SH2 code is tightly interconnected. Careless patches cause silent corruption.
    - **Literal pool sharing**: `MOV.L @(disp,PC),Rn` instructions share literal pools. Before overwriting ANY address in SH2 code, scan the entire section for `$Dnxx` opcodes that resolve to it (see [KNOWN_ISSUES.md](KNOWN_ISSUES.md) §SH2 Literal Pool Sharing).
    - **Test patches in isolation**: Interacting patches hide root causes. Never combine multiple SH2 patches without testing each one alone first (proven in B-006: reverting Patch #2 alone was insufficient).
    - **Verify encodings**: Always verify assembled SH2 opcodes against original ROM bytes with `python3`. Subtle encoding errors (wrong register field, wrong displacement) are invisible until runtime.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [matiaszanolli/sega-vr-disasm](https://github.com/matiaszanolli/sega-vr-disasm) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-21 -->
