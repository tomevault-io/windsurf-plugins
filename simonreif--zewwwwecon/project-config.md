---
trigger: always_on
description: R package + Quarto extension providing opinionated templates for reproducible economics research. The repo is simultaneously:
---

# AGENTS.md

## What this repo is

R package + Quarto extension providing opinionated templates for reproducible economics research. The repo is simultaneously:
- An R package (`DESCRIPTION`, `R/new_project.R`) — single exported function `new_project()`
- A Quarto extension (`_extensions/zewwww-econ/`) with two custom formats: `zewwww-econ-pdf` and `zewwww-econ-beamer`
- A project scaffold whose canonical template lives in `inst/FullProject/`

---

## Audience 1: Using the template (research projects)

### Starting a new project

```r
zewwwwEcon::new_project("path/to/new_project")
# or: RStudio New Project wizard → "zewwww-econ" entry
```

This copies `inst/FullProject/` verbatim into the target path.

### Data pipeline convention

Projects follow a strict directory layout (`.gitignore` enforces that A/C/D are not committed):
```
A_Orig/   raw source data         (git-ignored, .gitkeep only)
B_Prog/   R analysis scripts
C_Temp/   intermediate .rds       (git-ignored)
D_Out/    final output .rds       (git-ignored, g_*.rds = graphs, t_*.rds = tables)
Paper/    .qmd sourcing D_Out/
Slides/   .qmd sourcing D_Out/
```

Every `B_Prog/` script sources `B_Prog/0setup.R` first. Do not load additional packages beyond what `0setup.R` provides.

### Rendering documents

```bash
make                              # renders Paper/ and Slides/ PDFs
make Paper/Paper_TITLE.pdf        # render paper only
make Slides/Slides_TITLE.pdf      # render slides only
quarto render Paper/Paper_TITLE.qmd
quarto preview Paper/Paper_TITLE.qmd  # interactive preview
```

### Non-obvious YAML header variables

Paper: `number`, `repohash` (git SHA inline R expr), `prelim-note` (watermark), `jel`, `color-links` (RGB tuple e.g. `"82,125,164"`), `add-tex` (raw LaTeX into preamble).

Slides: `presenter`, `partner-logo`, `totalnumber`, `event`, `english`.

### Callout Blocks

Callout Blocks work in slides (`zewwww-econ-beamer`) but are **not supported** in the paper format (`zewwww-econ-pdf`).

### VSCode snippets

In `.qmd` files: `fig`+Tab → fenced figure block with `readRDS(here("D_Out/g_....rds"))` and `.notes-block`; `tbl`+Tab → equivalent for tables. Defined in `.vscode/fig.code-snippets` and `.vscode/tbl.code-snippets`.

### Writing analysis scripts (B_Prog/)

Every script follows this exact structure:

```r
# This file creates a [description]
# - Input:  [source file(s), or "None" for synthetic data]
# - Output: D_Out/g_name.rds

library(here)
source(here("B_Prog/0setup.R"))

# Get data ----
...

# Make plot ----
g_name <- ggplot(...) + ...
saveRDS(g_name, here("D_Out/g_name.rds"))
rm(..., g_name) # Clean workspace
```

Rules:
- Never load additional packages — everything comes from `0setup.R`
- Graph objects: `g_` prefix; table objects: `t_` prefix — variable name must match the `.rds` filename exactly (`g_hist` → `D_Out/g_hist.rds`)
- All file paths via `here()`; never relative (`"../"`) or absolute paths
- `set.seed()` before any random calls
- End every script with `rm()` covering all created objects, commented `# Clean workspace`
- Use `C_Temp/` for intermediate data shared between scripts; `D_Out/` for final paper/slides outputs only

### Updating the Makefile

For each new output, add to the `GRAPHS` or `TABLES` variable and add a target. Recipe lines require a literal tab character, not spaces.

```makefile
# Add filename to the appropriate variable:
GRAPHS = \
  D_Out/g_existing.rds \
  D_Out/g_name.rds

# Add a target:
D_Out/g_name.rds: $(R_SETUP) B_Prog/script.R
	Rscript B_Prog/script.R

# If the script reads from C_Temp/:
D_Out/g_name.rds: $(R_SETUP) B_Prog/script.R C_Temp/data.rds
	Rscript B_Prog/script.R

# Data prep writing to C_Temp/:
C_Temp/data.rds: $(R_SETUP) $(DATA) B_Prog/1dataprep.R
	Rscript B_Prog/1dataprep.R
```

`R_SETUP = B_Prog/0setup.R` is a dependency of every script target — if `0setup.R` changes, all downstream scripts rerun automatically.

### Adding figures to .qmd files

````markdown
:::: {#fig-name fig-pos="h" fig-cap="Caption text."}
```{r name, cache=TRUE, cache.extra=tools::md5sum(here("D_Out/g_name.rds"))}
#| echo: false
#| warning: false
#| message: false
#| fig-width: 3.5
#| fig-asp: 0.5
#| fig-align: center
#| fig-pos: H
g_name <- readRDS(here("D_Out/g_name.rds"))
g_name
rm(g_name)
```

::: {.notes-block}
**Notes:** Notes text.
:::
::::
````

`cache=TRUE` is recommended but for figures but needs `cache.extra=tools::md5sum(here("D_Out/g_name.rds"))` method — without it Quarto's cache will not detect when the underlying `.rds` changes. Cross-reference in text with `@fig-name`.

### Adding tables to .qmd files

````markdown
:::: {#tbl-name tbl-pos="h" tbl-cap="Caption text."}
```{r name}
#| echo: false
#| warning: false
#| message: false
#| tbl-align: center
#| tbl-pos: T
t_name <- readRDS(here("D_Out/t_name.rds"))
t_name
rm(t_name)
```

::: {.notes-block}
**Notes:** Notes text.
:::
::::
````

No `cache=TRUE` needed for tables. Cross-reference in text with `@tbl-name`.

### Customising notes blocks and appendix counters

**`.notes-block` attributes** (`notes-block.lua` defaults in parentheses):

```markdown
::: {.notes-block width="1.0" above="-5" below="15"}
**Notes:** Text.
:::
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [simonreif/zewwwwEcon](https://github.com/simonreif/zewwwwEcon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-05 -->
