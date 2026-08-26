---
trigger: always_on
description: - This repository develops a deterministic mechanical-design agent for FreeCAD. Keep changes focused on mechanical requirements, CAD working copies, FreeCAD integration, standard parts, design validation, engineering knowledge, and release infrastructure.
---

# AI Mechanical 3D CAD Design Agent project instructions

## Project scope

- This repository develops a deterministic mechanical-design agent for FreeCAD. Keep changes focused on mechanical requirements, CAD working copies, FreeCAD integration, standard parts, design validation, engineering knowledge, and release infrastructure.
- The project does not include rendering, video, or assembly-animation subsystems. Do not add media-production dependencies or workflows to the public agent unless the project scope is explicitly expanded and reviewed.
- Use the configured `mechanical-design` MCP for controlled design state and knowledge operations, and the configured `freecad` MCP for interactive inspection and CAD edits in the running FreeCAD GUI.
- The Mechanical Design Agent coordinates engineering state and evidence; it does not replace an engineer's approval or independently certify strength, safety, manufacturability, or standards compliance.

## Mechanical requirements and design discovery

- `superpowers:brainstorming` is an optional external workflow, not a project dependency or mandatory gate. When it is already installed and the user wants structured discovery, recommend it for complex, ambiguous, or new mechanical-design requests before geometry or implementation is proposed.
- Never install or configure Superpowers automatically. Its absence must not block project setup, CAD work, validation, or delivery; perform proportionate structured clarification directly in the conversation when it is unavailable or declined.
- For non-trivial new designs or behavioral changes, obtain user approval of the proposed design before implementation. Keep the ceremony proportional to the task, but do not treat an ambiguous prompt as authorization to invent engineering requirements.
- Establish, as applicable: intended function, operating sequence, units and coordinate system, dimensional envelope, interfaces, loads and duty cycle, motion and travel, materials, environment, applicable standards, standard parts, fits and tolerances, manufacturing constraints, maintenance needs, safety constraints, and acceptance criteria.
- Separate facts supplied by the user from derived values and agent assumptions. Report assumptions explicitly, and stop for direction when a missing choice would materially affect geometry, safety, compatibility, cost, or validation.
- Record explicit numeric requirements with units and validation tolerances. Never invent manufacturing tolerances or silently convert preliminary geometry into release-approved dimensions.

## macOS and Windows support

- Support both macOS and Windows as first-class development and runtime platforms. A feature is not cross-platform complete until its applicable automated tests and FreeCAD integration checks pass on both systems, or the remaining platform limitation is documented as a release blocker.
- Require Python 3.12 or newer. Keep Python code, configuration, package resources, database migrations, CLI behavior, and MCP tool schemas platform-neutral.
- Use `pathlib`, package resources, environment variables, and explicit configuration. Never commit hard-coded user home paths, macOS application paths, Windows drive letters, temporary directories, usernames, or machine-specific service locations.
- Resolve `FreeCADCmd` and FreeCAD GUI executables from explicit configuration first, then from reviewed platform-specific discovery. Current release acceptance targets official FreeCAD 1.1.3; support for another version requires a compatibility run rather than an unverified version claim.
- Do not rely on POSIX-only shell syntax, executable permissions, symlinks, or case-sensitive filenames in cross-platform product code. Use Python or dedicated platform adapters when behavior differs.
- Treat spaces, non-ASCII characters, Windows path separators, file locking, line endings, and UTF-8 encoding as normal supported conditions. Avoid shell interpolation of user-controlled paths.
- Keep PostgreSQL, Neo4j, and other local services bound to loopback interfaces. Use the documented Docker Compose/bootstrap path on both macOS Docker Desktop and Windows Docker Desktop unless a reviewed native deployment is explicitly selected.

## FreeCAD MCP operating rules

- Before reading or creating geometry, call `list_documents` or otherwise confirm that the local FreeCAD MCP bridge is connected.
- Prefer typed tools such as `create_document`, `create_object`, `edit_object`, and `get_object` for simple operations. Use `execute_code` only for bounded mutations that genuinely require the FreeCAD Python API.
- Keep the FreeCAD RPC bridge local. `remote_enabled` must remain `false` unless the user explicitly requests a separately reviewed remote-access configuration.
- Never read unrelated user files, start remote connections, install add-ons, or update pinned vendor sources as a side effect of a CAD task.
- Use `get_view` to inspect meaningful geometry changes. Confirm object names, placements, dimensions, shape state, and recompute results before advancing the design lifecycle.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Milkdomeda/ai-mechanical-3dcad-design-agent](https://github.com/Milkdomeda/ai-mechanical-3dcad-design-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
