---
trigger: always_on
description: >-
---


# Fireworks Tech Graph

Generate production-quality SVG technical diagrams exported as PNG via `cairosvg` (recommended), `rsvg-convert`, or `puppeteer`.

## Install Source

Install this skill from GitHub:

```bash
npx skills add yizhiyanhua-ai/fireworks-tech-graph
```

Public package page:

```text
https://www.npmjs.com/package/@yizhiyanhua-ai/fireworks-tech-graph
```

Do not pass `@yizhiyanhua-ai/fireworks-tech-graph` directly to `skills add`, because the CLI expects a GitHub or local repository source.

Update command:

```bash
npx skills add yizhiyanhua-ai/fireworks-tech-graph --force -g -y
```

## Helper Scripts (Recommended)

Four helper scripts in `scripts/` directory provide stable SVG generation and validation:

### 1. `generate-diagram.sh` - Validate SVG + export PNG
```bash
./scripts/generate-diagram.sh -t architecture -s 1 -o ./output/arch.svg
```
- Validates an existing SVG file
- Exports PNG after validation
- Example: `./scripts/generate-diagram.sh -t architecture -s 1 -o ./output/arch.svg`

### 2. `generate-from-template.py` - Create starter SVG from template
```bash
python3 ./scripts/generate-from-template.py architecture ./output/arch.svg '{"title":"My Diagram","nodes":[],"arrows":[]}'
```
- Loads a built-in SVG template
- Renders nodes, arrows, and legend entries from JSON input
- Escapes text content to keep output XML-valid

### 3. `validate-svg.sh` - Validate SVG syntax
```bash
./scripts/validate-svg.sh <svg-file>
```
- Checks XML syntax
- Verifies tag balance
- Validates marker references
- Checks attribute completeness
- Validates path data

### 4. `test-all-styles.sh` - Batch test all styles
```bash
./scripts/test-all-styles.sh
```
- Tests multiple diagram sizes
- Validates all generated SVGs
- Generates test report

**When to use scripts:**
- Use scripts when generating complex SVGs to avoid syntax errors
- Scripts provide automatic validation and error reporting
- Recommended for production diagrams

**When to generate SVG directly:**
- Simple diagrams with few elements
- Quick prototypes
- When you need full control over SVG structure

## Workflow (Always Follow This Order)

1. **Classify** the diagram type (see Diagram Types below)
2. **Extract structure** — identify layers, nodes, edges, flows, and semantic groups from user description
3. **Plan layout** — apply the layout rules for the diagram type
4. **Load style reference** — always load `references/style-1-flat-icon.md` unless user specifies another; load the matching `references/style-N.md` for exact color tokens and SVG patterns
5. **Map nodes to shapes** — use Shape Vocabulary below
6. **Check icon needs** — load `references/icons.md` for known products
7. **Write SVG** with adaptive strategy (see SVG Generation Strategy below)
8. **Validate**: Run `python3 -c "import xml.etree.ElementTree as ET; ET.parse('file.svg')"` to check XML syntax
9. **Export PNG**: Use `cairosvg` (recommended). See **SVG → PNG Conversion** section below for full method comparison
10. **Report** the generated file paths
11. **(Optional) Visual self-review** — if your runtime can read images, load the exported PNG back and inspect it. Syntactic validity does not guarantee visual correctness: arrows may cross through component interiors, labels may collide with lifelines or other labels, boxes may overlap, alt-frame text may sit on top of a message, or a legend may cover content. If you see any of these, revise the SVG and re-export; repeat until the rendered image is clean. Common fixes:
    - Route arrows through gaps between boxes, not through box interiors
    - Move arrow labels 6-8px away from the arrow line (offset-first); add background rects only when offset is insufficient
    - Widen inter-row/inter-column gutters so same-layer arrows have clear corridors
    - Collapse repeated cross-layer arrows into a single "delegates down" rail outside the content area
    - Move legend/notes out of any region where arrows or labels land
    - Increase viewBox height/width rather than packing elements tighter
    - If a filtered element (drop-shadow, blur) is missing one side of its border, move it ≥30px away from that viewBox edge, or remove the filter and rely on color/contrast for visual separation
  Skip this step silently if image reading is unavailable — do not guess.

## Diagram Types & Layout Rules

### Architecture Diagram
Nodes = services/components. Group into **horizontal layers** (top→bottom or left→right).
- Typical layers: Client → Gateway/LB → Services → Data/Storage
- Use `<rect>` dashed containers to group related services in the same layer
- Arrow direction follows data/request flow
- ViewBox: `0 0 960 600` standard, `0 0 960 800` for tall stacks

### Data Flow Diagram
Emphasizes **what data moves where**. Focus on data transformation.
- Label every arrow with the data type (e.g., "embeddings", "query", "context")
- Use wider arrows (`stroke-width: 2.5`) for primary data paths
- Dashed arrows for control/trigger flows
- Color arrows by data category (not just Agent/RAG — use semantics)

### Flowchart / Process Flow
Sequential decision/process steps.
- Top-to-bottom preferred; left-to-right for wide flows
- Diamond shapes for decisions, rounded rects for processes, parallelograms for I/O
- Keep node labels short (≤3 words); put detail in sub-labels

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yizhiyanhua-ai/fireworks-tech-graph](https://github.com/yizhiyanhua-ai/fireworks-tech-graph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
