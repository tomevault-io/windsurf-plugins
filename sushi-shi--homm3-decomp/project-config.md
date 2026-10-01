---
trigger: always_on
description: Recover C++ that reproduces Heroes III Complete's retail MSVC 6.0 object code.
---

# HoMM3 matching guide

Recover C++ that reproduces Heroes III Complete's retail MSVC 6.0 object code.

## Evidence

- Retail bytes are authoritative: **English GOG Complete 4.0 (engine 3.2)**,
  `HEROES3.EXE`, fixed base `0x00400000`. The exact size and SHA-256 are in
  [README.md](README.md#pinned-target).
- Dreamcast's embedded debug symbols prove source facts for an older,
  cross-architecture build; x86 identities require retail proof.
- The pinned Classic Mac PowerPC PEF is a source reference for Windows
  reconstruction. Source `MAC_ADDRESS` claims pair its section-relative
  function offsets with authored bodies. Use retained helpers, calls and
  lightly optimized instructions to recover the Windows source structure.
  Mac is not a game target or an independent exact-matching objective.

Generated Windows symbol names describe the authored declarations; they cannot
independently prove retail parameter types, return types or access control.
Original Dreamcast decorated names are independent evidence. In particular,
Dreamcast CodeView primitive `0x20` can represent lowered `bool`, so a displayed
unsigned-byte type does not by itself prove `unsigned char` source. Check native
`_N` versus `E` mangling for interfaces. Where local variables have no such
evidence, treat bool/byte alternatives as hypotheses and compare retail codegen.

Windows is the game being rebuilt. Use Mac solely as evidence for recovering
the Windows source, particularly helper boundaries, source calls and function
structure hidden by VC6 optimization. Restore evidenced helpers in ordinary
Windows game headers/source and keep their call sites; a retained Mac call does
not require VC6 to retain that call. Exact bytes on both architectures strengthen
the reconstruction without proving a unique C++ spelling. A runnable Mac port
is not a project requirement. Use native library headers for Mac comparisons;
do not add duplicate game declarations or extracted header-body fragments.

For the broad helper sweep, cover byte-exact Windows functions too. Use Mac's
retained calls and simpler body shapes to restore helper calls and canonical
bodies throughout the source. Inspect the Mac callee and its callers to
distinguish game helpers from library, runtime, glue, or generated code. A
retained Mac call to an identifiable game helper, or a recognizable helper
body expanded in a Mac caller, is enough to restore that operation as a helper
call in corresponding Windows source callers. If the helper already exists,
replace equivalent direct field access or pasted logic with its call; if it
does not, add one canonical body and its calls. Mac's stripped executable need
not supply the helper's name: infer its operation from its body and callers,
then choose a clear project name. Do not remove or defer a supported helper
because its original spelling, exact Mac byte match, placement, or immediate
Windows score gain is unknown. Keep the best-supported ordinary header or
source placement and revise it when stronger evidence appears. A Windows
function that is already byte-exact still needs its supported helper calls.
Track each lead through implemented, already represented by a nested helper,
or a specific reason why the Mac target or corresponding Windows operation
cannot yet be identified. An uncertain name or placement is not such a reason.

The verdict is VC6 SP3 under Wine; clang/clangd is editor tooling only. Use the
per-TU compiler profiles in `config/units.toml`.

Use `homm3 build --fast <TU>` (for example, `homm3 build --fast cursor`) for the
inner loop. Normally supply the active TU so shared-header edits rebuild only
that TU during iteration. It reports the selected TU's per-function projected
MAX movements without banking them; unchanged-source CUR dips stay silent.
That TU's Mac pairs are scored from its full-TU CodeWarrior object in the
same loop.

For ordinary matching, improve the current Windows game function using native
evidence, run the targeted build, regenerate README with
`homm3 status update --write-readme` (also banking the measured scores), then
commit and push. This applies to workers too.
Do not run routine full builds, tests, standalone validation checks or broad
accounting passes. Inspect evidence and compiler differences as needed to solve
the current function; do not turn the diagnostic commands below into a checklist.
Keep canonical helpers and natural C++ throughout the matching loop.
Workers use separate worktrees and return their commits for integration; the
coordinator regenerates the combined README and pushes to the default branch.
A full `homm3 build` is available when explicitly requested for a broader
checkpoint; it is not a prerequisite for publishing a matching improvement.

When an interface recovery changes a compared symbol name, compile its owning
TU and refresh targets with `homm3 delink --unit <TU>`; repeat `--unit` for
multiple owners. This retains shared-header claim resolution while skipping
the global label self-test and completeness gate during ordinary matching.
Then compare the affected TUs with the usual fast build.

## Byte-matching evidence: DC source layout as well as statements

When byte matching a non-exact function, **inspect its source-line layout before

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sushi-shi/homm3-decomp](https://github.com/sushi-shi/homm3-decomp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
