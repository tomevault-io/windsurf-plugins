---
trigger: always_on
description: Cursor rules for R development with best practices integration.
---

You are an R programming assistant, make sure to use the best practices when programming in R:

## Project Structure and File Organization
- Organize projects into clear directories: 'R/' (scripts), 'data/' (raw and processed), 'output/' (results, plots), 'docs/' (reports). For R packages, use 'inst/' for external files; for non-packages, consider 'assets/'.
- Use an 'Rproj' file for each project to manage working directories and settings.
- Create reusable functions and keep them in separate script files under the 'R/' folder.
- Use RMarkdown or Quarto for reproducible reports combining code and results. Prefer Quarto if available and installed.
- Keep raw data immutable; only work with processed data in 'data/processed/'.
- Use 'renv' for dependency management and reproducibility. All the dependencies must be installed, synchronized, and locked.
- Version control all projects with Git and use clear commit messages.
- Give a snake_case consistent naming for the file names. The file names should not be too long.
- Avoid using unnecessary dependencies. If a task can be achieved relatively easily using base R, use base R and import other packages only when necessary (e.g., measurably faster, more robust, or fewer lines of code).

## Package Structure
- If the R project is an R package, make sure to mention the dependencies used inside the package within the 'DESCRIPTION' file. All dependencies must have their version number mentioned (e.g: R6 (>= 2.6.1))
- If the R project is an R package, make sure a 'LICENSE' file is available. 
- If the R project is an R package, make sure a 'NEWS.md' file is available which should track the package's development changes.
- If the R project is an R package, make sure that each external file used inside the package is saved within the 'inst' folder. Reading the file should be done using the 'system.file' function. 
- If the R project is an R package, Always use 'devtools::load_all' before testing the new functions. 
- If the R project is an R package, run 'devtools::check()' to ensure the package has no issues. Notes are okay; avoid warnings and errors.
- If the R project is an R package, document functions using roxygen2. Use 'devtools::document()' to generate the required documentation (.Rd files) and 'NAMESPACE' file.

## Naming Conventions
- snake_case: variables and functions (e.g., \`total_sales\`, \`clean_data()\`). 
- UpperCamelCase: for R6, S3, S4, S7 class names (e.g., \`LinearModel\`).
- SCREAMING_SNAKE_CASE: constants and global options (e.g., \`MAX_ITERATIONS\`).
- Avoid ambiguous names (e.g., use \`customer_id\` instead of \`id\`).
- Use verbs for function names (e.g., \`plot_data\`, \`calculate_mean\`).
- Avoid function or variable names that has already been assigned by R, for example avoid 'sd', it's already a function in R. Another example would be 'data'.
- When working with R6 classes, always prepend a '.' to private methods and fields. An example of a method would be '.get_data()' which will be used as 'private$.get_data()'. 
    
## Coding Style
- Follow the [tidyverse style guide](https://style.tidyverse.org/).
- Use spaces around operators (\`a + b\`, not \`a+b\`).
- Keep line length <= 80 characters for readability.
- Use consistent indentation (2 spaces preferred).
- Use '#' for inline comments and section headers. Comment only when necessary (e.g., complex code needing explanation). The code should be self‑explanatory.
- Write modular, reusable functions instead of long scripts.
- Prefer vectorized operations over loops for performance.
- Always handle missing values explicitly (\`na.rm = TRUE\`, \`is.na()\`).
- When creating an empty object to be filled later, preallocate type and length when possible (e.g., 'x <- character(length = 100)' instead of 'x <- c()').
- Always use <- for variables' assignment, except when working with 'R6' classes. The methods inside the 'R6' classes are assigned using '='
- When referencing a function from a package always use the '::' syntax, for example 'dplyr::select'
- Always use 'glue::glue' for string interpolation instead of 'paste0' or 'paste'
    
## Performance and Optimization
- Profile code with \`profvis\` to identify bottlenecks.
- Prefer vectorized functions and the apply family ('apply', 'lapply', 'sapply', 'vapply', 'mapply', 'tapply') or 'purrr' over explicit loops. When using loops, preallocate type and memory beforehand.
- Use data.table for large datasets when performance is critical and data can fit in memory.
- When reading a CSV, prefer 'data.table::fread' or 'readr::read_csv' depending on the codebase. If the codebase is tidyverse‑oriented, prefer 'readr'; otherwise use 'data.table'.

- Use duckdb when data is out of memory.
- Avoid copying large objects unnecessarily; use references when possible.
    
## Testing and Validation
- Write unit tests with \`testthat\`.
- Use reproducible random seeds (\`set.seed()\`) for consistent results.
- Test functions with edge cases (empty inputs, missing values, outliers).
- Use R CMD check or \`devtools::check()\` for package development.
    
## Reproducibility
- Use RMarkdown or Quarto for reproducible reports combining code and results. Prefer 'Quarto' if already available and installed.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XD3an/awesome-ai-coding-all-in-one](https://github.com/XD3an/awesome-ai-coding-all-in-one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
