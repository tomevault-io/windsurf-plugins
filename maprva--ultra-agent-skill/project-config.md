---
trigger: always_on
description: >
---


# Ultra Query Skill

Ultra is a web application for making maps with MapLibre GL JS using data from OpenStreetMap
and other sources. It extends the classic Overpass Turbo concept with modern rendering, YAML-based
configuration, and support for many query providers beyond Overpass.

The user's goal is typically: "I want to query some OSM data and see it styled nicely on a map."
Your job is to produce a complete Ultra query document they can paste into Ultra and run.

## Output Format

**Always output Ultra queries directly in the chat as code blocks.** Do not create files, save
to disk, or use any file-creation tools for Ultra queries — even for long or complex ones.
The user needs to copy-paste the query into Ultra, and a code block in the chat is the most
convenient format for that. Use a plain fenced code block (triple backticks with no language
tag) so the entire query document (YAML frontmatter + query body) is easy to select and copy.

## Anatomy of an Ultra Query Document

Every Ultra query is a single text document with two parts:

```
---
(YAML frontmatter — configuration and styling)
---
(Query body — Overpass QL, SQL, GeoJSON, etc.)
```

The YAML frontmatter is optional. A bare Overpass query works fine on its own. But the frontmatter
is where the power lives — it controls the map style, query provider, popup behavior, and more.

## Default Behaviors

Follow these unless the user says otherwise:

- **Query provider**: Overpass QL. Set `type: overpass` only if auto-detection might be ambiguous;
  otherwise omit `type` since Overpass is auto-detected.
- **Bounding box**: Ultra provides template variables for the current map viewport that work
  across all query providers:
  - `{{bbox}}` — the standard Overpass format (south,west,north,east)
  - Individual values: `{{s}}`, `{{n}}`, `{{e}}`, `{{w}}` (short) or
    `{{south}}`, `{{north}}`, `{{east}}`, `{{west}}` (long)
  - Composite strings: `{{wsen}}` is equivalent to `{{w}},{{s}},{{e}},{{n}}`
  For Overpass, use `[bbox:{{bbox}}];` as the first statement.
  For other providers (QLever, Postpass, GeoJSON APIs), use the individual or composite
  shortcuts to embed viewport coordinates in your query.
  If the user names a specific region, use an `area` filter (Overpass) or a relation-based
  spatial filter (QLever `ogc:sfContains`) instead.
- **Basemap**: Don't include `extends` in the style unless the user asks for a specific basemap
  or the visualization requires sandwiching layers into an existing style (via `beforeLayerId`).
  When sandwiching is needed, a good default is
  `extends: https://styles.trailsta.sh/openmaptiles-osm.json`.
- **Styling**: Start minimal. Get the data on the map with a clean, readable style. Don't add
  elaborate color ramps, glows, or complex expressions unless the user asks. You can always
  iterate.
- **Output mode**: Choose the right Overpass `out` statement for the geometry type needed:
  - `out center;` for nodes or when you only need point locations of ways/relations
  - `out geom;` when you need the full geometry of ways (lines, polygons)
  - `out body;` + `>;` + `out skel qt;` for recursive descent (rarely needed in Ultra since
    `out geom` is simpler)

## Interpreting OSM Tags

OSM tag values often carry meaning beyond their everyday English definitions. The tagging
scheme grew organically from British English conventions and community consensus, so a
thoughtful approach is needed when translating user requests into queries.

**`highway=*` is a great example.** In OSM, "highway" means any public right of way — not
just major roads. The tag spans motor vehicle roads, footpaths, cycleways, and more:

| User intent | Typical OSM tags |
|-------------|------------------|
| "Roads" (motor traffic) | `highway` ∈ {`motorway`, `trunk`, `primary`, `secondary`, `tertiary`, `unclassified`, `residential`, `service`, `living_street`} |
| "Paths / trails" (foot) | `highway` ∈ {`footway`, `path`, `steps`, `pedestrian`} |
| "Bike infrastructure" | `highway=cycleway` or roads with `cycleway=*` tags |
| "Sidewalks" | `highway=footway` + `footway=sidewalk`, or `sidewalk=*` on roads |
| "Tracks" (agricultural/forest) | `highway=track` (note: *not* railroad tracks!) |

When a user asks for "roads," don't query all `highway=*` — that would include footways.

**Many other tags are confusing, for example:**

- `highway=unclassified`: a minor through-road (British term), *not* "unknown type"
- `natural=water`: can be used for any water bodies, including man-made
- `name:etymology=*` is confusing because `name:*=*` tags are usually for language codes, e.g. `name:fr=*`

## YAML Frontmatter Reference

All keys are optional. Here are the ones you'll use most:

### `style`

Controls MapLibre styling. Can be a URL to a full style, or an object with these keys:

- **`extends`**: URL to a basemap style to build on top of.
- **`layers`**: Array of MapLibre layer definitions for styling your query results.

```yaml
style:
  extends: https://styles.trailsta.sh/openmaptiles-osm.json  # optional
  layers:
    - type: circle
      paint:
        circle-color: red
        circle-radius: 5
```

### `type`

Force a specific query provider. Default is `auto` (auto-detected). Common values:
`overpass`, `postpass`, `qlever`, `geojson`, `sparql`, `ohsome`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MapRVA/ultra-agent-skill](https://github.com/MapRVA/ultra-agent-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
