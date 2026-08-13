---
trigger: always_on
description: Whenever making any database-related change, a new SQL script file **must be created first** in the repo root before (or alongside) the code change. This is non-negotiable.
---

# Blog-Engine — Claude Project Rules

## Database Script Rule (MANDATORY)

Whenever making any database-related change, a new SQL script file **must be created first** in the repo root before (or alongside) the code change. This is non-negotiable.

### What counts as a database-related change

- Adding, removing, or renaming a column in any table
- Creating or dropping a table
- Adding or changing an index, constraint, or default value
- Data migrations (UPDATE/INSERT/DELETE on existing rows)
- Changes to `UserSettings` / `Setting.cs` that add a new JSON-stored field (even though no schema migration is needed, a comment/no-op script documenting the intent is still required)
- Changes to `MigrationService.cs` that alter the schema
- Any Dapper query change that assumes a new column exists

### File naming convention

```
DBScripts/YYYY-MM-DD_short-description.sql
```

Use today's actual date (available in context as `currentDate`). Examples:

```
DBScripts/2026-05-17_add-google-analytics-id-to-settings.sql
DBScripts/2026-05-17_add-newsletter-preferences-column.sql
DBScripts/2026-05-17_create-audit-log-table.sql
```

### Script requirements

- Place in the **`DBScripts/` folder** at the repo root
- Must be **idempotent** — safe to run more than once (`IF NOT EXISTS`, `IF COL_LENGTH(...)`, `MERGE`, etc.)
- Include a header comment explaining what the change is and why
- For JSON-blob-only changes (no schema change needed), the script must document this explicitly:

```sql
-- DBScripts/YYYY-MM-DD_description.sql
-- Change: Added GoogleAnalyticsId to UserSettings JSON blob.
-- Schema impact: NONE — stored in Settings.JsonPayload, no column change.
-- Action required: None. Field is populated automatically on next settings save.
PRINT 'No schema migration required for this change.';
```

### Enforcement

Do **not** modify `Blog.Core/Domain/Setting.cs`, `Blog.Infrastructure/Data/MigrationService.cs`,
any `*Repository.cs`, or any `.sql` file without also writing the dated script.
If a task requires multiple DB changes, one script per logical change is preferred,
or combine them with clear section headers.

---

## sqlcmd Encoding Rule (MANDATORY)

All `sqlcmd` commands that execute SQL files **must** include `-f 65001` (UTF-8 code page).

```powershell
sqlcmd -S <server> -d <db> -U <user> -P <pass> -f 65001 -i "DBScripts\YYYY-MM-DD_script.sql"
```

### Why this is required

Without `-f 65001`, `sqlcmd` reads `.sql` files as Windows-1252 (CP1252). Any UTF-8 multi-byte
character in the file is misread as multiple CP1252 glyphs and stored verbatim in the database
(mojibake). For example, an em dash `—` (UTF-8: E2 80 94) becomes the three-character garbage
sequence `â€"` in NVARCHAR columns.

### What gets corrupted

Any SQL file that contains — directly or in N'' string literals — any of these characters:

| Character | UTF-8 bytes | Stored as (mojibake) |
|-----------|-------------|----------------------|
| — em dash | E2 80 94 | â€" |
| – en dash | E2 80 93 | â€" |
| ' right single quote | E2 80 99 | â€™ |
| ' left single quote | E2 80 98 | â€˜ |
| " left double quote | E2 80 9C | â€œ |
| " right double quote | E2 80 9D | â€ |
| … ellipsis | E2 80 A6 | â€¦ |
| • bullet | E2 80 A2 | â€¢ |
| (NBSP) | C2 A0 | Â  |

### Fix if corruption already occurred

Use `DBScripts/2026-05-23_fix-mojibake-encoding-all-posts.sql` as a template — it uses
`NCHAR()` integer codes (encoding-neutral) to identify and REPLACE all known mojibake
sequences. Run it with `-f 65001` too, even though NCHAR() codes are safe either way.

---

## Theme / Design / Layout Backward-Compatibility Rule (MANDATORY)

Any new theme, design system, or layout addition **must not break existing users** who have not
explicitly selected the new feature. This applies to all of:

- New layout names added to `ApplyLayoutPreset` / `validLayouts`
- New Inkwell presets added to `ApplyInkwellPreset` / preset selectors
- New `CustomThemeSetting` keys seeded by `SeedDefaultsAsync`
- New view components, partial views, or shared layout files
- Changes to `_PublicLayout.cshtml`, `_AdminLayout.cshtml`, or any Shared view
- Changes to `FooterViewComponent`, `NavbarViewComponent`, or any other `ViewComponent`
- New CSS variables, Tailwind config changes, or `inkwell.css` modifications

### Mandatory checks before shipping any theme/layout change

**1. View component resilience** — Every `ViewComponent` that selects a view by name from a
DB setting **must** have an explicit supported-values guard. Unknown or legacy values must fall
back to a safe default (e.g. `"Neutral"`), never throw `InvalidOperationException`.

```csharp
// Good — explicit guard
private static readonly HashSet<string> _supported =
    new(StringComparer.OrdinalIgnoreCase) { "Neutral", "Magazine", "Grid", "Minimal", "Classic", "Modern" };

var raw = layoutSetting?.EffectiveValue ?? "Neutral";
var layout = _supported.Contains(raw) ? raw : "Neutral";
return View(layout, model);

// Bad — crashes for any DB value not in the Views folder
return View(layoutSetting?.EffectiveValue ?? "Neutral", model);
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marutisoftwaresolutions/inkwell](https://github.com/marutisoftwaresolutions/inkwell) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
