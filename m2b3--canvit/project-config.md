---
trigger: always_on
description: `CLAUDE.md` is a symlink to this file.
---

# CanViT: guide for contributors and coding agents

`CLAUDE.md` is a symlink to this file.

Build software that is beautiful to read, difficult to misuse and easy to build
upon. The README is the product contract: what CanViT is, how to use it, and
what it reproduces. This file holds enduring engineering principles and the
conventions that are easy to get wrong, not work status or a catalog of
implementation details.

Coding agents do much of the development here and are among the code's main
readers. Design the tree, the APIs and the commands for how agents read, edit
and operate software.

## Layout

- `canvit-pytorch/`: the Python distribution `canvit-pytorch` (import
  `canvit_pytorch`). Public model classes are exported at the package top
  level and implemented in `.model`; subsystems are
  verbs: `canvit_pytorch.pretrain`, `.specialize` (probes, fine-tuning),
  `.evaluate` (benchmarks), `.viz` (recorded rollouts and viewpoint paths
  for the page and slides, and the model exported to run in the browser). Docs in
  `canvit-pytorch/docs/`; SLURM job scripts in `canvit-pytorch/slurm/`.
- `canvit-pytorch/tpu/`: a separate uv environment for ImageNet-1k
  fine-tuning on Cloud TPU (exact torch/torch_xla pins).
- `canvit-mlx/` and `canvit-nnx/`: native model packages with the same public
  model concepts and channels-last images. Each has its own uv environment.
- `canvit-core/`: framework-independent architecture definitions and shared
  checkpoint and preprocessing code. Backend packages depend on it.
- `canvit/`: the PyPI placeholder linking to the backend packages and homepage.
- `tests/backends/`: cross-framework contract tests; checkpoint conversion
  commands live in `tools/`.
- `site/`: the project page, published to https://m2b3.github.io/CanViT/ by
  `site/deploy.sh` as the single commit of the `gh-pages` branch.
  `site/record_bundles.sh` records its data with `canvit_pytorch.viz`;
  `site/check_paper_numbers.py` checks its numbers against the paper's
  generated macros. `site/AGENTS.md` holds the site's own conventions.
- `paper/`: the NeurIPS 2026 paper. `paper/latex/` holds its sources, `paper/exports/` the figures and data
  they read, `paper/exporter/` the pipeline that writes them from evaluation results (a uv project that
  depends on `canvit-pytorch/`). `paper/AGENTS.md` holds the paper's conventions.
- `.github/workflows/release.yml`: PyPI releases use
  `<package>-v<version>` tags matching the package's `pyproject.toml`.

A fact, constant or URL lives in one place and everything else points to it.

Place and name things by meaning, never by convenience.
A fact's home is the module whose name says what the fact is about: the path
a reader who has never seen the code would guess before searching. When no
such module exists, create it. The convenient alternatives (the module the
consumers already import, the file that needs no new import, the class that
happens to be nearby) make the edit easy and the design wrong, and every
later reader pays for it. If the reason for a placement, a name or an API is
about effort or about the current import graph ("already imported",
"closest", "avoids a new file"), the reason is the defect: stop and find the
right home.

Hardcoded copies of facts violate DRY. Facts about a
component (a policy's abbreviation and description, a checkpoint's geometry)
live in the component's own definition, and consumers read them from there; a
dict keyed by component names in some other module is a second copy that goes
stale without an error. The same holds for defaults: one declaration,
imported everywhere else.

Remove obsolete code, dead code, and code that the paper and its ablations
neither use nor reference. The released checkpoints and
the demos built on them count as uses; git history keeps what is removed.

## Design

Fight inconsistencies.

Aim for code that is obviously correct from its structure, not merely code
with no obvious defects. Types, names and assertions should carry as much of
the correctness argument as possible; review is the last line of defense.
Write for capable engineers: simplicity means fewer concepts and fewer ways to
get something wrong, not avoiding powerful language features.

- Keep computation in small, typed, pure functions where possible; isolate
  side effects at explicit boundaries.
- Seek the abstraction that makes a whole class of code unnecessary. Judge it
  by the code it eliminates and the future changes it makes straightforward.
  Avoid both repetitive plumbing and speculative generality.
- Challenge necessity first. Prefer deleting over simplifying, simplifying
  over optimizing, and optimizing over automating. All else equal, less code
  is better; the structure that remains should be beautifully designed.
  Leave sound code alone.
- Make invalid states unrepresentable: frozen dataclasses whose construction
  establishes invariants, `Literal` or enums for closed vocabularies,
  jaxtyping or asserted shapes for tensors, and the same checks when loading
  configs and checkpoints.
- Be suspicious of boolean switches, optional fields that only matter
  together, and modes that multiply branches. Model distinct alternatives as
  alternatives; do not force facts that coexist into an either-or type.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [m2b3/CanViT](https://github.com/m2b3/CanViT) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
