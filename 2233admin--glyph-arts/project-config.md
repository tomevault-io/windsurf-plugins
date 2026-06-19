---
trigger: always_on
description: glyph-arts -- terminal-visible chart toolkit. All chart types directly in the CLI -- no files, no GUI. incplot-style auto plotting (JSON/JSONL/CSV/TSV inference), plotext (kline/candlestick/line/scatter/step/bar/multibar/stackedbar/hist/heatmap/box/indicator/event/confusion/plotext overlays), SDR spectrum/waterfall, Diagon-style diagram (math/sequence/tree/table/frame/flowchart/GraphDAG/GraphPlanar), Mermaid/beautiful-mermaid-style diagrams, rich (table/tree/panel/gauge/pie/dashboard/rich_live),
---


# Glyph Arts Skill

## Vision

When AI lives in the terminal, visualization must live there too.
No browser. No generated files. No context switch.
glyph-arts gives the AI a native sense of sight inside the terminal.

## Invocation

```bash
SKILL=~/.claude/skills/glyph-arts
python $SKILL/scripts/chart.py <type> [--json '<data>'] [--file path.json] \
  [--duckdb 'SQL'] [--db PATH] \
  [--title 'T'] [--width N] [--height 20] [--theme pro] \
  [--sample N] [--xlabel X] [--ylabel Y] \
  [--xlim MIN MAX] [--ylim MIN MAX] \
  [--xscale linear|log] [--yscale linear|log] \
  [--orientation vertical|horizontal] [--output FILE] [--no-color] \
  [--link-data URL] [--link-title URL] [--statusline]

python $SKILL/scripts/chart.py animate <line|bar|scatter|sparkline> \
  --duration SEC --frames N --json '<data>'

python $SKILL/scripts/chart.py record demo.cast --cmd 'glyph-arts art DEMO' --duration 10
python $SKILL/scripts/chart.py record-replay demo.cast --output demo.gif

python $SKILL/scripts/chart.py to-hyperframes \
  --json '[{"label":"x","x":[1,2,3,4,5],"y":[10,20,15,30,25]}]' \
  --frames 30 --duration 5 --output-dir ./hf-demo
```

Chat drawing front door:

```bash
python $SKILL/scripts/chart.py chat image --file photo.jpg --width 80 --height 30
python $SKILL/scripts/chart.py chat photo.jpg --image-style retro-art
python $SKILL/scripts/chart.py chat incplot --json 'name,value\nA,3\nB,7'
python $SKILL/scripts/chart.py chat sdr spectrum --json '{"freq":[99.0,99.3],"power":[-93,-42]}'
python $SKILL/scripts/chart.py chat sequence --json 'Client->Server: GET /health'
python $SKILL/scripts/chart.py chat mermaid --json 'graph LR\nA[开始] --> B[完成]'
python $SKILL/scripts/chart.py chat diagram note --json 'NOTE\nCache refresh required'
python $SKILL/scripts/chart.py chat textplot --json '{"expr":"sin(x) / x","xmin":-20,"xmax":20}'
python $SKILL/scripts/chart.py chat turtle --json '{"commands":[["forward",30],["right",90],["forward",20]]}'
python $SKILL/scripts/chart.py chat graph --json 'Client -> API\nAPI -> DB'
python $SKILL/scripts/chart.py chat formula --json '\int exp(-x^2) dx = \sqrt{\pi}'
python $SKILL/scripts/chart.py chat formula-pretty --json '(a+b)/(c+d)'
python $SKILL/scripts/chart.py chat effects
```

`chat image` implies `--chat`; chart commands imply `--no-color`.

**stdin pipe (for large data):**
```bash
echo '<json>' | python $SKILL/scripts/chart.py <type>
cat data.json  | python $SKILL/scripts/chart.py line
```

**file input:**
```bash
python $SKILL/scripts/chart.py bar --file path/to/data.json
```

**check dependencies:**
```bash
python $SKILL/scripts/chart.py --check-deps
```

Width defaults to terminal width (`$COLUMNS`). Override with `--width 120`.

---

## Decision Tree -- Which Chart to Use

```
What is your data shape?
|
+-- Time series / sequence
|   +-- Unknown JSON/CSV/TSV shape     -> incplot
|   +-- Trend over time              -> plotext line
|   +-- Volume/count per period      -> plotext bar
|   +-- OHLC financial data          -> plotext kline
|   +-- Sparse scientific signal     -> uniplot
|
+-- Distribution / proportion
|   +-- Parts of a whole (<12 slices) -> rich pie
|   +-- Frequency distribution        -> plotext hist
|   +-- Two variables correlated      -> plotext scatter
|
+-- Comparison across categories
|   +-- Side-by-side bars             -> plotext bar (grouped -> multibar)
|   +-- Ranked list with scores       -> rich table
|   +-- Performance gauge (0-100)     -> rich gauge
|
+-- Hierarchy / tree structure        -> rich tree
|
+-- Density / matrix
|   +-- 2D correlation matrix         -> plotext heatmap
|   +-- RF spectrum trace             -> spectrum
|   +-- RF waterfall history          -> waterfall
|
+-- Structure / diagram text
|   +-- Formula / sequence / tree     -> diagram (Diagon backend when installed)
|   +-- Flowchart / DAG / planar graph -> diagram
|
+-- Continuous curve / function       -> drawille curve / textplot / turtle
|
+-- Multiple metrics at once          -> rich dashboard
|
+-- Inline sparkline (1 row)          -> sparkline
|   +-- Claude Code statusLine        -> sparkline --statusline
|
+-- Progressive reveal in terminal    -> animate line/bar/scatter/sparkline
|
+-- MP4/WebM/high-quality video       -> to-hyperframes, then npx hyperframes render
|
+-- Network / graph topology          -> graph
|
+-- Large ASCII label / banner        -> banner
|
+-- Styled text art / framed wordmark -> art
```

## Claude Code CLI Compatibility

Trigger keywords:

- `Claude Code statusline`, `statusLine.command`, `single-line metric` -> use `sparkline`, `indicator`, or `gauge` with `--statusline`.
- `OSC 8`, `terminal hyperlink`, `clickable title` -> add `--link-title URL`; for line/scatter series labels add `--link-data URL`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [2233admin/glyph-arts](https://github.com/2233admin/glyph-arts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
