---
trigger: always_on
description: Agentic CAD for 3D printing. A part is a Python function; a long-lived process
---

# nurb

Agentic CAD for 3D printing. A part is a Python function; a long-lived process
rebuilds it on save and pushes geometry to a browser without moving the camera.

Built on build123d (OCCT), so parts are real B-rep solids.

## The part contract

One convention carries the whole system:

```python
from nurb import *

@part
def dispenser(width=80.0, height=120.0, wall=2.0, draft=False):
    body = Box(width, height, wall)
    if draft:
        return body
    bed = body.bounding_box().min.Z
    keep = body.edges().filter_by(lambda e: e.bounding_box().min.Z > bed)
    return polish(body, keep, 1.0)
```

**Keyword defaults are the parameters.** That single declaration feeds the agent, the
CLI, the viewer's sliders, the tests, and any future configurator. Never add a
parallel `PARAMS` dict; the two would drift.

`draft` is optional and injected by the runtime, never passed by callers. When true,
skip the polish pass. Worth 20% on a real part, not the 18x a cube suggested: chamfers
are 23% of the gridfinity shelf's build.

The build is now nearly all of the loop. Tessellation used to look like the larger half
at 620ms, and Phase 4 found that almost all of it was one iterator in build123d rather
than any geometry; `builder._triangulate` reads the same triangles by index in 30ms.
Write a continuous dimension as a float (`chamfer_size=1.0`) and a count as an int
(`bracket_count=4`): the viewer reads the type of the default to decide whether that
parameter's slider steps by one.

## Commands

```
nurb new <name>      create parts/<name>.py and its card
nurb dev             watch, rebuild, serve the viewer on :7373 or the next free port
nurb build [part]    build once, report size and timing
nurb check [part]    run the printability rules, --strict for CI
nurb inspect [part]  faces, normals, concave edges, each finding on its face
nurb rules           print the design doctrine
nurb api             the vocabulary a part file gets, with signatures
nurb skill           print an agent skill file for any AI harness, --sync rewrites installed copies
nurb update          upgrade nurb, then re-sync the installed skill to match
nurb card [part]     regenerate a card's AUTO block
nurb verify [part]   the doctrine's verification list, --strict-ish exit code
nurb render [part]   write build/<part>.png, needs the render extra
nurb export [part]   write STL and STEP into build/, --formats for GLB
nurb extract         find duplication across sibling parts
nurb launcher        write viewer.command, a double-clickable `nurb dev`
```

`uv run pytest` runs the suite, which includes the parts in `examples/`.

A project is any directory containing `parts/`. There is no init step, and there
never should be.

## Layout

```
src/nurb/registry.py      @part, signature introspection
src/nurb/builder.py       load, build, tessellate, GLB
src/nurb/checks.py        printability rules, convexity, Finding/Context, variants
src/nurb/polish.py        the bisecting polish pass, and chamfer with real errors
src/nurb/probe.py         what `nurb inspect` measures, in the rules' own units
src/nurb/api.py           the vocabulary, derived from __all__ so it cannot drift
src/nurb/printers.toml    shipped printer profiles, named by a project's printer.toml
src/nurb/card.py          the card's AUTO block
src/nurb/extract.py       duplication across sibling parts, up to alpha-equivalence
src/nurb/measurements.py  measured(), and the refusal to guess
src/nurb/edit.py          writes slider values back into a part's keyword defaults
src/nurb/render.py        headless PNG, the only module that wants a browser
src/nurb/doctrine.md      the doctrine itself, shipped in the package
src/nurb/server.py        watcher, rebuild, HTTP + websocket on one port
src/nurb/viewer.html      three.js viewer, Z-up, camera persistence, sliders, section
src/nurb/vendor/three/    three.js r169, so the viewer needs no network
src/nurb/cli.py           command surface
examples/notch/           the real parts, which are also the calibration set
tests/test_notch_fit.py   the hanging interface, asserted for every configuration
tests/                    rules and examples, both cases per rule
```

## Rules

### This file is for developing nurb, not for using it

The part-design workflow (start `nurb dev` first thing, end every reply with the viewer URL, model while the user watches) belongs to the shipped skill (`src/nurb/agents.md`, mirrored into `skills/nurb/SKILL.md`) and applies in a user's parts project, never in this repo. Here you are building the tool. When verifying viewer or server changes, run `nurb dev` against `examples/notch` in the background and share the URL for that; `?part=<name>&variant=<name>` deep-links to the exact configuration you want looked at.

### Command names stay boring

The primary user is a language model. An agent that has never seen this tool can
guess `build`, `check`, `export`. It cannot guess a themed alias. Every clever name
is an indirection that degrades in a fresh context. The brand can be distinctive;
the interface cannot.

### Never port Fusion scaffolding

Notch's Fusion timelines contain constructions that exist only to work around a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shpigford/nurb](https://github.com/Shpigford/nurb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
