---
trigger: always_on
description: This is the SvelteKit documentation site for the Stacks v3 design system. Pages are written in mdsvex (`.md` files with Svelte support).
---

# stacks-docs — Claude Instructions

This is the SvelteKit documentation site for the Stacks v3 design system. Pages are written in mdsvex (`.md` files with Svelte support).

## Page anatomy

### Frontmatter

```yaml
---
title: "Page Title"
description: "…" # shown in the page header and meta
figma: "https://…" # optional — links to Figma file
svelte: "https://…" # optional — links to stacks-svelte component
js: true # only if the page has a JavaScript section
updated: 2026-01-15 # ISO date of last meaningful content update
---
```

### Script block

```svelte
<script lang="ts">
    import { ComponentName } from '@stackoverflow/stacks-svelte';
    import { IconName } from '@stackoverflow/stacks-icons/icons';
    import ClassTable from '$components/ClassTable.svelte';
    import Example from '$components/Example.svelte';
    import type { ClassTableRow } from '$components/ClassTable.svelte';

    const classes: ClassTableRow[] = [ … ];
</script>
```

### Content

Use standard markdown for prose. Svelte components can be used anywhere in the body. Fenced code blocks use explicit language tags (` ```html `, ` ```js `, etc.).

## Adding a new page

1. Create `src/docs/public/<section>/<name>.md`
2. Add an entry to `src/structure.yaml` under the correct section in **alphabetical order** — no `legacy:` key needed for new pages:

```yaml
- title: "Page Title"
  slug: "page-name"
```

## Available doc components

| Component          | Location                              | Purpose                                                        |
| ------------------ | ------------------------------------- | -------------------------------------------------------------- |
| `ClassTable`       | `$components/ClassTable.svelte`       | CSS class / JS attribute reference tables                      |
| `Example`          | `$components/Example.svelte`          | Bordered container for static example content                  |
| `Grid`             | `$components/Grid.svelte`             | Equal-width responsive column grid for Do/Don't blocks         |
| `GridColumn`       | `$components/GridColumn.svelte`       | Column within a `Grid`; accepts `extraClasses`, `padding` prop |
| `PreviewTable`     | `$components/PreviewTable.svelte`     | Table of component previews: Example \| Class \| Description   |
| `BannerPreview`    | `$components/BannerPreview.svelte`    | Static pre-built banner instance for illustration              |
| `BannerDemo`       | `$components/BannerDemo.svelte`       | Interactive banner playground with user controls               |
| `StacksEditorDemo` | `$components/StacksEditorDemo.svelte` | Interactive Stacks editor playground                           |
| `TableSortDemo`    | `$components/TableSortDemo.svelte`    | Interactive sortable table playground                          |

**Naming convention:** `Example` = generic wrapper; `*Preview` = static pre-built component instance; `*Demo` = interactive playground with user controls.

### ClassTable

Use `ClassTable` for every reference table — never raw markdown tables.

```svelte
<ClassTable {classes} expandable />

<!-- JS section tables — rename columns as appropriate -->
<ClassTable classes={attributes} headings={{ class: 'Attribute', parent: 'Applies to' }} />
<ClassTable classes={events}     headings={{ class: 'Event',     parent: 'Applies to' }} />
```

**ClassTableRow fields:** `class`, `parent`, `modifies`, `output`, `description`, `define`, `responsive`.

- Only populate relevant fields — `ClassTable` auto-hides columns with no data.
- Use `'N/A'` (string) for explicitly empty `parent`/`modifies` — renders as muted text.
- The `headings` prop renames any column: `headings={{ class: 'Selector', description: 'Notes' }}`.

### Example

All component examples must be wrapped in `<Example>`. Multiple labelled variants in one block:

```svelte
<Example>
    <div class="d-flex g16 ai-start fw-wrap">
        <div class="d-flex fd-column g8">
            <span class="ff-mono mb8">Label</span>
            <ComponentName />
        </div>
    </div>
</Example>
```

Sub-example labels always use `class="ff-mono mb8"`.

### Grid / GridColumn

`Grid` renders an equal-width responsive CSS grid (stacks to a single column below 780px). Use it for Do/Don't comparisons and side-by-side content blocks.

```svelte
<Grid>
    <GridColumn extraClasses="bg-red-100">
        <!-- Don't content -->
    </GridColumn>
    <GridColumn extraClasses="bg-green-100">
        <!-- Do content -->
    </GridColumn>
</Grid>
```

`GridColumn` props:

- `extraClasses` — appended to the column div (default: `bg-black-200`)
- `padding` — adds `p24` when `true` (default: `true`); set `false` if you're applying `py*` in `extraClasses` to avoid conflict
- `...rest` — any extra attributes (e.g. `style="background: var(--brand-color-orange)"`) are spread onto the outer div

For the `column-strike` diagonal overlay on "Instead of this" blocks, add it to `extraClasses`:

```svelte
<GridColumn extraClasses="bg-black-200 column-strike">…</GridColumn>
```

### Svelte components vs raw HTML

- **Always prefer** Svelte components from `@stackoverflow/stacks-svelte` when one exists.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [StackExchange/Stacks](https://github.com/StackExchange/Stacks) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
