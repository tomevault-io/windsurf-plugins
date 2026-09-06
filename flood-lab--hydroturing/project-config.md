---
trigger: always_on
description: Instructions for a coding agent asked to make an existing hydrologic model
---

# Building a HydroTuring sandbox

Instructions for a coding agent asked to make an existing hydrologic model
runnable under HydroTuring. A maintainer's agent evaluating a submission
end to end should start from `.claude/skills/hydroturing-evaluate-model/`,
which records the whole workflow and what the first evaluations taught. The work is always the same three files, and it
does not require understanding the model's internals.

## What you are building

```
models/<model-name>/
  model.yaml      declares what the model emits and needs
  Dockerfile      installs the model and the adapter
  ht_adapter.py   translates between the /io contract and the model's own API
```

## The contract

The harness exposes a small `/io` filesystem and runs your entrypoint once:

```
<entrypoint> --request /io/request.json
```

Paths inside `request.json` are relative to the request file's directory.

```
/io/request.json          read-only: opaque case id, model seed, timestep, n_steps, outputs
/io/input/forcing.csv     read-only: columns time, pr, tas, pet (mm/day, degC, mm/day)
/io/input/static.json     read-only: catchment attributes
/io/output/result.csv     write: one row per forcing row, spinup included
/io/output/run.json       write: {"status": "ok"}
```

The model seed is deterministic for reproducible stochastic inference, but it
is not the generator seed recorded in the host-side report. Exit 0 on success.
There is no network. Do not attempt to download weights or data at run time;
bake them into the image.

## The evaluation window

A submitted model is not run over the full generated record. The harness
cuts the record down to the largest flood event, located by the probe's own
reference model, with the full spinup in front of it: by default 30 days for
a daily model and 7 days for an hourly one, so a heavy model fits the probe's
time budget. `n_steps` in `request.json` is the length the adapter must
emit, spinup included; it is 395 rows for a daily model on the default
window, not the 4015 of the full record. Ask for a different window, or the
whole record, in `model.yaml`:

```yaml
window_days: 30      # or any positive number of days, or "full"
```

A probe may set a minimum window when its expectation only holds over a
longer stretch; the warming-response probe asks for a year. Reference
models always see the full record, because the acceptance gate is defined
on it.

## The timestep

`request.json` names the step the case runs at, as an ISO 8601 duration:
`PT1D`, `PT1H`, `PT15M`, `PT5M` or `PT1M`. Fluxes are rates in mm per day at
every step, so the depth moved in one row is the rate times the step
length, and a one-millimetre burst in one minute arrives as 1440 mm/day.
`model.yaml` declares the step the model was built at, and may list others
it also runs at, native step first:

```yaml
timestep: PT1D             # or a list: [PT1H, PT1M]
```

A resolution probe serves the same weather at several steps and runs every
model at its own step and at the next finer one, whatever the manifest
lists. It then measures how far the integrated runoff moved, in percent of
the rain. A model whose arithmetic assumes its native step moves a lot.

**Put learned parameters in the units of the step.** A model whose network
writes physical parameters (a recession coefficient per day, a percolation
cap in mm per day, a unit hydrograph in days) has to have those parameters
rescaled to the step the case runs at before its physics uses them, or the
physics drains its stores as many times too fast as there are steps in a
day, and the probe measures the adapter's units rather than the model. A
per-day fraction k becomes 1 - (1 - k)^dt, a per-day amount becomes
amount × dt, a hydrograph keeps its shape in days. The networks themselves
are fed what they were trained on, rates in mm per day; the physics is fed
depths per row. `models/dhbv2/ht_adapter.py` does this.

**Feed the model the rows at the step you were given.** Do not aggregate
hourly rows to days inside the adapter, or split days into hours, to reach
the step the model prefers. The adapter is the model's units layer, not a
resampler: it turns rates into the depths the model wants for the step it
was handed, and resampling would hide exactly the dependence on the step
that the probe exists to measure. On a single-step probe the manifest's
step is respected: a daily closure probe given an hourly-only model reports
INCOMPATIBLE, because closure is what it measures and it cannot measure it
on a model it cannot feed.

## Nothing from the future

The stress probes add one storm to a record and check that nothing the
model reports changes before it. Any path from later rows to earlier
output fails that: a bidirectional pass, a centred filter, attention over
the whole record, or statistics taken over the record you were handed,
such as normalising inputs by their mean or deriving catchment attributes
from all of it. If the model needs a climatology, take it from the first
year of the record and nothing later. Paired variants of one seed carry
the same `seed` in the request, so a stochastic model draws the same
numbers in both runs; keep it that way and do not reseed from the clock.

## Variable names and units

| Name | Meaning | Units |
| --- | --- | --- |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Flood-Lab/HydroTuring](https://github.com/Flood-Lab/HydroTuring) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-06 -->
