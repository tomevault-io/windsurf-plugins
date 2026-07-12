---
trigger: always_on
description: MCP server that builds production-ready Figma using the design
---

# Mimic AI

MCP server that builds production-ready Figma using the design
system on the user's target file. Accepts prompts, HTML, or any
description. Uses real components, tokens, and auto-layout.
Learns from every build.

## Prerequisites

- **FIGMA_TOKEN** must be set in the MCP server config
  (or in `~/.mimic-ai.json` as fallback). Without it,
  Mimic cannot read the library's components or text
  styles. Generate one in Figma: avatar (top-left) →
  Settings → Security → Personal access tokens →
  "Generate new token" (name: "Mimic AI", 90-day
  expiration). Five scopes required (all read-only):
  `current_user:read`, `file_content:read`,
  `file_metadata:read`, `library_assets:read`,
  `library_content:read`. Mimic validates the token on
  every build and guides users through setup or renewal
  if missing/expired.
- **Library file key** is prompted once per library and
  cached permanently. The user copies it from the library
  file's URL: `figma.com/design/<this-part>/...`
- **Figma plugin must be running.** In Figma desktop:
  Plugins > Development > Mimic AI > Run. The bridge
  connects automatically on first tool call.
- **fileKey** is the alphanumeric string in the Figma URL:
  `figma.com/design/<fileKey>/...` — pass it to
  `mimic_discover_ds`.
- **Start every build with `mimic_status`.** It returns the
  current phase and tells you what to do next.
- **Don't guess variable paths.** Discovery populates the
  cache. Use `figma_read_variable_values` or
  `figma_list_text_styles` to see what's available.

## Component-First Principle

**Target ~90% DS component usage, ~10% primitives with DS
variables.** Every build should maximize DS component coverage.
Before creating ANY frame, check if the DS has a component for
it. Section-level elements (header, footer, sidebar) and UI
patterns (cards, metrics, tables, badges, buttons, inputs)
should ALWAYS be DS components. Only use `figma_create_frame`
for truly custom layouts that have no DS equivalent — and even
then, bind every property to DS variables and text styles.

After `mimic_discover_ds`, ALWAYS call `mimic_map_components`
with all section-level elements in the design. For any missing
components, search the library via Figma MCP
`search_design_system` before building custom frames.

## Build Protocol

Every build follows 6 phases in order:
0. Target → 1. DS Discovery → 2. Style Inventory →
3. Build → 4. QA → 5. Report

Call `mimic_status` to start. It returns the current state
and what to do next.

## Phase 1+2 — DS Discovery (TWO CALLS)

Discovery is a two-step process that ensures community
libraries are never missed:

**Step 1 — Plugin discovery:**
```
mimic_discover_ds(fileKey)
```
Discovers variables, text styles, and components via the
Figma plugin API. Caches everything and computes enforcement
profile. Stays at Phase 1 (NOT build-ready yet).

The response includes `communityLibraryCheckRequired: true`
and `_stopBuild: true`. Build tools are blocked at Phase 1.

If multiple DS libraries are detected by the plugin, discovery
STOPS with `_userPrompt`. Present the prompt to the user
EXACTLY as written, wait for their pick, then re-call with
`libraryKey`.

**Step 2 — Community library check:**
Call Figma MCP `search_design_system` with query `"color"`,
`includeVariables: true, includeComponents: false,
includeStyles: false` on the fileKey. Collect all unique
non-null `libraryName` values AND one sample variable `key`
per library from the results. Then:
```
mimic_discover_ds(fileKey, {
  communitySearchResults: ["LibraryA", "LibraryB", ...],
  communitySearchVariableKeys: {
    "LibraryA": "first-variable-key-from-results",
    "LibraryB": "first-variable-key-from-results"
  }
})
```
The tool validates which libraries are actually enabled in
the file (filters out phantom libraries from search). If
multiple real libraries remain, it returns a `_userPrompt`
with a lettered list. If only one, it auto-selects.

This check is **enforced by the tool** — build tools require
Phase 2, which only unlocks after community verification.

Check `completenessWarnings` in the response. If components
were not found on the page, use Figma MCP
`search_design_system` to find them by name.

After discovery, call `mimic_map_components` with the HTML
element types to get the exact component keys for the build.

**Component mapping workflow:**

**With FIGMA_TOKEN (recommended):** One call is enough.
REST API discovery caches ALL library components, so
`mimic_map_components({ elementTypes })` returns found
components + confirmed gaps immediately. Missing types
are real gaps — proceed to build with primitives.

**Without FIGMA_TOKEN (fallback — two calls):**
1. `mimic_map_components({ elementTypes })` — returns
   found + missing with search terms.
2. Search via Figma MCP `search_design_system`. One
   search per missing type.
3. `mimic_map_components({ elementTypes,
   librarySearchResults })` — confirms gaps.

**Community libraries (no file key available):**
If the selected library is a community file and the user
can't provide the library file key, re-call
`mimic_discover_ds` with `skipRestApi: true`. Discovery
proceeds with plugin-only data (variables + page-scan
components). Use Figma MCP `search_design_system` +

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [miapre/mimic-ai](https://github.com/miapre/mimic-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-12 -->
