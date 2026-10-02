---
trigger: always_on
description: generates the callers. **And a line break in a generated `value` is `&#10;`** - a raw one is
---

# Working in this repository

Rules that came out of building this server, written down so they survive a new machine and a
fresh session. They are not style preferences — each one is here because ignoring it cost real
work.

## Agents first

**LabVIEW work is DELEGATED to the matching `labview-*` agent, not done in the main session.**
The user's standing rule of 2026-09-27. Work directly only when the user asks for it in that
session.

| Request | Agent |
|---|---|
| a new VI | `labview-vi-generator` |
| implement a SUPPLIED VI whose front panel must be kept (exam template) | `labview-vi-generator` - pass the supplied VI; it grafts (Phase 6g) |
| change an existing VI | `labview-vi-editor` |
| a class, class hierarchy or interface | `labview-class-generator` |
| unit tests (default framework) | `labview-caraya-unit-test` |
| LUnit / VI Tester tests, only when named | `labview-lunit-unit-test` / `labview-vitester-unit-test` |
| documentation | `labview-doc-generator` |
| a DQMH module or event | `labview-dqmh-module` |

The reason is what the agent definitions carry: a session working directly never reads them, and
a rule that lives only there is invisible to it — twelve German comments and sixteen German
descriptions shipped exactly that way (see "Everything you write INTO a VI is English"). Relay an
agent's `NEEDS CLARIFICATION` block to the user verbatim and continue THAT agent via
`SendMessage`; never answer it on the user's behalf. Parallel agents share one LabVIEW, so the
project-open/close and swap rules under "ONE AGENT, ONE OUTPUT DIRECTORY" apply to every
multi-agent task.

## Generating LabVIEW code

**First decide what kind of thing you are looking for.** This routing question comes before any
lookup, and getting it wrong sends you to the wrong index and makes you conclude "there is no
function for this":

| What you want | Construct | Where to look |
|---|---|---|
| a **whole working diagram** — a state machine, a producer/consumer, "how do I stream to TDMS" | a shipping example to read and adapt | `lvai_example_index` |
| a computation on **data** — read a file, sort, parse, compare | primitive `Node`, or a subVI `Call` | `lvai_palette_index`; terminal names from an export |
| a **property or action of a LabVIEW object** — a VI, control, panel, project, the application | `Property Node` / `Invoke Node` | `lvai_vi_server_reference` |
| a **whole application skeleton** — producer/consumer, a dialog, a subVI stub | an NI `.vit` template, copied to a `.vi` | `templates\Frameworks\DesignPatterns\`; `docs/labview-vit-templates.md` |
| a **VI's icon** | neither — AIXML cannot carry one | `lvai_set_vi_icon`, which drives VI Server for you |

The second row is the one that gets forgotten. "Get this VI's icon", "list a project's items",
"is this VI broken", "read a control by name", "what does this VI call" are none of them functions
and will never appear in a palette — they are properties and methods, and the catalogue is the only
index for them.

**Check whether NI already built it, before designing anything.** `lvai_example_index` lists the
shipping examples of this installation with NI's own description and keywords, and needs no
running LabVIEW. It answers a different question from the palette index — that one says *which VI
may I call*, this one says *is this whole diagram already written*. Feed a hit's path to
`lvai_convert_vi_to_aixml` and read how NI wired it.

Two numbers worth knowing before you call it. **609 of the 951 examples are listed by default**:
the rest need LabVIEW FPGA, LabVIEW Real-Time or a licensed toolkit, and a hit you cannot open is
worse than no hit. The count held back is always reported; `includeSpecialised` shows them.
And the index is **cached on disk and warmed at start-up**, so calls cost about 176 ms — but the
first ever build on a machine reads 2510 files and takes **about 50 seconds**. This file used to
claim 400 ms flatly; that was the warm figure, and before the cache existed every server restart
brought the full minute back.

The cache never expires on its own. After installing or upgrading LabVIEW or an add-on, rebuild it
once — `refresh=true`, or `LabVIEWMCP --examples --refresh` — because nothing else will notice.
Every answer carries the cache's build date, so a stale index is visible rather than mysterious.

Do this first for anything pattern-shaped: state machines, producer/consumer, queued message
handlers, continuous acquisition, file streaming. `State Machine Fundamentals.vi` is thirty seconds
of reading and it is the canonical shape.

**Reading an example is cheap the second time; reading your own VI never is.** An export costs a
median of 331 ms, a p99 of 24 s and a worst case of 93 s, measured over 1677 VIs — and the time
goes on LabVIEW loading the VI, not on writing XML, so a big export is not a slow one (size and
duration correlate at r = 0.002). `lvai_convert_vi_to_aixml` caches exports of **installation**
VIs on disk under **`%USERPROFILE%\.labviewmcp\cache\aixml`** — the examples tree, `vi.lib`,
`user.lib` and every LVAddon. Your own code is deliberately never cached: an export depends on the
VI's subVIs too, and those change behind a caller whose own timestamp never moves. Every answer

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Zuehlke/labview-mcp](https://github.com/Zuehlke/labview-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
