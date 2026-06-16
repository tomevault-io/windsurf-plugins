---
trigger: always_on
description: Temper reference: pack
---



# Pack: Quality Pack Manager

**Goal:** Display all defined quality packs with their enable/disable status, link targets, phase scoping, and connection health. Allow users to toggle packs, create new packs, quick-create launcher packs, and configure links and phases.

---

## Pack Resolution: Three-Tier System (v4.3.0)

Quality packs resolve from three tiers in priority order. Higher tiers shadow lower ones.

```
Priority 1 (highest) → .claude/packs/{name}/rules.md           (project-local)
Priority 2           → ~/.claude/packs/{name}/rules.md          (global / user-wide)
Priority 3 (lowest)  → $CLAUDE_PLUGIN_ROOT/.claude/packs/{name}/rules.md  (built-in)
```

**Why:** Teams ship project-specific packs in the repo. Users create global packs across all projects. Built-in packs provide defaults.

**Resolution:** When the same pack name exists in multiple tiers, only the highest-priority version is loaded. Project-local always wins over global, which always wins over built-in.

### Pack Discovery Algorithm

Scan all three tiers, deduplicate by name (highest priority wins):

```
Step 1: Scan built-in packs
  For each directory in $CLAUDE_PLUGIN_ROOT/.claude/packs/ (excluding stacks/):
    - If {name}/rules.md exists → add to manifest with scope: "built-in"

Step 2: Scan global packs
  For each directory in ~/.claude/packs/ (excluding stacks/):
    - If {name}/rules.md exists → add to manifest with scope: "global"
    - If name already in manifest → replace (global shadows built-in)

Step 3: Scan project-local packs
  For each directory in .claude/packs/ (excluding stacks/):
    - If {name}/rules.md exists → add to manifest with scope: "project"
    - If name already in manifest → replace (project shadows all)
```

---

## Cached Pack Manifest (v4.4.0)

Pack discovery results are cached to `.temper/pack-manifest.json` for instant subsequent loads.

### Cache Behavior

- **First run:** Full filesystem scan of all three tiers → write manifest
- **Subsequent runs:** Load from cache (near-instant)
- **Cache is rebuilt when:**
  1. `temper.config` file modified (project or global) — check mtime
  2. Pack directories added or removed in any tier — check directory listing
  3. Manifest `version` field doesn't match expected schema version
  4. Any pack's `rules.md` file modified — check mtime of each rules_path

### Manifest Structure

```json
{
  "version": 1,
  "last_built": "2026-04-20T10:00:00Z",
  "config_mtime": "2026-04-20T09:30:00Z",
  "packs": [
    {
      "name": "quality",
      "enabled": true,
      "scope": "built-in",
      "phases": "all",
      "link": null,
      "connected": null,
      "rules_path": "$CLAUDE_PLUGIN_ROOT/.claude/packs/quality/rules.md",
      "rule_summary": "Code quality: method length, DRY, naming"
    },
    {
      "name": "tdd",
      "enabled": true,
      "scope": "built-in",
      "phases": ["build"],
      "link": null,
      "connected": null,
      "rules_path": "$CLAUDE_PLUGIN_ROOT/.claude/packs/tdd/rules.md",
      "rule_summary": "TDD: RED-GREEN-REFACTOR enforcement"
    }
  ]
}
```

### Manifest Operations

**Read manifest:**
1. Check `.temper/pack-manifest.json` exists
2. If not: run full discovery, write manifest, return
3. If exists: load, check staleness conditions
4. If stale: run full discovery, overwrite manifest, return
5. If fresh: return cached manifest

**Invalidate manifest:**
- After toggling packs (enabled status changed)
- After creating a new pack
- After modifying pack link or phases config

---

## Pack Configuration Schema (v4.3.0)

### Extended Config Format

The `packs:` field in `.claude/temper.config` now supports objects with `name`, `link`, and `phases`:

```yaml
packs:
  - name: quality                        # Simple format (no link, all phases)
  - name: tdd
    phases: [build]                      # Only during build phase
  - name: security
    phases: [review, check]              # Only during review and check
  - name: api-standards
    link: plugin://my-api-linter         # Links to an installed plugin
  - name: sec-review
    link: skill://security-review        # Links to a skill
  - name: git                            # Simple format
```

### Backward Compatibility

Simple string format still works:
```yaml
packs:
  - quality
  - tdd
```
This is equivalent to:
```yaml
packs:
  - name: quality
  - name: tdd
```

### Parsing Note

When reading `packs:`, each entry can be either a **string scalar** (simple format) or a **mapping** with `name` key (extended format). Parser must check type:
- If string → treat as `{name: <value>, phases: "all", link: null}`
- If mapping → read `name` (required), `phases` (default: "all"), `link` (default: null)

---

## Phase Scoping (v4.3.0)

Packs can be restricted to specific Temper phases. Only packs scoped to the current phase are loaded.

### Available Phases

`plan`, `design`, `build`, `review`, `check`, `fix`

### Default Behavior

If `phases` is omitted or set to `all`, the pack activates during every phase.

### Phase Filtering

When a stage starts:
1. Read current phase from the command being executed (e.g., `/temper:build` → phase = "build")
2. Filter manifest packs: only include packs where `phases` is "all" or contains the current phase
3. Load filtered packs' rules + any linked context

---


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [galando/temper](https://github.com/galando/temper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-16 -->
