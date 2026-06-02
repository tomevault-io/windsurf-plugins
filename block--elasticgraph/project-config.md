---
trigger: always_on
description: This file provides guidance to AI coding agents when working with code in this repository.
---

# AGENTS.md

This file provides guidance to AI coding agents when working with code in this repository.
It is tool-neutral and intended to be shared across agent runtimes.

## Memory Bank

- Use `ai-memory/README.md` for durable project context and architecture notes.
- If present, use `ai-memory/PROMPT.md` and `ai-memory/TASKS.md` for active feature/task context.
- Keep memory-bank updates concise and aligned with implemented behavior.

## Project Overview

ElasticGraph is a schema-driven, scalable, cloud-native, batteries-included GraphQL platform backed by Elasticsearch/OpenSearch. It's designed as a modular system with a small core and numerous built-in extensions, organized as a Ruby monorepo containing 20+ gems.

## Common Commands

### Testing
- `script/run_specs` - Run entire test suite (uses flatware for parallelization)
- `script/run_gem_specs [gem_name]` - Run tests for a specific gem (e.g., `elasticgraph-support`)
- `bundle exec rspec [path]` - Run specific test file or directory
- `bundle exec rspec --only-failures` - Run only previously failed tests
- `bundle exec rspec --next-failure` - Run failures one at a time (for iterative debugging)
- `script/flatware_rspec [path]` - Run tests in parallel (faster for large test runs, slower for small subsets)

**Important**: Integration/acceptance tests require a running datastore:
```bash
bundle exec rake elasticsearch:test:boot
# or
bundle exec rake opensearch:test:boot
```

### Build & Validation
- `script/quick_build` - Run abridged CI build (recommended before opening PRs)
- `script/lint` - Run linter (Standard Ruby)
- `script/lint --fix` - Auto-fix linting issues
- `script/type_check` - Run Steep type checker
- `script/spellcheck` - Check spelling (uses codespell)
- `script/spellcheck -w` - Auto-fix spelling issues

### Schema & Artifacts
- `bundle exec rake schema_artifacts:dump` - Regenerate schema artifacts after schema definition changes
- Schema definition files: `config/schema.rb` and `config/schema/*.rb`

### Local Development
- `bundle exec rake boot_locally` - Boot ElasticGraph locally (from a new project)
- `bundle exec rake site:serve` - Serve project website locally at http://localhost:4000/elasticgraph/
- `bundle exec rake site:preview_docs:[gem_name]` - Preview API docs for a specific gem (faster feedback loop)

### Documentation
- API documentation uses YARD
- 100% documentation coverage is required for all public methods and classes.
- Website source: `config/site/`
- Example queries: `config/site/examples/*/queries/`
- When writing links in documentation, use permalinks (links to a specific commit/version)
- Prefer relative links to ElasticGraph documentation over external links to ruby-doc.org

#### Validated Code Snippets in User Guides

One of the website's guiding principles is that **all code snippets in guides must be validated** — pulled from real example projects under `config/site/examples/` rather than hand-written inline in the markdown. This ensures CI catches any future API breakage that would invalidate the docs.

When you add or modify code examples in `config/site/src/guides/*.md`, do this from the start (don't write inline `code='...'` blocks first and migrate later):

1. **Pick or create an example project** under `config/site/examples/<name>/`. Each project needs at minimum:
   - `schema.rb` — schema definition file (validated by `schema_artifacts:dump`)
   - `local_settings.yaml` — points `schema_artifacts.directory` at `config/site/examples/<name>/schema_artifacts`
   - Optional: `queries/<category>/*.graphql` (validated against the schema by the query registry)
   - Optional: additional `.rb` files for snippets that don't need to execute (e.g. `Rakefile`-style examples)
2. **Mark snippet ranges** in the source files with `# :snippet-start: <name>` / `# :snippet-end:` comment fences. The fenced region becomes available as `<example>.snippets.<file>.<name>`. Whole files are also exposed as `<example>.files.<file>`, but **prefer fenced snippets** — they let you keep the standard Block copyright header at the top of the file (and any boilerplate like `schema.json_schema_version 1`) without it leaking into the rendered docs. Trim each snippet to just what's pertinent to the guide.
3. **Reference snippets from markdown** with the data parameter:
   ```
   {% include copyable_code_snippet.html language="ruby" data="<example>.snippets.<file>.<name>" %}
   ```
   Never write inline `code='...'` blocks for guide content — those bypass validation.
4. **Run** `bundle exec rake site:examples:<name>:extract_snippets` to regenerate `config/site/src/_data/<name>.yaml` (this file is gitignored — it gets rebuilt on the fly).
5. **Preview** with `bundle exec rake site:serve` to confirm the snippet renders correctly before opening a PR.

For an end-to-end example, see `config/site/examples/custom_resolver/` and how its snippets are pulled into `config/site/src/guides/custom-graphql-resolvers.md`.

## Architecture

### Monorepo Structure
All gems follow the pattern: `elasticgraph-[name]/` containing:
- `lib/elastic_graph/[name]/` - Source code
- `spec/` - RSpec test suite
- `Gemfile` - Symlinked from root `Gemfile`
- `[name].gemspec` - Gem specification

### Gem Categories


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [block/elasticgraph](https://github.com/block/elasticgraph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
