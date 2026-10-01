---
trigger: always_on
description: Rules for contributors: how the repo is laid out, the conventions every node
---

# ComfyUI-AusBoss contributor guide

Rules for contributors: how the repo is laid out, the conventions every node
follows, and the checks to run.

A suite of polished ComfyUI custom nodes by ausboss. Public nodes must solve a
repeated workflow need, keep a compact graph footprint, and pass backend plus
browser acceptance before release.

## READ FIRST: write for the people who use the nodes

Everything a user reads starts with the plainest version. That covers node
descriptions, tooltips, the `?` help pages (`js/docs/*.md`), the README,
example workflow notes, report and error text, and the CHANGELOG. Get more
technical later in the text, or in its own section.

- Open with what it does and when to use it, in everyday words and short
  sentences. Someone new to ComfyUI should get it on the first read.
- A tooltip is one or two short sentences: what this is, and what to pick.
- Measurements, test results, edge cases and how it works go further down,
  under a heading such as "Technical details".
- Keep jargon out of the opening (resample, warp, frame, affine,
  estimator, canvas space). If a term is needed, say what it means the
  first time.
- Put a number up top only when it helps someone decide what to do.

If a sentence needs a second read, rewrite it.

## Which nodes ship

A node ships only when its mapping key is listed in `PUBLIC_NODE_IDS` in
`scripts/validate_nodes.py`; the validator fails on any registered key that
is not listed there.

## Hard rules

- Never modify `LICENSE`.
- Never bump `version` in `pyproject.toml` — a version bump that lands on
  main **publishes to the Comfy Registry automatically** (see Releasing).
- Keep diffs minimal: touch only the lines the task needs.

## Third-party independence

- Never copy third-party code, assets, fonts, icons, CSS, or documentation.
- Review ecosystem overlap before accepting a public node. Generic overlap is
  fine, but implementation, naming, interaction design, and documentation must
  be this repository's own work.

## Architecture

```text
__init__.py       # NODE_MODULES list → importlib merge of all mappings.
                  # Fail-soft: a broken module logs and is skipped, the rest load.
nodes/
  node_<name>.py  # exactly one node (or one tight family) per file;
                  # exports NODE_CLASS_MAPPINGS + NODE_DISPLAY_NAME_MAPPINGS
  _<topic>_helpers.py  # shared backend logic, underscore prefix = not a node
js/
  <name>/index.js # frontend entry per node or pack-wide feature, e.g.
                  # appearance/ (.js files auto-load)
  shared/*.mjs    # import-only shared modules (.mjs files do NOT auto-load)
docs/             # developer docs
scripts/          # offline checks in stdlib Python. validate_nodes.py is
                  # the entry point; registry_contract.py holds the rules
                  # that keep nodes visible to registry scanners;
                  # registry_status.py reads the Registry API; dev/ is the
                  # Node.js canvas harness (docs/live_testing.md).
example_workflows/  # example workflows (regular workflow JSON, not API JSON)
```

## Conventions

- Public mapping keys use `AUSBOSS_NODES_<Purpose>`. The mapping key is the
  workflow-compatibility contract and must never be renamed after release.
- Inputs and outputs only grow after a release: append an input as
  optional, append an output, and never rename, reorder, remove or newly
  require one. Saved workflows keep widget values by position and links by
  slot, and an API prompt must carry every required input.
  `tests/test_node_api.py` holds the pack to `tests/fixtures/node_api.json`;
  refresh that snapshot with the test's `--update` after a compatible change
  or a new node.
- Appending an input to a node with a card or panel takes one more step.
  Workflows saved by earlier releases end that node's values with an empty
  value for each card and panel, and values come back by position, so the
  new input opens holding that empty value. Give a widget input a
  `resetUnknown` fallback in its card (`js/widget_cards/index.js`), or a
  `RESET_UNKNOWN` beside `NODE_CLASS` in the node's own `js/<name>/index.js`
  if it has no card (as LoRA Loader does). The fallback equals the node's
  default, which is what an API prompt without the input runs with. Then
  list the input in `tests/saved_widget_values.test.mjs`. Cards and
  panels themselves are never saved: set `widget.serialize = false` right
  after `addDOMWidget` (`options.serialize` only keeps a widget out of the
  prompt).
- Write those keys as **string literals** inside `NODE_CLASS_MAPPINGS` and
  `NODE_DISPLAY_NAME_MAPPINGS` — never a `NODE_ID` variable. Registry scanners
  (ComfyUI-Manager) AST-parse the source without importing it, so a variable
  key makes every node invisible and "install missing custom nodes" stops
  offering the pack. `scripts/validate_nodes.py` enforces this.
- Assign each mapping **once**, at module level, to a non-empty dict literal,
  and never mention the name again — no `update()`, no `del`, no
  `alias = NODE_CLASS_MAPPINGS`. A scanner reads that one literal and stops,
  so anything done to the mapping afterwards is invisible to it. Both
  mappings must carry exactly the same keys.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ausboss/ComfyUI-AusBoss](https://github.com/ausboss/ComfyUI-AusBoss) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
