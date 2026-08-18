---
trigger: always_on
description: You are working in a **coupled conceptual-design toolkit**: SUAVE does the physics, nTop does the
---

# CLAUDE.md - agent guide for ntop-suave

You are working in a **coupled conceptual-design toolkit**: SUAVE does the physics, nTop does the
geometry, and the two are wired into a fixed point. The SV-1 rocket under `examples/` is a
reference example, not the purpose of the repo. The purpose is the coupling.

Read this file fully before writing code. Then read `docs/REFERENCE.md`. Both exist so you do not
have to rediscover things that cost hours to find.

---

## 1. Orientation, in one minute

```
rocketgen/
  config.py          THE CONTRACT. Every module imports its dataclasses from here.
  ntopgen/           nTop side: author a notebook, run it, parse what it measured
    universe.py        load the block/type universe, resolve signatures and revisions
    recipe.py          typed builder that emits nTop recipe JSON
    driver.py          ntopcl process driver: convert, templates, run, parse outputs
    rocket_notebook.py the SV-1 parametric notebook (the reference example)
  sizing/            SUAVE side
    atmosphere.py      cached US Standard 1976
    aero.py            component drag and normal-force build-up
    propulsion.py      three-phase solid motor
    trajectory.py      3-DOF RK4 mission integrator
    masses.py          group-weight statement with provenance per line
    loop.py            THE COUPLING. converge_point() and size()
  doe.py             factorial and Latin hypercube trade studies
  report/            scripted figures and the reportlab assembly
run_sv1.py           staged driver: --stage smoke | size | doe | all
scripts/bootstrap.py fetches SUAVE, locates the nTop block universe
```

There are now TWO reference examples, and the second one is where most of the hard-won knowledge
lives:

| | SV-1 | IV-1 |
|---|---|---|
| Configuration | single body, ogive-cylinder-boattail, cruciform fins | two stages, strakes, interstage, divert pack |
| Mission | air launch, cruise, terminal dive | vertical canister launch, pitchover, staging, lofted intercept |
| Exercises | the coupling itself | mass leaving mid-flight, reference area changing at separation, vortex lift |
| Modules | `config.py`, `sizing/*.py`, `ntopgen/rocket_notebook.py` | `config_iv1.py`, `sizing/*_iv1.py`, `ntopgen/stack_notebook.py` |
| Driver | `run_sv1.py` | `scripts/iv1_converge.py` |

**IV-1 is built as PARALLEL modules, not as changes to the SV-1 ones.** That is deliberate. SV-1 is
the regression baseline: its 296 tests are the contract that says the shared physics still works.
When you add a third vehicle, do the same. Share code by importing or by lifting a function to
module level in the original, never by editing the original's behaviour.

The data flow is a loop, not a pipeline:

```
  design vector
      |
      v
  [1] masses.build_masses  ---------------------------+
      |                                              |
      v                                              |
  [2] ntopgen.rocket_notebook.measure_rocket        | measured volume, wetted area,
      |   (ntopcl builds the solid and measures it)   | cavity volume, CG, inertia, S(x)
      v                                              |
  [3] sizing.aero.RocketAero  <---------------------+
      |
      v
  [4] sizing.trajectory.Mission.fly
      |
      v
  [5] constraint residuals -> new design vector, back to [1]
```

`sizing/loop.py::converge_point` is that loop. Start there when you need to understand anything.

---

## 2. Setup

```bash
uv venv --python 3.11 .venv
uv pip install --python .venv/Scripts/python.exe -e ".[dev]"
uv run --python .venv/Scripts/python.exe scripts/bootstrap.py
```

`bootstrap.py` fetches SUAVE and locates the nTop block universe. Neither is committed. Run
`scripts/bootstrap.py --check` any time to see what is and is not working.

**The dependency pins are not negotiable.** SUAVE 2.5.2 breaks on all three current majors:

| Pin | Why |
|---|---|
| `numpy<2` | SUAVE fails on numpy 2.x |
| `scipy<1.14` | SUAVE imports `scipy.integrate.cumtrapz`, removed in 1.14 |
| `setuptools<81` | SUAVE's bundled `pint` imports `pkg_resources`, removed in 81 |

If you "fix" a dependency warning by upgrading one of these, the whole repo stops importing.

Run tests with:

```bash
.venv/Scripts/python.exe -m pytest tests -q
```

The full suite takes about 5 minutes because parts of it drive real `ntopcl` subprocesses.

---

## 3. Hard rules

These are the rules the repo was built under. Keep them.

### 3.1 No invented numbers

Every empirical constant, coefficient and material property carries a source comment **and** an
entry in a module-level `SOURCES` dict passed to `config.register_sources`.

If a value is a guess, the word **`GUESS`** must appear in its source string. There are tests that
assert this. The engineering report prints every flagged entry in a table, so a guess that hides
becomes a guess that ships.

```python
SOURCES = {
    "aero_base_drag": "Fleeman, Tactical Missile Design, 2nd ed., Chapter 2, Figure 2.16",
    "my_new_factor": "GUESS: no source found for this; 0.85 chosen to match the trend",
}
register_sources(SOURCES)
```

Never quietly widen a tolerance or nudge a constant to make a test pass. Fix the model, or record
the discrepancy.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bradrothenberg/ntop-suave](https://github.com/bradrothenberg/ntop-suave) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-18 -->
