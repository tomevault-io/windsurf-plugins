---
trigger: always_on
description: **When implementing specs in `.specify/specs/`, invoke the SpecFlow skill.**
---

# Supertag CLI - Claude Code Context

## SpecFlow Development

**When implementing specs in `.specify/specs/`, invoke the SpecFlow skill.**

This project uses spec-driven development. Specs are stored in `.specify/specs/<feature-id>/` with:
- `spec.md` - What and Why (requirements)
- `plan.md` - How (technical design)
- `tasks.md` - Work items with TDD enforcement

**Triggers:** "work on F-XXX", "implement F-XXX", "/specflow"

## Documentation Locations

**When releasing a new version, update ALL of these:**

| File | Purpose | Location |
|------|---------|----------|
| `CHANGELOG.md` | Internal version history (detailed) | `./CHANGELOG.md` |
| `README.md` | Technical documentation, CLI usage, MCP setup | `./README.md` |
| `SKILL.md` | PAI skill documentation with USE WHEN triggers | `./SKILL.md` |
| `CHANGELOG.md` | Public release notes (customer-facing) | `~/work/web/invisible-store/tana/CHANGELOG.md` |
| `USER-GUIDE.md` | Customer-facing user guide | `~/work/web/invisible-store/tana/USER-GUIDE.md` |
| Marketing description | Store listing and marketing copy | `~/work/web/invisible-store/tana/index.html` |

## Release Checklist

**IMPORTANT: Update CHANGELOG.md BEFORE running release.sh**

1. Update `CHANGELOG.md` - Change `[Unreleased]` to `[X.Y.Z] - YYYY-MM-DD`
2. Update version number in `package.json`
3. Run `bun run typecheck` - Ensure TypeScript type checks pass
4. Run `bun run test:full` - Ensure all tests pass
5. Run `./release.sh X.Y.Z --push` to build, tag, and push
6. Update public `CHANGELOG.md` at `~/work/web/invisible-store/tana/CHANGELOG.md`
7. Update store listing if features changed

**Note:** The release script updates `package.json` version automatically if you pass a version argument. Step 2 can be skipped if using `./release.sh X.Y.Z`.

## PR Checklist

**IMPORTANT: Run these checks before pushing a PR:**

1. Run `bun run typecheck` - TypeScript types must pass
2. Run `bun run test` - Fast tests must pass (1741+ tests)
3. Push and verify CI passes (GitHub Actions runs full test suite)

## Key Architecture

- **Main CLI**: `supertag` - query, write, sync, server, workspaces
- **Export CLI**: `supertag-export` - Playwright browser automation
- **MCP Server**: `supertag-mcp` - AI tool integration via Model Context Protocol

## Important Technical Notes

### Tana Input API Inline References

**Two ways to create references via Input API:**

1. **Inline reference in text** (within node name or child text):
   ```html
   <span data-inlineref-node="NODE_ID">Display Text</span>
   ```
   Example payload:
   ```json
   {"name": "Meeting with <span data-inlineref-node=\"abc123\">John Doe</span> today"}
   ```

2. **Child reference node** (entire child is a reference):
   ```json
   {"children": [{"dataType": "reference", "id": "NODE_ID"}]}
   ```

**IMPORTANT:** Do NOT end a node with an inline reference - always add text after the closing `</span>` tag.
- ✅ `"Meeting with <span data-inlineref-node=\"id\">John</span> today"`
- ❌ `"Meeting with <span data-inlineref-node=\"id\">John</span>"`

**Note:** Tana Paste syntax (`[[Node Name]]`, `[[text^id]]`) does NOT work in Input API - use the HTML span syntax above.

See `src/mcp/tools/create.ts` for implementation.

### Config Namespace
Uses `supertag` namespace (not `tana`) to avoid conflicts with official Tana app:
- Config: `~/.config/supertag/config.json`
- Data: `~/.local/share/supertag/`
- Cache: `~/.cache/supertag/`

### Export Format
Tana exports now wrap data in `storeData` object. The schema registry handles both formats.

### Export Location
Tana JSON exports are stored at: `~/Documents/Tana-Export/main/`
Files are named: `{workspaceId}@{date}.json` (e.g., `M9rkJkwuED@2025-12-12.json`)

### Entity Detection (_flags)
Based on Tana developer insights from Odin Urdland:
- **Entity flag**: `props._flags % 2 === 1` (LSB set = entity) - NOTE: uses `_flags` with underscore prefix
- **User override**: `props._entityOverride` (takes precedence if present)
- Entities are "interesting" nodes: tagged items, library items, "Create new" items
- Export contains ~13,735 entities with `_flags=1` out of 1.3M total nodes

**Entity Detection Priority** (in order):
1. `props._entityOverride` - Explicit user override (if true/false, use that)
2. `props._flags % 2 === 1` - Automatic entity flag from Tana
3. `props._ownerId.endsWith('_STASH')` - Library items (inferred)
4. Has tag in `tag_applications` table - Tagged items (inferred)

**Key Files:**
- `src/db/entity.ts` - Entity detection functions (`isEntity`, `isEntityById`, `findNearestEntityAncestor`)
- `src/types/tana-dump.ts` - Zod schema with `_flags` and `.passthrough()` to preserve props
- `tests/entity-detection.test.ts` - Comprehensive entity detection tests

### Field/Tuple Structures

Tana stores field values in tuple nodes. There are **two patterns**:

1. **Standard Field Tuples** (extracted ✅):
   - Parent → Tuple → [Label, Value1, Value2, ...]
   - May or may not have `_sourceId`
   - Up to 50 children

2. **Mega-Tuple Flat Structure** (not extracted ❌):
   - Single tuple with 50-1000+ children
   - Field labels: `"  - FieldName:"` (indentation in name)
   - Values: `"    - Value text"` (more indentation)
   - Labels and values are siblings, not parent-child

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jcfischer/supertag-cli](https://github.com/jcfischer/supertag-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
