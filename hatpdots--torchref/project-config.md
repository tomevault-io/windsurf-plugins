---
trigger: always_on
description: Working notes for AI agents (and humans) editing TorchRef. This file is the authority on
---

# AGENTS.md

Working notes for AI agents (and humans) editing TorchRef. This file is the authority on
conventions; where `.github/instructions/*.md` disagrees with it, this file wins.

TorchRef is GPU-accelerated **crystallographic refinement** built on PyTorch: models are
`nn.Module`s, targets are losses, and gradients come from autograd. The domain is X-ray
crystallography — structure factors, Miller indices, space groups, ADPs, R-factors — not
machine learning for its own sake and not structural biology. Screen every change against
crystallographic convention (units, sign of phases, Å and Å², d-spacing vs resolution).

---

## 1. Environment

Use the project's own virtual environment / conda environment — never a bare `python` or
`python3`. Machines that host this repo usually carry several other interpreters with
TorchRef's dependencies at the wrong versions, and those fail subtly (wrong numbers) rather
than loudly. If you don't already know which interpreter is the right one, find it before
running anything: look for a `.venv`/`.env` beside the repo, check `which -a python`, or
confirm with `python -c "import torchref, torch; print(torchref.__file__, torch.__version__)"`.

**Probe for a batch scheduler before deciding how to run tests.** Don't assume either way:

```bash
command -v sinfo && sinfo -s     # SLURM present? which partitions, what's idle
```

- **No scheduler** (laptop, workstation, CI container): just run the command normally.
- **SLURM present**: read `sinfo` output and pick a partition from what actually exists and
  has idle nodes — partition names are site-specific, so never hardcode one. Then use
  `srun -c 8 --pty …` for short interactive work and `sbatch -c 8 …` for anything long.
  Keep off the login node for real work, and don't request a GPU node unless the change
  touches a CUDA/Triton path (`sinfo` also tells you whether GPU nodes exist and are free).
  Check `squeue -u "$USER"` before queuing more.

---

## 2. Hard rules

### 2.1 The whole package runs in float32

**If float64 is needed for a result to be correct, the formula is wrong — fix the formula.**

Single precision is the design constraint, not a performance compromise. MPS has no float64
at all, and CUDA float64 throughput is a small fraction of float32 on the hardware this runs
on, so a float64 dependency makes a code path unusable on half the supported devices.

Practically:

- **Never hardcode a dtype.** Take it from the config: `torchref.config.get_float_dtype()`,
  `get_int_dtype()`, `get_complex_dtype()`, or from an input tensor. Roughly 200 call sites
  already do this; follow them.
- `torch.float64` *is* a supported configuration (`TORCHREF_DTYPE_FLOAT=float64`) used as an
  eager numerical reference and in gradient checks. Code must **work** in float64, must not
  **require** it, and must not silently downcast (see `tests/integration/test_dtype_config_float64.py`).
- When you hit precision trouble, reformulate: log-space accumulation, `logsumexp`,
  `log1p`/`expm1`, shift-and-center before squaring, Kahan/pairwise sums, subtract the mean
  before a covariance, factor out the dominant scale. Cast to `.double()` only as a
  last resort, and then only for a small, contained, non-differentiable block (a 3×3 eigen
  solve, a matrix exponential) — with a comment saying which conditioning problem forces it.
- NumPy interop at I/O boundaries (gemmi, reciprocalspaceship, OpenMM) is naturally float64;
  that is fine. Convert at the boundary, not deep inside a kernel.
- Integers default to **int32**, complex to **complex64**. Note the MPS trap: `scatter_reduce`
  `amax`/`amin` on int64 fails there — use `index_add_` or a stable `argsort` for grouped
  reductions.

### 2.2 Code does not track its own history

Git and `docs/changelog.rst` record what changed. Source files describe what **is**.

Do not write, in code or docstrings:

- `# NEW:`, `# CHANGED:`, `# FIXED:`, `# was: ...`, `# previously we did X`
- "This replaces the old `foo()`", "as of 0.6.2", "after the refactor", "legacy path"
- Commented-out former implementations kept "for reference"
- Benchmark numbers from a one-off run ("3.2× faster than before", "took 4.1 s on 1DAW")
- Dated notes, initials, TODO owners, or audit-cluster references

Two narrow exceptions:

1. A **public deprecation** the user must act on — use a `Warnings`/`.. deprecated::` block
   or `DeprecationWarning`, stating the replacement, not the history.
2. A constant whose value was chosen empirically may state **the criterion and the
   conclusion** in one or two lines (e.g. "3.0 is the floor: 2.5 degrades the F-residual by
   20× on the worst test case") — never the full sweep table.

When you change something, put the note in `docs/changelog.rst` under the current version
heading, one line, user-facing.

### 2.3 Docstrings: NumPy style, on every public thing

Rendered by Sphinx with `napoleon` (`napoleon_numpy_docstring = True`,
`autodoc_typehints = 'description'`). Google style is off and will render wrong.

A docstring answers *"how do I call this and what will it do to me?"* for someone who will
**not** read the body. Document the contract, plus any trap that produces a silently wrong
result.

```python
def compute_structure_factors(

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HatPdotS/TorchRef](https://github.com/HatPdotS/TorchRef) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
