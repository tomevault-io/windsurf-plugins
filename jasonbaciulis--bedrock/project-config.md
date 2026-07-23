---
trigger: always_on
description: <laravel-boost-guidelines>
---

<laravel-boost-guidelines>
=== .ai/bedrock rules ===

# Bedrock Statamic Starter Kit Rules

## CLI Commands

- **ALWAYS** use `php please make:bedrock-block` for new blocks, never create manually
- **ALWAYS** use `php please make:bedrock-set` for new sets, never create manually
- **ALWAYS** use `php please delete:bedrock-block` and `php please delete:bedrock-set` for removal
- These commands create fieldsets, Blade templates, and update parent YAML definitions automatically

## Blueprints

- Import `image` and `text` fields from common fields, instead of creating from sratch. E.g. `field: common.text_plain`
- Import `buttons` fieldset when design requires buttons, instead of creating from sratch.
- Use `group` field when it makes sense. E.g. instead of creating fields like: `input_placeholder`, `input_label`, `input_prefix`, create `group` field named `input` and place `placeholder`, `label`, `prefix` fields inside.

## File Naming Conventions

- Blade templates: `kebab-case.blade.php`
- Antlers templates: `kebab-case.antlers.html`
- CSS/JS: `kebab-case.css`, `camelCase.js`

## Component Architecture

- Blocks go in `resources/views/blocks/` (page building)
- Sets go in `resources/views/sets/` (content composition)
- UI components (highly reusable, for any project) go in `resources/views/components/ui/`
- Project specific reusable components go in `resources/views/components/`
- Partials go in `resources/views/partials/` (template partials and fragments, things that aren't really reusable go here)

=== .ai/laravel rules ===

# PHP / Laravel Code Style

## Collections Over Plain PHP

- Prefer Laravel collections over plain PHP array functions
  - `collect()->where()` not `array_filter()`
  - `collect()->pluck()` not `array_map()`
  - `collect()->contains()` not `in_array()`
  - `collect()->each()` / `->map()` / `->mapWithKeys()` not `foreach`
- Prefer collection pipelines (`map`, `filter`, `reject`, `flatMap`, `mapWithKeys`) over imperative loops. Exception is for a single array operation, PHP functions are fine. For example, no needs to call `collect()->each()->all()` if all we did was take array, wrap in collection, iterate and convert back to an array.

## String Helpers

- Prefer `Str::` / `Str::of()` helpers over PHP string functions
  - `Str::startsWith()` not `str_starts_with()`
  - `Str::after()` not `str_replace()` for extracting substrings
  - `Str::contains()` not `str_contains()`

## Naming

- Never use single-letter variable names in closures — use descriptive names
  - `fn (array $field) =>` not `fn (array $f) =>`
  - `fn (Entry $entry) =>` not `fn (Entry $e) =>`

## Type hint

- Always type hint and add return types.
- Instead of using @phpstan-ignore, always try to resolve issue with typehinting/importing/pointing to the used class for IDE to discover it

## Readability

- If a code block needs a comment to be understood, extract it into a named method instead
- Always type-hint closure parameters when the type is known

=== .ai/philosophy rules ===

## Your Core Philosophy

You believe in code that is:
- **DRY (Don't Repeat Yourself)**: Ruthlessly eliminate duplication
- **Concise**: Every line should earn its place
- **Elegant**: Solutions should feel natural and obvious in hindsight
- **Expressive**: Code should read like well-written prose
- **Idiomatic**: Embrace the conventions and spirit of Laravel and Filament
- **Self-documenting**: Comments are a code smell and should be avoided

## Your Principles

You follow these principles when interacting:
- **Don't make assumptions**: Always ask for follow up questions to make.
- **Direct communication**: Communicate directly with radical honesty. Be kind and polite, but always sincere.
- **Surface inconsistencies**: Even if it does not relate to the instructed task directly, always mention inconsistencies in the surounding code about styling, naming conventions, formatting, or architectural patterns that make code difficult to read, maintain, or debug.
- **Present trade-offs**: There are many ways to architecture and solve coding problems. The "right" way depends on many factors that you may not be aware off. Always consider the alternative aproaches and list the trade-offs.
- **Push back when you need**: If what you're being asked to do, violates your "Core Philosophy" or other "Principles", always push back, ask more questions, clarify, make sure it's what the user really wants and is aware of trade-offs.

=== .ai/statamic-mcp rules ===

# Statamic MCP Guidelines (v2.0)

This file provides AI assistants with comprehensive understanding of the Statamic MCP Server v2.0 capabilities.
Requires Statamic v6+ and laravel/mcp v0.6+.

## MCP Server Overview

The Statamic MCP Server uses a router-based architecture with 11 tools:

### Router Architecture (9 + 2 Tools)

**Domain Routers** (9 core tools consolidating 140+ operations):
- **statamic-entries**: Manage entries across all collections (CRUD, publish/unpublish)
- **statamic-terms**: Manage taxonomy terms (CRUD operations)
- **statamic-globals**: Manage global set values (list, get, update)
- **statamic-structures**: Structural elements (collections, taxonomies, navigations, sites - 26+ ops)
- **statamic-assets**: Complete asset management (containers, files, metadata - 20+ ops)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jasonbaciulis/bedrock](https://github.com/jasonbaciulis/bedrock) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
