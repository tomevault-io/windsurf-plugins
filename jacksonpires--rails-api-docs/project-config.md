---
trigger: always_on
description: Notes for future sessions working on this gem. Does not duplicate the README — focuses on design decisions, gotchas, and how to contribute without breaking things.
---

# CLAUDE.md

Notes for future sessions working on this gem. Does not duplicate the README — focuses on design decisions, gotchas, and how to contribute without breaking things.

## Architecture in one sentence

`RouteInspector` (routes) + `ControllerInspector` (Prism AST of `params.permit`) + `SchemaInspector` (AR columns) → `Config::Builder` assembles hash → `Config::Appender` merges with existing YAML → `Doc::Renderer` (ERB + `CurlRenderer`) → self-contained HTML.

## How to run

```bash
bundle install
bundle exec rake test           # 91 runs, ~0.1s
```

End-to-end smoke test against a real Rails app:
```bash
bash /tmp/rad_smoke_test.sh     # spins up app in /tmp/rad-host, mounts gem, hits /rails/api-docs
bash /tmp/rad_smoke_build.sh    # exercises rake task + OUTPUT= override + missing-config branch
```

## Conventions (non-obvious)

- **Hyphenated gem name, single `RailsApiDocs` module** (no nesting). Files live in `lib/rails-api-docs/...` (with hyphen).
- **Generator namespace forced** in `init_generator.rb` via `namespace "rails-api-docs:init"`. Without it, Rails would infer `rails_api_docs:init` from the module name.
- **`:init` and `:update` are aliases via inheritance.** `UpdateGenerator < InitGenerator` only overrides the namespace; behavior is identical. Add new flags / logic to InitGenerator only — UpdateGenerator inherits everything (including `class_option`s, which Thor preserves on subclasses).
- **Tests using `Rails::Generators::TestCase`** clash if you define a helper named `render` — use a different name (already tripped on this).
- **Hashes with string keys passed to a method with kwargs**: always wrap them in explicit `{}` — Ruby 3 sometimes interprets them as kwargs.

## Sensitive areas — touch carefully

### `Config::Appender` — append-only preserves user edits
Endpoint identity is `"#{method} #{path}"`. **Existing fields always win** over generated ones. If you change the logic, make sure the tests in `test/config/appender_test.rb` still cover:
- User edits to description/body/examples survive.
- Existing `general_configurations` are never overwritten (but new keys are added).
- Header comments (`#` lines at the top) preserved via `Loader.header`.

**Inline** comments inside the YAML are lost on re-dump — documented limitation in the README.

### `CurlRenderer` shell-escape gsub trap
This line:
```ruby
str.gsub("'") { "'\\''" }
```
**Uses the block form on purpose.** The replacement-string form (`str.gsub("'", "'\\''")`) falls into the `\'` trap — `gsub` interprets it as "post-match reference" and duplicates the content. If you touch this, run `test_escapes_single_quotes_in_body_for_shell` before committing.

### Engine `rake_tasks` / `generators` are class-level macros
They **cannot** live inside `initializer "...".do`. They must be top-level in the Engine class body. Broke this once while reorganizing.

### ERB template uses `<% ... -%>` trim mode
Whitespace control via the trailing `-`. If you add tags, keep the pattern or the HTML will sprout random blank lines.

### `render_field` is the single source of truth for field rendering
Body, params, request headers, response headers, and response schema all flow through `Renderer#render_field`. To add a new per-field attribute (a new badge or meta line), edit only that one method. The template just calls `render_fields(endpoint["body"])` etc. — never expand field markup inline in the ERB.

Order of badges in a field-row matters for visual hierarchy — see the method body for the order (format → in → readonly → writeonly → nullable → required → metas).

### `field["example"]` is the single source of truth for sample values
`CurlRenderer#body_json` and `Renderer#default_response_example` both prefer `field["example"]` over the type-derived `sample_value` fallback. The Builder + BodyInferrer seed `example:` for every inferred field, so by the time the renderer runs, there's almost always a real value to display. If you add a new place that needs a placeholder, follow the same precedence: user example wins; fall back to `RailsApiDocs::SampleValue.for(type)`.

### `RailsApiDocs::SampleValue` is the single source of truth for type→sample
Three consumers (BodyInferrer, Builder for path params, CurlRenderer, Renderer) all call `SampleValue.for(type)`. Adding a new type maps in one place.

### Pragmatic vs verbose YAML scaffold
Default emits the commonly-edited subset (`description`, `example` on every field; minimal `responses["200"]` stub). `--verbose-yaml` (or `RailsApiDocs.configuration.verbose_yaml = true`) emits **every** key with defaults — useful for discoverability, costly in lines per endpoint (~10× larger). The verbose defaults live in `verbose_field_defaults` / `verbose_endpoint_meta` methods (not constants — return fresh hashes per call so `YAML.dump` doesn't emit anchors).

## Filters & extensibility

User-facing filters on `Configuration`:
- `ignored_path_prefixes` (strings)
- `ignored_controllers` (strings **or** regexps)
- `only_controllers` (strings **or** regexps, whitelist)
- `ignored_actions` (strings)
- `api_only` (boolean) — see `JsonRouteDetector` below


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jacksonpires/rails-api-docs](https://github.com/jacksonpires/rails-api-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
