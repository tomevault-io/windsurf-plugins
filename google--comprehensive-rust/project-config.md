---
trigger: always_on
description: This repository contains the source code for Comprehensive Rust, a family of
---

# Project Overview

This repository contains the source code for Comprehensive Rust, a family of
courses on Rust developed by Google, starting with Rust foundations, and
including deep dives into specialized topics like Android, Chromium, bare-metal
development, and concurrency. The project is a Rust workspace that leverages
`mdbook` to generate a course website.

## Key Technologies

- **Rust:** The primary programming language for the course subject, custom
  tools, and examples.
- **mdbook:** A command-line tool to create books from Markdown files, used for
  generating the course website.
- **Custom mdbook Preprocessors:** `mdbook-course` and `mdbook-exerciser` are
  Rust binaries that extend `mdbook`'s functionality, for example, to extract
  exercise starter code.
- **`cargo xtask`:** A custom binary within the workspace used for project
  automation, simplifying common development tasks.

# Building and Running

The project uses `cargo xtask` for project-specific automation, like builds,
tests, and managing translations.

## Setup

1. **Install Rust:** Follow the instructions on
   [https://rustup.rs/](https://rustup.rs/).
2. **Clone Repository:**
   ```bash
   git clone https://github.com/google/comprehensive-rust/
   cd comprehensive-rust
   ```
3. **Install Project Tools:**
   ```bash
   cargo xtask install-tools
   ```
   This is a necessary first step for working with this repository. It will
   install the correct versions of all tools used by the project.

## Commands

All commands are run using `cargo xtask`. Run `cargo xtask --help` for a full
list of options.

- **Serve the Course Locally:** Starts a web server to view the course content.
  ```bash
  cargo xtask serve [--language <ISO_639_language_code>] [--output <output_directory>]
  ```
  (e.g., `cargo xtask serve -l da` for the Danish translation)

- **Build the Course:** Creates a static version of the course in the `book/`
  directory.
  ```bash
  cargo xtask build [--language <ISO_639_language_code>] [--output <output_directory>]
  ```

- **Run Rust Snippet Tests:** Tests all Rust code snippets included in the
  course material.
  ```bash
  cargo xtask rust-tests
  ```

- **Run Web Driver Tests:** Executes web driver tests located in the `tests/`
  directory.
  ```bash
  cargo xtask web-tests [--dir <book_html_directory>]
  ```

# Development Conventions

- **Project Automation:** `cargo xtask` is the primary interface for common
  development tasks.
- **Course Content:** Markdown files in the `src/` directory, structured
  according to `src/SUMMARY.md`.
- **Code Formatting:** `dprint fmt` is used to format all source files according
  to `rustfmt.toml` and `dprint.json`. Note that you must first install the
  project tools with `cargo xtask install-tools`.
- **Contributions:** Refer to `CONTRIBUTING.md` for guidelines on contributing
  to the project.
- **Style:** Refer to `STYLE.md` for style guidelines. When making changes to
  Markdown files in `src/`, always first read `STYLE.md` and follow its
  conventions.
- **GitHub Actions:** The project uses composite GitHub Actions to simplify CI
  workflows. These actions should be preferred over hand-written commands.
  - **`apt-get-install`:** This action efficiently installs Debian packages. It
    configures `dpkg` and `apt` to skip documentation and translations, and
    ensures that `apt-get update` is run only once per job. This significantly
    speeds up CI runs.
  - **`install-mdbook`:** A composite action to install `mdbook` and its
    dependencies, including `pandoc` and `texlive`.
  - **`setup-rust-cache`:** A composite action that configures the
    `Swatinem/rust-cache` action.

## Developing Exercises

Exercises allow students to practice what they have learned. When adding or
updating exercises, follow these structural conventions:

- **File Structure:**
  - `exercise.md`: Contains the problem description and a code block with
    placeholders.
  - `exercise.rs`: Contains the full solution code, including a license header
    and `ANCHOR` tags to delimit sections.
  - `solution.md`: Includes the full solution code from `exercise.rs`.
  - `Cargo.toml`: Must define a `[[bin]]` target pointing to `exercise.rs` so
    that the solution code is compiled and tested.

- **Content Inclusion:**
  - Use `{{#include exercise.rs:anchor_name}}` in `exercise.md` to show specific
    parts of the code (e.g., setup, main).
  - Use `{{#include exercise.rs:solution}}` in `solution.md` to show the
    solution code _without_ the license header. Ensure `exercise.rs` has a
    `// ANCHOR: solution` line before the first line of the solution. It is
    unnecessary to add a `// ANCHOR_END: solution` line at the bottom of the
    file.

- **Testing:**
  - Run `cargo xtask rust-tests` to ensure the solution code compiles and runs
    correctly.
  - Run `cargo check -p <crate_name>` to verify the specific exercise crate.

## Markdown Conventions

- **Headings:**
  - **H1 (`#`):** Used for the main title of each page. Each slide has exactly
    one title.
  - **H2 (`##`):** Used for major sections. Slides do not use H2 headings to
    save vertical space; more slides are created instead.
  - **H3 (`###`):** Used for sub-sections, but not on slides.

- **Emphasis:**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [google/comprehensive-rust](https://github.com/google/comprehensive-rust) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
