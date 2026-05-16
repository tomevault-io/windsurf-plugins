---
trigger: always_on
description: `codenames` is a library that converts any numerical value into a human-readable name following a specific theme (e.g. cities).
---

# Codenames Library Development Guide

`codenames` is a library that converts any numerical value into a human-readable name following a specific theme (e.g. cities).

```
1 -> "moscow"
2 -> "london"
3 -> "paris"
```

One of the primary use cases is to generate unique deployment URLs for preview environments:

```
PR 1234 -> https://london.example.com
```

## API Example

```typescript
import codename from "codenames/cities-20";

const name = codename(1234); // "london"
```

## Technology Stack

- **Runtime**: Bun
- **Language**: TypeScript
- **Formatter**: Prettier
- **Build**: Zero dependencies preferred
- **CLI**: Command-line interface included

## Project Structure

- `/words/` - Raw theme wordlists (.txt files)
- `/dist/` - Compiled TypeScript output
- `/scripts/` - Build and generation scripts
- `index.ts` - Main entry point
- `cli.ts` - Command-line interface
- Theme files follow pattern: `{theme}-{count}.ts` (e.g., `cities-20.ts`)

## Core Commands

- `bun install` - Install dependencies
- `bun add <name>` - Add a dependency
- `bun run build` - Build the project
- `bun run <script>` - Run a script
- `bun test` - Run tests
- `npx codenames <number>` - Generate codename from CLI

## CLI Implementation

- Use `#!/usr/bin/env node` shebang for Node.js compatibility
- Support both `codenames` and `cn` as command aliases
- Accept theme selection via flags: `--theme cities` or `-t cities`
- Support list size selection: `--size 20` or `-s 20`
- Include `--help` command with usage examples
- Return non-zero exit codes for errors
- Support piping and stdout/stderr properly

## Coding Principles

- Write lean, clear, and efficient code without over-engineering
- Prioritize readability and maintainability over cleverness
- Use descriptive variable names that clearly indicate purpose
- Keep functions small and focused on a single responsibility
- Avoid premature optimization - make it work, then make it fast if needed
- Use ES6+ features: arrow functions, destructuring, template literals, optional chaining
- NO COMMENTS unless explicitly requested by user

## Architecture & Design

- Structure code with clear separation of concerns
- Export a simple, intuitive public API
- Keep internal implementation details private
- Design for extensibility without compromising simplicity
- Use pure functions where possible for easier testing and deterministic output (same input → same output)
- Use deterministic hashing for consistent output
- Consider offering multiple wordlists or themes
- Ensure collision resistance appropriate for use case
- Provide options for output format

## Error Handling

- Throw meaningful errors for invalid inputs (NaN, Infinity, negative numbers)
- Use specific error classes when appropriate (e.g., `InvalidInputError`)
- Include helpful error messages with suggestions for resolution
- Handle edge cases: decimals, very large numbers, non-numeric inputs
- Provide graceful fallbacks where appropriate

## Wordlist/Theme Guidelines

### Theme Structure

- Each theme should have a minimum of 100 unique entries
- Raw wordlists stored as `.txt` files in `/words/` directory
- Generated TypeScript files placed in project root
- Use TypeScript const arrays with as const assertion
- Sort entries alphabetically for consistency
- Support multiple list sizes: 10, 20, 30, 50, and 100 entries
- Default list size is 20 entries
- Default theme is cities (available without configuration)
- Ensure all themes are appropriate for professional use

### Creating New Themes

1. Create a `.txt` file in `/words/` directory (e.g., `/words/mythical.txt`)
2. Add one word per line, lowercase, alphabetically sorted
3. Run `bun run scripts/generate.ts` to generate sized variants
4. Generated files will appear as `mythical-10.ts`, `mythical-20.ts`, etc.

Example generated file structure:

```typescript
// mythical-20.ts
export const words = [
  "dragon",
  "griffin",
  "phoenix",
  // ... 20 entries total
] as const;

export default function codename(input: number): string {
  // implementation
}
```

### Built-in Themes

- **Cities**: paris, london, tokyo, rome, berlin, madrid, sydney, vienna, athens, dublin
- **Animals**: cat, dog, bird, fish, cow, pig, duck, bee, fox, owl
- **Colors**: red, blue, green, yellow, black, white, gray, pink, orange, purple
- **Emotions**: love, hate, joy, sad, fear, mad, happy, angry, glad, calm
- **Food**: bread, milk, egg, rice, meat, fish, cake, apple, cheese, pasta
- **Nature**: tree, sun, sky, rain, moon, star, wind, sea, water, rock
- **Snacks**: chips, nuts, cookie, pretzel, popcorn, candy, fruit, cheese, cracker, yogurt
- **Space**: star, moon, sun, mars, venus, earth, saturn, jupiter, mercury, pluto

### Wordlist Generation Process

- Source wordlists are stored in `/words/*.txt` files
- Use `scripts/generate.ts` to create sized variants (10, 20, 30, 50, 100)
- Generated TypeScript files are placed in the project root
- Each generated file exports a default codename function
- Maintain alphabetical order in both source and generated files

### Theme Requirements

- Entries must be lowercase, alphanumeric strings
- No special characters or spaces (use hyphens if needed)
- Avoid offensive or controversial terms

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kriasoft/codenames](https://github.com/kriasoft/codenames) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
