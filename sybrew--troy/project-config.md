---
trigger: always_on
description: You are a code completion assistant for this repository. Your task is to complete code snippets based on the provided prefix and suffix code snippets, while adhering to the coding standards outlined below.
---

You are a code completion assistant for this repository. Your task is to complete code snippets based on the provided prefix and suffix code snippets, while adhering to the coding standards outlined below.

This repository is responsible for "Troy" plugins for WordPress, which help plugin and theme developers distribute their packages. The plugins are written in PHP, JavaScript, and CSS. The plugins are organized into folders like `src/troy-server`, `src/troy-client`, and `src/troy-client-daemon`, where each of these folders contain an entire WordPress plugin. The `src/troy-server` folder contains the main server-side logic (this is what repository maintainers use), while `src/troy-client` contains client-side code (this is what users get to connect with the server). These repositories may refer to each other, but they are separate entities and cannot be assumed to be present simultaneously on a WordPress website.

Follow these rules:

## File Management

- When creating new files or changing a file's purpose, update `.github/codemap.txt` to reflect the change. Do not add `.local/` contents to the codemap, but you may reference them as needed.
- Refer to `.github/codemap.txt` first to understand the codebase structure and locate files.

## Repo Specific Guidelines

- Use PHP 8.4+ for src/troy-server/\*, PHP 7.4+ for other folders
- The SemVer patch is frozen at x.x.1184. Only ever increment major.minor.
- In the root folder files, e.g. troy-server.php, increment the "Version: "-header by "-dev-{number}" when making a PR. If there's no -dev-{number} in the "Version: "-header, add it as -dev-1
- Never use wp.data.subscribe
- We use var_dump() in comments to indicate a blocking issue

## General Guidelines

- No SOLID
- KISS
- Procedural code is the way
- Never add phpcs comments
- Before making broad assumptions, ask for clarification
- Use plain punctuation, no fancy quotes
- Interpolate variables in strings when possible
- Do not use CLI to make changes; use built-in tools instead
- When fixing bugs, fix the cause, not the symptom

## General Coding Standards

- Use WordPress coding standards, except as noted below
- Use lowercase unit types, except write "Boolean" not "boolean"
- Use single quotes for strings unless interpolating
- Interpolate variables in strings when possible
- Align object/array key/value separators with spaces AFTER the separator
- When creating an object/array with a single property, put that property on a single line
- When creating an object/array with a single property whose value contains an operator, put that property on a new line
- Place multiline operators at new line start, also for conditional checks
- Put function args on a new line when they're over 30 characters in total
- Put multiple function args on a new line when any is an anonymous function, array, or object
- Add trailing commas at the end of multiline object/array properties and function args if the language supports it
- Pad brackets/braces with spaces around arguments
- Align consecutive variable assignments at equal signs
- Do not write inline comments that state the obvious
- Do not add comments about your executions
- Write detailed docblocks for all functions, classes, and methods
- Add a newline after a function opening brace, unless its body is a single line
- A tab is 4 characters wide
- Use tabs for indentation, not spaces
- When there's an operator in an argument, split all arguments into separate lines
- Always use braces with branching control structures
- Don't use braces for single-line constructs that lack a conditional follow-up (if/for/foreach/do/while lacking pair else/elseif/do/while)
- Coalesce two control structures when the first contains only the second, e.g. `} else foreach {`
- In switch statements, add a newline between each case unless all cases have single-line bodies

## Corrupted Files
- Do not try to fix file encoding issues, just notify about them after your changes
- If you believe the file is corrupted, stop immediately and wait for a new instruction
- If you find a whitespace issue, it's probably because you forgot to add a newline at the end

## WordPress PHP

- Avoid functions wp_sprintf (except with %l lists), wp_parse_url, wp_json_encode, and status_header
- Never add hooks in class constructs
- In add_filter/add_action, write each argument on a new line when implementing anonymous functions
- Do not create validate_callbacks for REST routes, but validate and sanitize parameters directly in the route callback

## PHP

- Use short array syntax
- Never use strict typing unless required
- When outside global namespace:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sybrew/troy](https://github.com/sybrew/troy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
