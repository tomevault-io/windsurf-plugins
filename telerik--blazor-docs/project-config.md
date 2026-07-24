---
trigger: always_on
description: This guide outlines best practices for Razor component development and documentation in Blazor.
---

# Blazor Code Style Guide

This guide outlines best practices for Razor component development and documentation in Blazor.

## Razor File Structure

Use the following top-down order in `.razor` files:

1. `@page`
2. `@using`
3. `@namespace`
4. `@inherits`
5. `@implements`
6. `@inject`

Insert `<style>` or `<script>` between the component markup and `@code` block, unless the example revolves around them.

## `@code` Section Order

1. Component references
2. Component parameters
3. Methods/event handlers (logical order; lifecycle methods last)
4. Data generation
5. Class declarations

Use `#region` for longer sections if needed.

## Access Modifiers

- Razor properties & handlers: `private`
- Lifecycle handlers: `protected`
- Other classes & members: `public`

## Naming Conventions

- **PascalCase** for members (`Id`, `FirstName`)
- **camelCase** for local variables and lambdas
- Avoid repeating class names in properties (`Product.Name`, not `Product.ProductName`)
- Use specific and meaningful model names (`GridModel`, `Product`)
- For `Data`: prefer `[Component]Data` or meaningful plural (`GridData`, `Products`)
- For `@ref`: use `[Component]Ref` (`GridRef`)

## Components

- Minimal configuration for clarity
- Use self-closing tags if simple and one-lined
- Prefix parameter values with `@` unless literals
- Use `nameof()` for member names

**Parameter order:**

1. `@ref`
2. `Data` / `OnRead`
3. `TItem`, `TValue`
4. `TextField`, `ValueField`
5. `Value`, `ValueExpression`
6. Other parameters
7. Event handlers

Pair one-way bound params with `Changed` handlers. Break to new lines after ~80 characters.

## Event Handlers

- Use `On[Component][Event]` or meaningful verbs (`OnButtonClick`, `RebindGrid`)
- Use `new[Noun]` in simple lambdas (`newValue`)
- Use `args` for complex types or `*Args` types

Example:

```razor
<TelerikNumericTextBox OnValueChanged="@( (int newValue) => OnValueChanged(newValue) )" />
```

```cs
async Task OnValueChanged(int newValue) { }
```

## Data Generation

- Use `OnInitialized` or static data
- Use `List<T>`
- Avoid ID = 0
- Use `Random` for real-world-like values
- Prefer loops for brevity

## Cross-Platform Syntax

- Use `Path.DirectorySeparatorChar`, not `"/"` or `"\"`
- Match exact casing for files and folders

## Comments

- Place conceptual comments **outside** snippets
- Use short inline comments **inside** snippets only when needed

# Copilot Instructions for Documentation Style

This guide is adapted from the full Progress documentation style guide for use with GitHub Copilot or similar tools. It focuses on actionable, concise rules to help write clear, consistent, and accessible product documentation.

## Dcouemnting New Component Structure and Specifics

Every component documentation is placed in a separate file, which is located in the `docs/components` folder. 
This folder contains folders for each component, and each folder contains the documentation files for the component. Most common files are:

- `overview.md`: A high-level overview of the component, its purpose, and usage.
- `events.md`: A detailed description of the events that the component emits, including their parameters and usage.
- `templates.md`: If the component has templates, this file provides information about the templates that can be used with the component, including examples and usage guidelines.
- `appearance.md`: A file that describes the appearance of the component, including its styles, themes, and customization options.

!!!IMPORTANT !!! Each component folder has an accessibility folder which contains the accessibility documentation for the component. This folder is automatically generated. With that in mind, do not create such folder when creating a new component documentation.

## Tone and Voice

### Basic Things to Avoid

* Do not denigrate or insult any group of people.
* Do not use jokes at the expense of the users.
* Do not use "simply", "It's that simple", or "It's easy".
* Do not use "let's do something".
* Do not use "please note" and "at this time".
* Do not use third-person singular pronouns ("he" or "she"). Use "they" instead to avoid gender issues.
* Do not write in first-person plural ("we", "our").
* Do not use metaphors or abbreviations.

### Writing for Global Audiences

Many users are non-native English speakers and come from various cultures. The content we write and the language we use has to translate well and to provide no room for interpretation.

* Use simple, objective, and non-ambiguous language.
* Do not use culture-specific language or pop-culture references.
* Do not be too cutesy.  
* Keep sentences and paragraphs short and concise. If a sentence or a paragraph is becoming too long, split it.

## General Guidelines for Writing

### Write in American English

|American   |British   |
|:--        |:---      |
|behavior   |behaviour |
|color      |colour    |
|organize   |organise  |
|license    |licence   |

### Use the Active Voice

* Avoid using the Passive Voice whenever possible.
* Yet, if you opt for the Passive Voice, use it consistently within a sentence.

|Not that good                 |Much better
|:---                          |:---                                        

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [telerik/blazor-docs](https://github.com/telerik/blazor-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
