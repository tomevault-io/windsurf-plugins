---
trigger: always_on
description: JACE (Java Apple Computer Emulator) is a Java Apple II emulator with a scriptable terminal
---

# JACE — Guide for Agents

JACE (Java Apple Computer Emulator) is a Java Apple II emulator with a scriptable terminal
mode (`--terminal`) used to automate Apple II software testing.

**This file is an index.** Detail lives in `docs/jace/*.md` — read only the file you need.

| Read this | When you are |
|---|---|
| `docs/jace/commands.md` | Looking up any terminal command's syntax or semantics (the full reference) |
| `docs/jace/setup-and-disks.md` | Launching JACE, choosing a slot, or hitting a disk-image problem (incl. the cadius 146,432-byte patch) |
| `docs/jace/debugging-guide.md` | Debugging 65C02 code: `$FC` debug opcodes, breadcrumbs, breakpoints, failure diagnosis, Language Card switches |
| `docs/jace/automation-recipes.md` | Writing a full automation script; or need implementation/architecture internals |
| `docs/jace/applesoft.md` | Working with Applesoft BASIC (cold-start without a disk, **hello-world quickstart** — re-runnable test `./hello-world-test.sh`, tokenizing, variable table layout) |
| `docs/jace/assembly-quickstart.md` | Writing, assembling (ACME), or testing a 6502 assembly program in JACE (incl. the Apple-II-not-C64 notes) |
| `docs/jace/mixed-basic-assembly.md` | Combining Applesoft BASIC with embedded 6502 machine code via the DATA+POKE pattern (load raw bytes, CALL, memory layout) |
| `docs/jace/mixed-basic-assembly-advanced.md` | The mixed-BASIC/assembly pattern above but with multiple routines, parameter passing, or memory-management/reuse templates |
| `docs/jace/advanced-assembly.md` | Writing advanced 6502 assembly in JACE: the ACME compile pipeline, Apple //e memory mapping, lo-res/DHGR video memory model, YIQ/NTSC color, or choosing a zero-page strategy (cooperate with BASIC vs run outside it) |
| `docs/jace/unit-tests.md` | Running `mvn test` on JACE itself (not the emulator REPL) |
| `docs/jace/mockingboard.md` | Touching Mockingboard / AY-3-8910 sound emulation |
| `docs/jace/floating-point.md` | Calling the Applesoft FP ROM (FADDT/FSUBT/FMULTT/FDIVT/SQR/ATN/GETADR) from raw 6502 assembly, with macro patterns and debugging via `$FC` |
| `docs/jace/changelog.md` | Wanting the history of changes to JACE and to these docs |
| `examples/*/README.md` | Wanting a complete, verified, worked program (keyboard input + GETLN semantics: `roman-numeral`; DATA+POKE mixed BASIC/asm: `hello-world-mixed`; DHGR: `dhgr-color-wheel`, `dhgr-pinwheel`; lo-res: `cat-on-rug-lores`) as a starting template instead of writing one from scratch |

---

## Non-Negotiables

Read these five before doing anything. Each one has cost real debugging time.

1. **Always use Maven, never the native binary.** `/Users/brobert/Downloads/Jace` silently
   ignores `--terminal` and opens a window. All scripting goes through Maven.
2. **`run N` does NOT execute N cycles.** It free-runs for `N/1000` milliseconds with a
   **100 ms floor**, so any `N < 100,000` runs ~100,000+ cycles — up to 100x what you asked
   for. For exact stepping use `step`/`tick`; to stop at an address use `runto`; for frames
   use `runvbl`. See `docs/jace/commands.md`.
3. **Use slot 7 for ProDOS `.po` images.** Slot 6 emulates real floppy rotation — a ProDOS
   boot takes ~600 real seconds. Slot 7 (SmartPort) is instant.
4. **Always wrap invocations in `timeout`.** Programs under test hang; `timeout 90 ...` and
   check for exit code 124.
5. **JACE is an Apple II, not a C64.** No PETSCII, no high-bit ASCII, no $C64 memory map
   ($C000 screen does not exist). Plain 7-bit ASCII, text screen $0400–$07FF, console output
   via JSR $FDED. See `docs/jace/assembly-quickstart.md`.

## Booting a hard disk example

This boods a disk image using the SmartPort (note: this executes much faster than disk drive emulation so it's generally recommended unless you really want to debug RWTS routines)
After the boot sequence and startup program have run for a bit, a screenshot is recorded and the emulator exits.
```bash
cd ~/Documents/code/jace
timeout 90 mvn -q exec:java -Dexec.mainClass="jace.JaceLauncher" -Dexec.args="--terminal" <<'EOF'
bootdisk d1 /path/to/disk.po 7
run 5000000
screenshot /tmp/frame.png
qq
EOF
```

`mvn -q javafx:run -Djavafx.args="--terminal"` also works. `qq` quits; `qqq` terminates the JVM.

## The Commands You Will Actually Use

Full reference, including 20+ further commands, in `docs/jace/commands.md`.

| Command | Alias | Purpose |
|---|---|---|
| `reset` | - | Ensure the emulator is powered up and issue a cold start to put it in a known state |
| `bootdisk d1 <file> [slot]` | `bd` | Insert + reset + run until PC >= $2000 |
| `loadbin <file> <addr>` | `lb` | Load a binary straight into RAM (no disk needed) |
| `savebin <file> <addr> <size>` | `sb` | Dump memory to a file |
| `4000G` | — | Set PC and start executing (**async** — must follow with `run`/`expect`) |
| `step [count]` | `s` | Single-step instructions — **exact**, unlike `run` |
| `runto <addr>` | `rt` | Run until PC hits an address — exact |
| `runvbl` | `rv` | Advance to the next VBL edge; use before memory dumps |
| `run [count]` | `g` | Free-run for a while — approximate only, see caveat above |
| `loadbasic <file>` | `lbas` | Load a basic program from a text file into memory |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [badvision/jace](https://github.com/badvision/jace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
