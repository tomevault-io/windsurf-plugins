---
trigger: always_on
description: Rubyzen is a Ruby architectural linter that lets you write lint rules as RSpec tests. Inspired by Konsist (Kotlin) and Harmonize (Swift), it wraps RuboCop AST to provide a high-level, easy-to-use API for enforcing architectural rules across a codebase.
---

# Rubyzen Architecture Guide

## What is Rubyzen

Rubyzen is a Ruby architectural linter that lets you write lint rules as RSpec tests. Inspired by Konsist (Kotlin) and Harmonize (Swift), it wraps RuboCop AST to provide a high-level, easy-to-use API for enforcing architectural rules across a codebase.

Instead of configuring YAML rules, you write standard RSpec tests:

```ruby
it 'controllers do not call ActiveRecord directly' do
  expect(controllers.all_methods.call_sites.with_name('where')).to zen_empty
end
```

## Core Concepts

Rubyzen has four main building blocks:

| Concept | Purpose | Example |
|---|---|---|
| **Declarations** | Domain objects wrapping AST nodes | `ClassDeclaration`, `MethodDeclaration`, `CallSiteDeclaration` |
| **Collections** | Typed arrays of declarations with filtering/aggregation | `ClassesCollection`, `MethodsCollection` |
| **Providers** | Mixins that add capabilities to declarations | `CallSiteProvider`, `BlocksProvider` |
| **Matchers** | RSpec matchers for asserting on collections | `zen_empty`, `zen_true { }`, `zen_false { }` |

## Data Flow

```
Project
  └── files → FileCollection
        ├── .classes → ClassesCollection
        │     ├── .all_methods → MethodsCollection
        │     │     ├── .parameters → ParametersCollection
        │     │     ├── .call_sites → CallSiteCollection
        │     │     ├── .if_statements → DeclarationCollection
        │     │     ├── .rescues → RescuesCollection
        │     │     └── .raises → RaisesCollection
        │     ├── .attributes → AttributesCollection
        │     ├── .macros → MacrosCollection
        │     ├── .rescues → RescuesCollection
        │     └── .raises → RaisesCollection
        ├── .modules → ModulesCollection
        ├── .call_sites → CallSiteCollection
        ├── .blocks → BlocksCollection
        │     └── .call_sites → CallSiteCollection
        ├── .constants → ConstantsCollection
        └── .requires → RequiresCollection
```

Every arrow is a method that returns a typed collection. Collections support chaining via filtering methods.

## Folder Structure

```
lib/rubyzen/
├── rubyzen.rb                    # Entry point, Zeitwerk loader, configuration, matchers
├── project.rb                    # Parses all .rb files, returns FileCollection
├── declarations/                 # Domain objects wrapping AST nodes
│   ├── file_declaration.rb
│   ├── class_declaration.rb
│   ├── module_declaration.rb
│   ├── method_declaration.rb
│   ├── parameter_declaration.rb
│   ├── call_site_declaration.rb
│   ├── block_declaration.rb
│   ├── constant_declaration.rb
│   ├── require_declaration.rb
│   ├── attribute_declaration.rb
│   ├── if_statement_declaration.rb
│   ├── macro_declaration.rb
│   ├── raise_declaration.rb
│   └── rescue_declaration.rb
├── collections/                  # Typed arrays with filtering/aggregation
│   ├── base_collection.rb        # Extends Array, provides filter method
│   ├── file_collection.rb
│   ├── classes_collection.rb
│   ├── modules_collection.rb
│   ├── methods_collection.rb
│   ├── parameters_collection.rb
│   ├── call_site_collection.rb
│   ├── blocks_collection.rb
│   ├── constants_collection.rb
│   ├── requires_collection.rb
│   ├── attributes_collection.rb
│   ├── macros_collection.rb
│   ├── raises_collection.rb
│   ├── rescues_collection.rb
│   └── declaration_collection.rb
├── providers/                    # Mixins included in declarations
│   ├── file_path_provider.rb
│   ├── line_number_provider.rb
│   ├── lines_of_code_provider.rb
│   ├── class_name_provider.rb
│   ├── source_code_provider.rb
│   ├── call_site_provider.rb
│   ├── blocks_provider.rb
│   ├── if_statements_provider.rb
│   ├── constants_provider.rb
│   ├── requires_provider.rb
│   ├── attributes_provider.rb
│   ├── macros_provider.rb
│   ├── raises_provider.rb
│   ├── rescues_provider.rb
│   ├── visibility_provider.rb
│   └── collection_filter_provider.rb
├── matchers/                     # RSpec custom matchers
│   ├── matcher_helpers.rb
│   ├── zen_empty_matcher.rb
│   ├── zen_true_matcher.rb
│   └── zen_false_matcher.rb
├── parsers/
│   └── a_s_t_parser.rb          # Wraps RuboCop AST ProcessedSource
├── cache/
│   └── parse_cache.rb           # SHA256-based in-memory parse cache
└── version.rb                    # Gem version constant

sample_project/
├── src/                          # Sample app with intentional violations
│   ├── controllers/
│   ├── models/
│   ├── presenters/
│   ├── repos/
│   ├── services/
│   ├── requests/
│   ├── tests/
│   └── config.rb
└── spec/                         # Lint rules as RSpec tests
    ├── spec_helper.rb            # Shared context with common collections
    ├── controllers/
    ├── models/
    ├── presenters/
    ├── tests/
    └── ...
```

## How the Pieces Connect

### Declarations include Providers

Each declaration includes providers as mixins to gain capabilities. The `node` and `parent` attributes are used by providers to traverse the AST.

```ruby
class MethodDeclaration
  include Rubyzen::Providers::CallSiteProvider    # adds .call_sites

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [perrystreetsoftware/Rubyzen](https://github.com/perrystreetsoftware/Rubyzen) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-29 -->
