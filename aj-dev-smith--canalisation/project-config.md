---
trigger: always_on
description: A plant grown by simulating **auxin**, the hormone that tells plant cells where to
---

# Canalisation — a xenobotany engine

A plant grown by simulating **auxin**, the hormone that tells plant cells where to
become things. Runs in real time in a browser. One rule governs the whole project:

> **Nothing about the plant's shape is drawn.** No shape code, no outlines, no
> curves, no counts. Every form — where leaves go, the angle between them, the
> vein networks, the leaf silhouettes, petal number, fruit lobing — falls out of
> chemistry. If you find yourself writing a shape, you have taken a wrong turn.

The only spatial priors in the entire codebase are documented in
[docs/SCIENCE.md](docs/SCIENCE.md) under "What is imposed". Keep that list short.
Adding to it is a real cost and should be argued for, not slipped in.

## Read these before changing anything

| Doc | Why |
|---|---|
| [docs/SCIENCE.md](docs/SCIENCE.md) | The biology, the papers, what emerges vs what is imposed |
| [docs/TUNING.md](docs/TUNING.md) | Hard-won parameter regimes. **Read before touching any constant.** Hours of sweeps live here |
| [docs/PITFALLS.md](docs/PITFALLS.md) | Bugs that cost hours. Several will bite you again if you do not know them |
| [docs/JOURNAL.md](docs/JOURNAL.md) | Negative results, design forks and why they went the way they did |
| [docs/ROADMAP.md](docs/ROADMAP.md) | What is unfinished, ranked, with my recommendation |

## Build and run

```bash
node build.js            # concatenates src/*.js into canalisation.html
open canalisation.html   # no server needed, no dependencies
```

`canalisation.html` is a **build artifact** — never edit it. Source is `src/`,
numbered so the concatenation order is the dependency order. `build.js` strips
`import`/`export`, warns about duplicate top-level declarations (the bundle is one
shared scope — name collisions are silent otherwise and cost a debugging cycle),
and **compiles the bundle before writing it**, exiting non-zero if it does not
parse. It used to only warn, and the warning had a hole; PITFALLS.md has the day
that cost.

Tests are headless Node, no browser:

```bash
node test/smoke.mjs                                # structural invariants; the CI gate
node test/pattern.mjs '{"T":40,"D":6}' '{"G":0}'   # is the tissue patterning at all?
node test/phyllo.mjs                               # divergence angle stats
node test/margin.mjs                               # grow a leaf outline, ASCII silhouette
node test/fruit.mjs                                # grow fruits, ASCII radius map
node test/flower2.mjs                              # full life cycle incl. axillary flowers
node test/vein.mjs                                 # vein network + hierarchy ratios, ASCII
node test/lamina.mjs                               # blade at cell resolution: is there contrast to draw?
node test/species.mjs                              # grow every species, print what each one does
node test/whorl.mjs                                # floral organ identity — does q span its range?
node test/flower.mjs                               # one isolated axis: florigen, floralCount, fruit set
node test/focus.mjs '[{"tag":"a"}]'                # meristem probe: divergence, lock, primordium peak ratio
node test/ring.mjs                                 # T/D/geometry map on STATIC tissue, checked for stationarity
node test/shoot.mjs                                # senescence: does the specimen finish, and in what order
node test/senesce.mjs                              # senescence, drawn: does a dying blade change, and do the veins go last
node test/fall.mjs                                 # a shed blade: is the fall a falling plate, and do real blades differ
```

Two more are **archived experiments**, not live checks. They are the code that
produced the negative results in [docs/JOURNAL.md](docs/JOURNAL.md), kept so those
results stay reproducible. Both still run; neither should be read as a current
diagnostic:

```bash
node test/inhib.mjs 0 1     # falsified: a second inhibitor with its own length scale
node test/ring2.mjs 0 1     # falsified: confining initiation to a thin generative ring
```

Both take `<shard> <nshard>` so a long sweep can be split across processes.

`test/shoot.mjs` is both kinds at once. It checks the shipped senescence, and it
also reproduces a falsified hypothesis — abscission driven by auxin transport — by
switching the whole-plant stream on (`shootOpts.enabled`, off everywhere else).
The stream in `src/38_shoot.js` ships disabled for the same reason `rhoI: 0` keeps
the dead second inhibitor in `10_auxin.js`: **a negative result you cannot
re-measure is just a story.** Nothing in the running piece reads it.

**A harness can outlive the parameters it sweeps.** `test/sweep.mjs` was removed
because it swept two meristem options that no longer exist, so two thirds of its
grid was duplicate rows wearing distinct labels. If you add a sweep, assert the
knob still moves the number before trusting the table.

**Always test the science headlessly before touching the renderer.** A visual bug
and a simulation bug look identical on screen, and the headless harnesses give you
numbers in seconds instead of minutes.

`test/fall.mjs` is the one harness here that can **fail on the physics rather than

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aj-dev-smith/canalisation](https://github.com/aj-dev-smith/canalisation) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
