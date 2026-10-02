---
trigger: always_on
description: An MCP server for the inside of an Office file: the VBA in Excel, Word,
---

# Working on xlide_mcp

An MCP server for the inside of an Office file: the VBA in Excel, Word,
PowerPoint and Access, and the document around it. Visual Basic 6 projects open
the same way.
[README.md](README.md) says what it does. This says how to change it.

Adopt these first:

- `F:\GitHub\RIDM_Recursive_Invariant_Discovery_Model\RIDM.MD`
- `F:\GitHub\AI_Best_Practices\docs\agentic_ai_programming_best_practices.md`
- `F:\GitHub\AI_Best_Practices\docs\ai_smells_for_agents_to_avoid.md`
- `F:\GitHub\AI_Best_Practices\docs\ui_ux_guidelines_for_agents.md`

## The one rule that shapes everything else

**Python is normative. Every other implementation ports from it.**

`python/` tracks the upstream libraries. A sibling directory per language tracks
`python/`. Changes flow one way: implement in Python, regenerate the contract, and
the diff under `contract/` is the work list for every port. A port that fixes
something Python has wrong fixes it in Python too, or the fix is lost at the next
sync.

This is not a preference about languages. The server is a thin layer over four
libraries that hold the measured knowledge of the Office file formats:

| Library | What it knows |
|---|---|
| [pyOpenVBA](https://github.com/WilliamSmithEdward/pyOpenVBA) | How to read and write VBA, UserForm designs and Power Query inside the containers. |
| [pyOfficeEditor](https://github.com/WilliamSmithEdward/pyOfficeEditor) | The document surface: cells, formulas, formatting, tables, validation, rows and columns. |
| [pyVBAanalysis](https://github.com/WilliamSmithEdward/pyVBAanalysis) | 165 diagnostics, each measured against its host's object model. |
| [pyVBAharness](https://github.com/WilliamSmithEdward/pyVBAharness) | How to run VBA in desktop Office without wedging on a dialog. |

The split between the first two is the file itself: pyOpenVBA edits the VBA
project, pyOfficeEditor edits the document it lives in.

A port reimplements the *server*. It does not reimplement that knowledge, and it
is never the place a format discovery lands: that belongs upstream, and reaches
here through a version bump.

### One debt against that rule, mostly paid

`shapes.py` and `xlsx.py` held format knowledge this server should not own: the
worksheet grid, and the drawing layer where a button keeps the macro it runs.
Both were ported from XLIDE because no library reached either.

**The grid is done.** pyOfficeEditor covers it, so `cells.py` is a thin adapter
over it and the tools read and write through that. The swap was provable rather
than hopeful, which is the point of the corpus: every cell conformance case
passed before and after, unchanged. What the adapter owns is translation, not
format knowledge - an integer where the file stores a double, a leading `=` on a
formula, a JSON-safe date - because those are answers the corpus pins.

**Writing shapes is done, but for one write.** pyOfficeEditor 0.3 adds, removes
and repoints shapes, and keeps a Forms control's four parts in agreement while it
does, so shape writes go through it. The same proof held: every shape
conformance case passed before and after the swap. The one left here is the
macro on a Forms control Excel 2007 saved, which lives only in VML, where
pyOfficeEditor does not look; it goes with the reader.

**Reading shapes still has one legacy gap.** pyOfficeEditor 0.4 now reports the
anchor cells, alt text, hidden state and ActiveX controls, and supplies a public
`cell_origin` for placing a shape at a cell. It does not list a Forms control
saved only in VML by Excel 2007. A regression fixture proves this: its shape list
contains only the ordinary drawing shape after the DrawingML twins are removed.
The reader in `shapes.py`, with `xlsx.py` under it, still reads that control and
its macro; it merges pyOfficeEditor's control state, position and chart series
by name. Once the VML-only control is covered upstream, migrate all reads to
pyOfficeEditor and delete the reader and `xlsx.py` together. The other gaps
were [pyOfficeEditor#4](https://github.com/WilliamSmithEdward/pyOfficeEditor/issues/4).

Until then, nothing new goes into either.

## Layout

```
contract/          tool-surface.json, conformance.json   generated, normative
docs/porting.md    how a port is built and verified
python/            the reference implementation
  tests/           the suite; the ones marked live need Windows with Office
  tools/           the two contract exporters
<language>/        a port
```

Inside `python/src/xlide_mcp/`:

```
server.py         builds the MCPServer and registers every tool group
instructions.py   what the calling model is told at initialize
config.py         settings, and the workspace roots that bound every path
paths.py          resolving a caller's path, or refusing it with the reason
hosts.py          extension -> host, and what can be done with each
project.py        the VBA project: modules, kinds, guarded saves
tokens.py         content tokens, the guard on a stale write
textual.py        the file as text: what a diff of it reads
cells.py          worksheet cells, through pyOfficeEditor, and its formula engine
xlsx.py           the OOXML package surface the shape reader still needs
grid.py           the same, through Excel, for the formats that are not OOXML

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [WilliamSmithEdward/xlide_mcp](https://github.com/WilliamSmithEdward/xlide_mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-02 -->
