---
trigger: always_on
description: >
---


# Draw Diagram Skill

Generate production-quality editable diagrams in **.drawio** (Draw.io / diagrams.net) or **.excalidraw** (Excalidraw) format.

## Quick Start

1. **Determine format** — Default to `.drawio` unless user requests Excalidraw. Draw.io is preferred for cloud/infra diagrams (has official AWS, Azure, GCP, K8s icon libraries). Excalidraw is preferred for hand-drawn-style sketches, whiteboard-style brainstorming, or when the user explicitly asks.
2. **Read the appropriate reference** before generating:
   - For `.drawio` → Read `references/drawio-format.md`
   - For `.excalidraw` → Read `references/excalidraw-format.md`
3. **Search for icon shapes** — Run `python scripts/find_icon.py <term>` to find the correct shape style string for cloud provider icons.
4. **Review examples** — Check `references/examples/` for complete working diagrams similar to the request.
5. **Apply the design tests** — Before generating, verify your plan passes the Isomorphism Test and Education Test (see Design Philosophy below).
6. **Generate the file** following the format spec exactly. For 10+ components, build section-by-section (see Building Large Diagrams).
7. **Run the render-view-fix loop** — This is mandatory, not optional. Validate structure, render to PNG, audit, fix, repeat (see Step 5 in Workflow).
8. **Run the Pre-Delivery Checklist** — Every item must pass before delivering.
9. **Save** — If the user specifies a save location, use that. Otherwise, save to the workspace root `.diagrams/` folder (create it if it doesn't exist).
10. **Export (optional)** — If user wants PNG/SVG/PDF: `python scripts/export_drawio.py <file.drawio> --format png` (embeds editable diagram in the exported file).
11. **List diagrams** — If the user asks to "show diagrams", "list diagrams", or "what diagrams do we have", check the `.diagrams/` folder in the workspace root.

## Format Selection Guide

| Use Case | Recommended Format |
|---|---|
| Cloud architecture (AWS/Azure/GCP) | `.drawio` |
| Kubernetes / networking | `.drawio` |
| System design / microservices | `.drawio` |
| UML / ER diagrams | `.drawio` |
| Quick whiteboard sketch | `.excalidraw` |
| Hand-drawn aesthetic | `.excalidraw` |
| User says "excalidraw" | `.excalidraw` |
| User says "draw.io" / "drawio" | `.drawio` |

## Workflow

### Step 1: Understand Requirements
- What system/architecture to diagram?
- Which cloud provider(s)? (AWS, Azure, GCP, multi-cloud)
- Level of detail? (high-level overview vs detailed with subnets/ports)
- Any specific components to include?

### Step 2: Look Up Icons & Review Examples
- Run `python scripts/find_icon.py <term> --provider aws` to find correct shape strings
- Check `references/examples/` for a similar diagram to use as a structural starting point
- Plan layout on a mental grid before writing XML/JSON

### Step 3: Plan Layout
- Identify all components/nodes
- Determine groupings (VPCs, resource groups, namespaces, zones)
- Plan edge connections and data flow direction
- Choose left-to-right or top-to-bottom flow
- Place connected elements near each other to minimize edge crossings

### Step 4: Generate
- Read the appropriate reference file (`references/drawio-format.md` or `references/excalidraw-format.md`)
- Generate valid XML/JSON
- Use proper cloud provider icon shapes where available
- Use containers/groups for logical boundaries
- Label all edges with protocols/actions where relevant
- Apply consistent color coding

### Step 5: Render-View-Fix Loop (Required)

This is the most important step. AI-generated diagrams almost always have visual problems on the first pass — overlapping labels, edges through boxes, unbalanced spacing. You MUST validate your own output before delivering.

**Minimum (always do this):**
1. Run `python scripts/render_diagram.py <file> --validate-only` — catches structural errors (duplicate IDs, broken edge refs, missing parent cells).

**Full visual validation (required for Excalidraw, strongly recommended for Draw.io):**
1. **Render** — Run `python scripts/render_diagram.py <file>` to produce a PNG screenshot.
2. **View** — Read the PNG to actually see the rendered diagram.
3. **Audit against vision** — Compare the rendered result to what you designed in Steps 3-4. Ask:
   - Does the visual structure match the conceptual structure you planned?
   - Does the eye flow through the diagram in the intended order?
   - Are containers/groups clearly distinguishable from their children?
   - Are all labels readable and not overlapping neighboring elements?
4. **Check for defects** — Look for these specific problems:
   - Edges crossing through unrelated boxes
   - Labels rendered as literal HTML tags (missing `html=1;`)
   - Text overflowing its container
   - Overlapping elements or cramped spacing
   - Orphaned elements with no connections
5. **Fix** — Edit the file to resolve any issues found.
6. **Repeat** — Re-render and re-check until the diagram passes both the vision check and the defect check.

If Playwright is not available, fall back to structural validation + the Pre-Delivery Checklist (below).

### Step 6: Save & Deliver
- Save to user-specified location, or default to workspace `.diagrams/` folder
- Present to user with instructions on how to open

### Step 7: Export (Optional)
If the user wants an image/PDF:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [alexpov/claude-skill-draw-diagram](https://github.com/alexpov/claude-skill-draw-diagram) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
