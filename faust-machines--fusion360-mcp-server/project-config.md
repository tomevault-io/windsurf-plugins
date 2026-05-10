---
trigger: always_on
description: This is an MCP server (81 tools) that connects AI coding agents to Autodesk Fusion 360 for CAD automation. It consists of two pieces:
---

# AGENTS.md

## Overview

This is an MCP server (81 tools) that connects AI coding agents to Autodesk Fusion 360 for CAD automation. It consists of two pieces:

1. **MCP Server** (this repo) — speaks MCP protocol over stdio, forwards commands to Fusion via TCP
2. **Fusion 360 Add-in** — runs inside Fusion, executes commands on the main thread via CustomEvent bridge

## How the system works

```
Claude Code ──stdio──> MCP Server ──TCP :9876──> Fusion Add-in ──CustomEvent──> Main Thread
                                   <──JSON──                    <──result──
```

The add-in uses a CustomEvent + work queue pattern to safely dispatch all Fusion API calls to the main thread. Socket threads submit work items and block on a per-item `threading.Event` until the main thread completes execution.

## Available tools (80)

### Scene & Query
| Tool | Description |
|------|-------------|
| `ping` | Health check (no Fusion API, instant) |
| `get_scene_info` | Design name, bodies, sketches, features, camera |
| `get_object_info` | Detailed info about a named body or sketch |
| `list_components` | List all components in the design |

### Sketching
| Tool | Description |
|------|-------------|
| `create_sketch` | New sketch on xy/yz/xz, optional offset |
| `draw_rectangle` | Rectangle in most recent sketch |
| `draw_circle` | Circle in most recent sketch |
| `draw_line` | Line in most recent sketch |
| `draw_arc` | Arc (center + start + sweep angle) |
| `draw_spline` | Fit-point or control-point spline |
| `create_polygon` | Regular polygon (3–64 sides) |
| `add_constraint` | Geometric constraint (coincident, parallel, tangent, etc.) |
| `add_dimension` | Driving dimension (distance, angle, radial, diameter) |
| `offset_curve` | Offset connected sketch curves |
| `trim_curve` | Trim at intersections |
| `extend_curve` | Extend to nearest intersection |
| `project_geometry` | Project edges/bodies onto sketch plane |

### Features
| Tool | Description |
|------|-------------|
| `extrude` | Extrude a sketch profile |
| `revolve` | Revolve a profile around an axis |
| `sweep` | Sweep a profile along a path |
| `loft` | Loft between two or more profiles |
| `fillet` | Round edges (all/top/bottom/vertical) |
| `chamfer` | Chamfer edges |
| `shell` | Hollow out a body |
| `mirror` | Mirror a body across a plane |
| `create_hole` | Hole feature on a body face |
| `rectangular_pattern` | Pattern in rows and columns |
| `circular_pattern` | Pattern around an axis |
| `create_thread` | Threads (cosmetic or modeled) |
| `draft_faces` | Draft/taper for mold release |
| `split_body` | Split a body using a plane |
| `split_face` | Split faces of a body |
| `offset_faces` | Push/pull faces by a distance |
| `scale_body` | Scale uniformly or non-uniformly |
| `suppress_feature` | Suppress a timeline feature |
| `unsuppress_feature` | Re-enable a suppressed feature |

### Body Operations
| Tool | Description |
|------|-------------|
| `move_body` | Translate a body by (x, y, z) |
| `boolean_operation` | Join/cut/intersect two bodies |
| `delete_all` | Clear the design |
| `undo` | Undo last operation |

### Direct Primitives
| Tool | Description |
|------|-------------|
| `create_box` | Box (via TemporaryBRepManager) |
| `create_cylinder` | Cylinder |
| `create_sphere` | Sphere |
| `create_torus` | Torus |

### Surface Operations
| Tool | Description |
|------|-------------|
| `patch_surface` | Patch surface from boundary edges |
| `stitch_surfaces` | Stitch surface bodies into one |
| `thicken_surface` | Thicken a surface into a solid |
| `ruled_surface` | Ruled surface from an edge |
| `trim_surface` | Trim a surface with another body |

### Sheet Metal
| Tool | Description |
|------|-------------|
| `create_flange` | Create a flange on an edge |
| `create_bend` | Add a bend |
| `flat_pattern` | Create flat pattern |
| `unfold` | Unfold specific bends |

### Construction Geometry
| Tool | Description |
|------|-------------|
| `create_construction_plane` | Offset, angle, midplane, 3-point, tangent |
| `create_construction_axis` | Two-point, intersection, edge, perpendicular |

### Assembly
| Tool | Description |
|------|-------------|
| `create_component` | Create a sub-assembly component |
| `add_joint` | Joint between two components |
| `create_as_built_joint` | Joint from current positions |
| `create_rigid_group` | Lock components together |

### Inspection & Analysis
| Tool | Description |
|------|-------------|
| `measure_distance` | Minimum distance between entities |
| `measure_angle` | Angle between entities |
| `get_physical_properties` | Mass, volume, area, center of mass |
| `create_section_analysis` | Section plane through model |
| `check_interference` | Detect collisions between components |

### Appearance & Parameters
| Tool | Description |
|------|-------------|
| `set_appearance` | Assign material appearance from library |
| `get_parameters` | List all user parameters |
| `create_parameter` | Create a new parameter |
| `set_parameter` | Update a parameter value |
| `delete_parameter` | Remove a parameter |

### Export
| Tool | Description |
|------|-------------|
| `export_stl` | Export body as STL |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [faust-machines/fusion360-mcp-server](https://github.com/faust-machines/fusion360-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
