---
trigger: always_on
description: MCP server plugin for KLayout GUI — enables AI tools to control KLayout via MCP protocol over HTTP on `127.0.0.1:8765`. macOS only for now.
---

# KlayoutClaw

MCP server plugin for KLayout GUI — enables AI tools to control KLayout via MCP protocol over HTTP on `127.0.0.1:8765`. macOS only for now.

## Directory Structure

```
KlayoutClaw/
├── plugin/
│   ├── klayoutclaw_server.lym    # KLayout autorun macro (MCP server, v0.6)
│   └── klayoutclaw_ui.lym        # KLayout autorun macro (UI panel + status bar)
├── tools/
│   ├── gds_to_image.py           # GDS → PNG converter (gdstk + matplotlib)
│   ├── route_worker.py           # Subprocess routing engine (numpy/scipy/scikit-image)
│   └── evaluate_worker.py        # Configurable device design evaluation (gdstk)
├── skills/
│   ├── scripts/
│   │   └── mcp_client.py            # Shared MCP client for all skills
│   ├── geometry/
│   │   ├── SKILL.md
│   │   └── scripts/
│   │       ├── add_rect.py
│   │       ├── add_polygon.py
│   │       ├── add_path.py
│   │       ├── create_cell.py
│   │       └── add_instance.py
│   ├── display/
│   │   ├── SKILL.md
│   │   └── scripts/
│   │       ├── toggle_layer.py
│   │       └── show_only.py
│   ├── image/
│   │   ├── SKILL.md
│   │   └── scripts/
│   │       ├── add_image.py
│   │       ├── list_images.py
│   │       └── remove_image.py
│   ├── visual/
│   │   ├── SKILL.md
│   │   └── scripts/
│   │       └── capture.py
│   ├── nanodevice_flakedetect/
│   │   ├── SKILL.md              # Orchestrator (dispatches subagents)
│   │   └── scripts/
│   │       └── core.py           # Shared CV utilities (morph, contour, Chamfer)
│   ├── nanodevice_flakedetect_align/
│   │   ├── SKILL.md              # SIFT + Chamfer cross-substrate alignment
│   │   └── scripts/              # sift_align, source_contour, footprint, sweep, refine
│   ├── nanodevice_flakedetect_detect/
│   │   ├── SKILL.md              # Per-material segmentation (4 scripts)
│   │   └── scripts/              # graphite, graphene, bottom_hbn, top_hbn
│   ├── nanodevice_flakedetect_combine/
│   │   ├── SKILL.md              # Coordinate transforms + overlay
│   │   └── scripts/              # ecc_register, transform, overlay
│   ├── nanodevice_flakedetect_commit/
│   │   └── SKILL.md              # Insert polygons into KLayout
│   ├── nanodevice_flakedetect_review/
│   │   └── SKILL.md              # Visual validation protocol
│   ├── nanodevice_gdsalign/
│   │   ├── SKILL.md              # GDS template alignment orchestrator
│   │   └── scripts/
│   │       ├── extract_markers.py # Parse GDS L5/0 marker pairs
│   │       ├── detect_markers.py  # Template-match markers in image
│   │       ├── align_gds.py       # Compute similarity transform
│   │       └── commit_gds.py      # Warp image + contours, commit to KLayout
│   ├── nanodevice_e2e_design/
│   │   └── SKILL.md              # E2E device design methodology (device-agnostic)
│   ├── nanodevice_routing/
│   │   ├── SKILL.md
│   │   └── scripts/
│   │       ├── place_pads.py
│   │       ├── route_multiwindow.py
│   │       └── clear_routes.py
│   └── e2e_judge/
│       ├── SKILL.md              # Agentic E2E test harness with LLM judge
│       └── scripts/              # conftest, harness, judge, verifier, run_tests
├── agent/                          # qlaybot v0.4.4 — Pi-Agent SDK wrapper
│   ├── src/                        # TypeScript source (see agent/CLAUDE.md)
│   ├── tests/                      # 697 tests: unit / integration / e2e
│   ├── workspace/                  # Domain knowledge (SOUL, TOOLS, RULES)
│   ├── package.json
│   └── CLAUDE.md                   # Agent dev instructions
├── tests/
│   ├── test_connection.py        # Protocol-level MCP connection test
│   ├── test_connection.sh        # E2E connection test (install + launch + verify)
│   ├── test_phase0_func.py       # Phase 0: connection + geometry functional tests
│   ├── test_phase0_mcp.py        # Phase 0: MCP protocol tests
│   ├── test_phase1_mcp.py        # Phase 1: LYM server MCP tests
│   ├── test_phase1_worker.py     # Phase 1: route/evaluate worker tests
│   ├── test_phase2_phase3_func.py # Phase 2-3: skills + flakedetect functional tests
│   ├── test_phase4_docs_integration.py # Phase 4: docs integration tests
│   ├── test_phase4_mcp.py        # Phase 4: GDS alignment + routing MCP tests
│   ├── test_phase4_skills_func.ts # Phase 4: skills functional tests (TS)
│   ├── create_hallbar.py         # Hall bar creation via execute_script
│   ├── create_hallbar_unrouted.py # Hall bar with pin markers, no traces
│   ├── evaluate_gds.py           # Hall bar structural evaluation (gdstk)
│   ├── evaluate_routing.py       # Routing structural validation (gdstk)
│   ├── test_gdsalign.py          # GDS alignment pipeline tests (12 tests)
│   ├── test_hallbar.sh           # E2E Hall bar test (Claude + tmux)
│   └── test_autoroute.sh         # E2E autoroute test
├── docs/
│   ├── tools.md                  # MCP tool reference (10 tools)
│   ├── skills.md                 # Skills CLI reference (geometry, display, image, visual, nanodevice_flakedetect, nanodevice_gdsalign, nanodevice_routing, nanodevice_e2e_design)
│   ├── ui-plugin.md              # UI plugin architecture + pya Qt pitfalls
│   ├── plans/                    # Architecture design docs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [caidish/KlayoutClaw](https://github.com/caidish/KlayoutClaw) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
