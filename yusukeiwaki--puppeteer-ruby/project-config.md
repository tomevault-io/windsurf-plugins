---
trigger: always_on
description: - Read `CLAUDE.md` and `CLAUDE/` first; they define the CDP architecture, porting workflow, testing strategy, and concurrency plan.
---

# Repository Guidelines

## Start Here (Project-Specific Guidance)

- Read `CLAUDE.md` and `CLAUDE/` first; they define the CDP architecture, porting workflow, testing strategy, and concurrency plan.
- This repository is a CDP-based Ruby port of Puppeteer, **focused on Chrome/Chromium only**.
- CI covers Ruby 3.2, 3.3, 3.4 with latest Chrome.

### Technical Details

- **Ruby version**: Minimum is 3.2+
- **Concurrency**: Uses `socketry/async` (version 2.35.1+) for Fiber-based concurrency. See `CLAUDE/concurrency.md` for details.

## Project Structure & Module Organization

- `lib/puppeteer/`: core implementation (entry points: `puppeteer.rb`, `browser.rb`, `page.rb`, `frame.rb`, `element_handle.rb`, `connection.rb`, `cdp_session.rb`).
- `spec/integration/`: browser-driven specs; fixtures in `spec/assets/`.
- `spec/puppeteer/`: unit tests that do not require a browser.
- `docs/api_coverage.md`: API implementation status.

## Build, Test, and Development Commands

- Run all tests: `bundle exec rspec`
- Run a single file: `bundle exec rspec spec/integration/page_spec.rb`
- Debug (non-headless): `DEBUG=1 bundle exec rspec spec/integration/page_spec.rb`
- Lint: `bundle exec rubocop` (auto-fix: `bundle exec rubocop -a`)

### Useful Environment Variables

- `PUPPETEER_EXECUTABLE_PATH_RSPEC`: custom Chrome path
- `PUPPETEER_CHANNEL_RSPEC`: Chrome channel (for example `chrome-dev`)
- `PUPPETEER_NO_SANDBOX_RSPEC`: add `--no-sandbox` flag

## Coding Style & Naming Conventions

- Follow `.rubocop.yml`; prefer explicit keyword arguments for public APIs.
- Public APIs mirror Puppeteer naming but use Ruby `snake_case`.
- Custom errors inherit from `Puppeteer::Error`.
- Use `Puppeteer::AsyncUtils` for async operations; see `CLAUDE/concurrency.md` for patterns.

## Type Annotations (rbs-inline)

- Add `# rbs_inline: enabled` at the top of the file (after `# frozen_string_literal: true` if present).
- Use doc-style `# @rbs` annotations for parameters and return types.
- Always include descriptions with `--` (example: `# @rbs url: String -- Target URL`).
- Use `A?` for nullable types and `A | B | nil` for unions that include nil.
- Avoid `@rbs!` blocks (RubyMine doesn't recognize them).
- Avoid `**options` in public APIs; RubyMine shows it as `untyped`. Prefer explicit keyword args.

**Generate signatures:**
- `bundle exec rake rbs` (writes to `sig/`, which is gitignored)
- `bundle exec steep check` (after generating RBS)

**Manually maintained signatures (tracked):**
- `sig/_external.rbs` for external dependency stubs.
- `sig/_supplementary.rbs` for types rbs-inline cannot infer (e.g., `extend self`, singleton helpers).

**Example:**
```ruby
# frozen_string_literal: true
# rbs_inline: enabled

class Example
  attr_reader :name #: String

  # @rbs name: String -- The name to set
  # @rbs return: void -- No return value
  def initialize(name)
    @name = name
  end
end
```

## Testing Guidelines

- Integration tests are `spec/integration/` with `type: :puppeteer` helpers.
- Use `sinatra: true` for tests that need a local server.
- Use `match_golden` for screenshot comparisons where applicable.

## Agent Notes (Porting/Review)

### Source Code Porting

- When porting from upstream, use `packages/puppeteer-core/src/cdp/` as the primary source.
- Mirror upstream behavior, error messages, and option handling as closely as possible.
- Enable required CDP domains before relying on their events (see `CLAUDE/cdp_protocol.md`).

### Test Porting Guidelines

When porting tests from upstream `test/src/*.spec.ts` to `spec/integration/*_spec.rb`:

**Structure & Order**
- Keep `it` blocks in the **exact same order** as upstream
- Use the **same test names** (translated to Ruby style, e.g., `'should type into a textarea'`)
- Do NOT add extra `context`/`describe` wrappers unless upstream has them
- Do NOT add Ruby-specific tests in the middle; add them at the end if needed

**Ruby-Specific Tests → `*_ext_spec.rb`**
- When porting, separate Ruby-only features into `*_ext_spec.rb` files (e.g., `keyboard_ext_spec.rb`)
- Ruby-specific features include: block DSL (`page.keyboard { ... }`), `press('Shift') { press('Key') }` syntax
- Keep upstream-equivalent tests in the main spec file for easy comparison
- Example: `keyboard_spec.rb` (upstream port) + `keyboard_ext_spec.rb` (Ruby extensions)

**Test State Setup**
- Use `with_test_state` block instead of `include_context 'with test state'`
- Access test helpers via block arguments: `page:`, `server:`, `https_server:`, `browser:`, `browser_context:`
- Example:
  ```ruby
  it 'should click button' do
    with_test_state do |page:, server:, **|
      page.goto("#{server.prefix}/input/button.html")
      page.click('button')
      expect(page.evaluate('() => globalThis.result')).to eq('Clicked')
    end
  end
  ```

**Asset Files**
- `spec/assets/` files must be **identical** to upstream `test/assets/`
- Fetch assets directly: `wget https://raw.githubusercontent.com/puppeteer/puppeteer/main/test/assets/xxx`
- Do NOT hand-edit asset files; if upstream changes, re-fetch

**Code Translation**
- `page.evaluate(() => expr)` → `page.evaluate('() => expr')` (string form)
- `page.$('selector')` → `page.query_selector('selector')`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [YusukeIwaki/puppeteer-ruby](https://github.com/YusukeIwaki/puppeteer-ruby) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
