---
trigger: always_on
description: Community-driven database of audio software, plugins, DAWs, and hardware for music production.
---

# Catalog

Community-driven database of audio software, plugins, DAWs, and hardware for music production.

## Tech Stack

- TypeScript, Node.js, pnpm
- YAML for data files, SQLite for distribution
- Zod for schema validation

## Key Commands

- `pnpm install` - Install dependencies
- `pnpm validate` - Validate all YAML data files
- `pnpm typecheck` - Run TypeScript type checking
- `pnpm build` - Build SQLite database
- `pnpm format` - Format YAML files with Prettier
- `pnpm format:check` - Check formatting

## Project Structure

- `data/` - YAML source files (manufacturers, software, content, hardware, accessories)
- `schema/` - Category, format, and platform definitions
- `scripts/` - Build and validation tools
- `dist/` - Built SQLite database output

## Conventions

- Filenames are slugs: lowercase with hyphens (e.g., `serum.yaml`, `massive-x.yaml`)
- All entries must pass `pnpm validate` before commit
- Use Prettier for YAML formatting
- Data follows strict schemas defined in `scripts/lib/types.ts`

## Data Entry Format

Manufacturers require: name, url
Software requires: name, manufacturer, primaryCategory, platforms
Content requires: name, manufacturer, primaryCategory
Hardware requires: name, manufacturer, primaryCategory, description
Accessories require: name, manufacturer, primaryCategory, description

Optional fields: categories (array), url, description

Note: Slugs are derived from filenames, not stored in the YAML files.

## Field Formatting Conventions

**`manufacturer`** must be a slug reference (the manufacturer's filename without `.yaml`), not the display name:

```yaml
manufacturer: hologram-electronics  # correct (slug)
manufacturer: Hologram Electronics   # wrong (display name)
```

**`description`** uses flow scalar format (Prettier auto-wraps long lines):

```yaml
description: Dream Sequence is a programmable pedal that uses sequencing,
  envelope shaping, and pitch shifting to create synth-like arpeggios.
```

**`details`** uses block scalar `|-` with paragraphs separated by blank lines:

```yaml
details: |-
  First paragraph of details text here.

  Second paragraph continues here with more information.
```

**`specs`** uses block scalar `|-` with `"- "` prefixed list items:

```yaml
specs: |-
  - Octave up and down pitch shifting
  - Pattern sequencer
  - Tap tempo
  - MIDI in/out
```

Do NOT use YAML arrays for `details` or `specs`. Do NOT use `|` (use `|-` to strip trailing newlines).

**Hardware I/O entries** use this field order with all fields present:

```yaml
io:
  - name: Audio Input
    signalFlow: input
    category: audio
    type: line
    connection: 1/4-inch
    maxConnections: 1
    position: Top
```

**IO field validation** uses a two-tier system:

- **Strict (errors, blocks CI):** `signalFlow`, `category`, `position`, `price.currency`
  - `signalFlow`: input, output, bidirectional
  - `category`: audio, midi, digital, power
  - `position`: Top, Bottom, Left, Right
  - `currency`: ISO 4217 codes (USD, EUR, GBP, etc.) — see `schema/currencies.yaml`
- **Advisory (warnings, non-blocking):** `type`, `connection`, `link.type`
  - Known values listed in `schema/io-types.yaml`, `schema/io-connections.yaml`, `schema/link-types.yaml`
  - Unknown values produce warnings in `pnpm validate` output
  - Add new values to schema files via PR when they're confirmed valid

**Semantic distinction:** `type` describes the signal characteristic (line, instrument, headphone, midi, usb, expression). `connection` describes the physical connector (1/4-inch, xlr, usb-c, 5-pin din). Don't swap them.

## Content Entries

Content entries (presets, sample packs, expansions) live in `data/content/` as a separate collection. Content `primaryCategory` values include `preset`, `preset-pack`, `sample-pack`, `drum-sample-pack`, `loop-pack`, `sound-library`, `soundfont`, `impulse-response`, and `multisample`.

- Content entries do NOT have `platforms`, `formats`, or `identifiers` fields
- Use `compatibleWith` to reference host software or hardware products by slug:

```yaml
name: Zeus Presets for Serum
manufacturer: some-vendor
primaryCategory: preset-pack
compatibleWith:
  - serum
```

```yaml
name: Haunted Hearts
manufacturer: elektron
primaryCategory: preset-pack
compatibleWith:
  - elektron-digitone
```

Advisory warning W123 fires if a `compatibleWith` slug doesn't match an existing software or hardware file. Category aliases in `schema/category-aliases.yaml` map common synonyms (e.g., `soundbank` → `preset-pack`) to canonical categories.

## Accessory Entries

Accessory entries (cables, stands, acoustic treatment) live in `data/accessories/` as a separate collection. Accessory `primaryCategory` values include `cable`, `power-conditioner`, `mic-stand`, `boom-arm`, `pop-filter`, `shock-mount`, `reflection-filter`, `acoustic-treatment`, `acoustic-panel`, and `windscreen`.

- Accessory entries do NOT have `io` or `variants` fields (use those for hardware only)
- Like hardware, accessories require: name, manufacturer, primaryCategory, description

## Product Lineage


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/existential-engineering) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
