---
trigger: always_on
description: The Ruby LSP project aims to be a complete solution to provide an IDE-like experience for the Ruby language on VS Code.
---

# Project Overview

The Ruby LSP project aims to be a complete solution to provide an IDE-like experience for the Ruby language on VS Code.
Its parts are:

- `ruby-lsp` gem: language server implementation and extra custom functionality to support the VS Code extension. This
  is the top level of the repository
- Ruby code indexer: static analysis engine to support features like go to definition, completion and workspace
  symbols. This is entirely implemented inside `lib/ruby_indexer`
- Companion VS Code extension that includes several integrations. The extension is entirely implemented in the `vscode`
  directory
- Jekyll static documentation site. Fully implemented in `jekyll`

# Ruby LSP gem

## Architecture

The Ruby LSP is organized in components that composed the full language server functionality.

### Server plumbing

The basic server functionality for communicating with the LSP client, remembering client capabilities, handling requests
and notifications.

- `lib/ruby_lsp/server.rb`
- `lib/ruby_lsp/base_server.rb`
- `lib/ruby_lsp/utils.rb`
- `lib/ruby_lsp/client_capabilities.rb`
- `lib/ruby_lsp/global_state.rb`

### Requests and notifications

Requests and notifications are implemented in an extensible way so that add-ons can contribute to the base features
provided.

- `lib/ruby_lsp/requests/`: base request implementations
- `lib/ruby_lsp/listeners/`: static analysis logic, implementing through listeners. Traversal of ASTs is performed by a
  `Prism::Dispatcher` and listeners register for the node events that they are interested in handling. This allows
  listeners to encapsulate distinct logic without having to perform multiple traversals of the AST
- `lib/ruby_lsp/response_builders/`: builder pattern to allow multiple listeners to contribute to the same language
  server response

### Document storage

Document related information is saved in a hash stored of `{ uri => Document }`. The language server handles Ruby, RBS
and ERB files to provide Ruby features.

- `lib/ruby_lsp/store.rb`: document storage
- `lib/ruby_lsp/document.rb`: base document class
- `lib/ruby_lsp/ruby_document.rb`: Ruby document handling
- `lib/ruby_lsp/erb_document.rb`: ERB document handling
- `lib/ruby_lsp/rbs_document.rb`: RBS document handling

### Add-on system

The Ruby LSP includes an add-on system that allows other gems to define callbacks and listeners that can contribute to
features provided in the editor.

Examples:

- Contributing a code lens button to jump from Rails controller action to corresponding view
- Contributing location results when trying to go to definition on the symbol used to define a Rails callback
- Contributing diagnostics and formatting for a specific linting tool
- Displaying a window message warning

- `lib/ruby_lsp/addon.rb`: major implementation of the add-on system. Feature contributions are connected to listeners
  and response builders

## Testing

The gem uses a mix of unit tests, which are pure Ruby, and a custom built framework that matches response expectation to
fixture files.

For request or notification related tests that aren't using fixtures, the structure should use the provided test helpers:

```ruby
def test_feature_name
  source = <<~RUBY
    # Ruby code to test
  RUBY

  with_server(source) do |server, _uri|
    # Make LSP request
    # Assert response
  end
end
```

The custom built framework runs all language server features against the files in `test/fixtures`. If there's a file
with the same name under `test/expectations`, it will assert that the response matches what is expected for each request
that has an expectation file. If there aren't any expectation files, the feature will still run against the fixture to
verify that it does not raise. Fixture files are simply Ruby and expectation files are JSON ending in `.exp.json`.

## Type checking

The Ruby LSP codebase is fully typed with Sorbet's typed strict sigils using inline comment RBS annotations and RBI
files.

**Common RBS Patterns:**

```ruby
# Method signatures (placed above method definition)
#: (String name) -> void
def process(name)
  # ...
end

# Variable annotations (placed after assignment)
@documents = {} #: Hash[URI::Generic, Document]

# Attribute type declarations (placed above attribute)
#: String?
attr_reader :parent_class

# Generic types
#: [T] () { (String) -> T } -> T
def with_cache(&block)
  # ...
end

# Union and nullable types
result = nil #: (String | Symbol)?
```

Type syntax reference: https://sorbet.org/docs/rbs-support


## Commands

```bash
# Run all tests
bundle exec rake

# Run specific test file
bin/test test/requests/completion_test.rb

# Run tests matching a pattern
bin/test test/requests/completion_test.rb test_name_pattern

# Type check with Sorbet
bundle exec srb tc

# Lint with RuboCop
bin/rubocop

# Auto-fix RuboCop violations
bin/rubocop -a
```

# VS Code extension

The VS Code extension provides several integrations, some of which interact with the language server.

## Architecture

The extension's entrypoint is implemented in `vscode/src/extension.ts` and `vscode/src/rubyLsp.ts`. This is where we
handle activation, detecting workspaces, registering commands and subscribers.

### Integrations


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Shopify/ruby-lsp](https://github.com/Shopify/ruby-lsp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
