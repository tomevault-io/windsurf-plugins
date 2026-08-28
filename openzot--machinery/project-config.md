---
trigger: always_on
description: This repository is a demonstration: a software factory that ships one working
---

# The machinery

This repository is a demonstration: a software factory that ships one working
machine every shift, unattended. You are the factory. Each run of
`orders/new-machine.yaml` is one shift; the workflow commits whatever you leave
in the working tree and publishes `site/` to GitHub Pages.

A machine is a control panel for something that does not exist - a lunar
regolith refinery, a 1960s telephone exchange, a kelp harvester on the sea
floor - with a live simulation behind it, faults you can trigger and recover
from, and the manual that teaches you to run it. The panel is the product. It
is judged by looking at it, and you will look at it before you are done.

## Layout

```
site/
  index.html            the catalogue page (reads machines.json; do not edit)
  machines.json         the catalogue - one entry per machine, append only
  machines/<slug>/
    index.html          the panel: structure only - links machine.css, loads machine.js
    machine.css         every rule the panel and the manual need
    machine.js          every line of behaviour; exposes window.machine
    manual.html         the operating manual, in the machine's own design
scripts/check.sh        validates the catalogue (static); must exit 0 before a shift ends
scripts/probe.sh        commissions the newest machine in a browser (dynamic); must exit 0 too
scripts/probe.js        what probe.sh runs; also takes the screenshots you look at
orders/new-machine.yaml the standing order you are running
```

## What is on the machine

No need to go looking: the shift installs the toolbox before you start.

- `node` v22 and `python3`. There is no `package.json` here and there must not
  be one - machines are vanilla and dependency-free.
- **Playwright with headless Chromium.** `require('playwright')` resolves from
  any directory. `scripts/probe.js` uses it; so can you, for anything the probe
  does not cover (drive a lever, read a gauge, screenshot a state).
- **The `view` tool.** You can look at images. The probe writes PNGs to
  `/tmp/machinery/`; view them. This is not optional - a panel is a visual
  object and the order requires you to look at it and change it in response.
- `prettier --check index.html machine.css machine.js manual.html`,
  `htmlhint index.html manual.html`, `quick-lint-js machine.js`,
  `node --check machine.js`. Point the linter straight at the file.

Scratch files - briefs, test scripts, screenshots, notes - go in `/tmp`. A
fifth file in the machine directory fails the check. The workflow commits
whatever is left in the working tree.

## Before you design: look at the shelf

1. Read `site/machines.json`.
2. For the three most recent entries, run
   `node scripts/probe.js <slug> --shots-only` and **view**
   `/tmp/machinery/<slug>-desktop.png`. You now know what the shelf looks like.
3. List ten candidates across different domains, eras and design languages.
   Strike anything that resembles an existing entry in kind, domain + era,
   design, interaction or colour. Build the one that is most unlike the rest.

## The design brief

Write it to `/tmp/brief.md` before any markup. It decides, in one line each:

| Decide | Examples |
| --- | --- |
| **The machine** | what it is, what it does, what goes wrong with it |
| **The place and the era** | a Baltic icebreaker's engine room, 1971; a Martian water plant, 2080 |
| **Who built the panel** | a shipyard, a state railway, a start-up, a monastery - it shows |
| **Chassis material** | hammer-finish grey enamel, brushed aluminium, bakelite, teak and brass, painted steel, moulded polymer, CRT glass |
| **Legends** | engraved and paint-filled, screen-printed, Dymo tape, stencilled, backlit film, etched |
| **Lighting model** | incandescent jewel lamps, neon, nixie, LED bars, CRT phosphor, vacuum-fluorescent, e-ink, reflective paint under room light |
| **Typography** | which system faces and how: condensed caps for legends, a narrow mono for readouts, a serif for the manual - and the letter-spacing, weight and case that make it period |
| **Palette** | chassis, accent, alarm - three hex colours, plus what else the machine needs; no two machines on the shelf share a colour story |
| **The signature control** | the one thing you operate that no other machine has: a patch bay, a pull chain, a dead-man handle, a chart recorder, a slide rule, a telegraph |
| **The layout logic** | why the controls sit where they sit - process flow left to right, a mimic diagram in the middle, start-up sequence top to bottom |
| **Sound** | none, or what: relays, a hum, a klaxon - after a gesture only |

Then build the panel that brief describes, and nothing else.

## Slop tells

These are rejected at review, which is to say: by you, when you look at the
screenshot. If you see one, fix it before you move on.

- A **dashboard**: a grid of rounded cards with a number in each. That is a
  web page about a machine, not a machine.
- **Glow-on-dark by default.** Dark chassis with neon-blue edges is one era's
  look (and the arcade's), not every machine's. A 1950s panel is grey enamel in
  daylight; a 2020s skid is white powder-coat with a touchscreen.
- **Emoji**, stock icon fonts, clip-art, or gradients that do not describe a
  material.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openzot/machinery](https://github.com/openzot/machinery) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
