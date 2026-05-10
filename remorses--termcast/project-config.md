---
trigger: always_on
description: <!-- This AGENTS.md file is generated. Look for an agents.md package.json script to see what files to update instead. -->
---

<!-- This AGENTS.md file is generated. Look for an agents.md package.json script to see what files to update instead. -->

# Project Coding Guidelines

NOTICE: AGENTS.md is generated using `bun agents.md` and should NEVER be manually updated. only update PREFIX.md

ALWAYS use bun to install dependencies

---

# termcast specific rules

## Porting @raycast/api components and hooks to termcast

ALWAYS use termcast to import things, instead of relative imports. This is possible thanks to exports in package.json. for example:

import {List} from 'termcast'

ALWAYS use .tsx extension for every new file.

NEVER use mocks in vitest tests

When running the e2e vitest suite, ALWAYS use the repo scripts (`bun e2e`, `bun e2e <file>`, `bun e2e -u`). NEVER run `vitest` directly. e2e already passes -u so no need to pass again.

after running e2e see git diff to make sure we don't see unexpected changes in snapshots

prefer object args instead of positional args. as a way to implement named arguments, put the typescript definition inline

## see files in the repo

use `git ls-files | tree --fromfile` to see files in the repo. this command will ignore files ignored by git

## Goal

This project ports @raycast/api components and apis to use @opentui/react and other Bun APIs

We are basically implementing the package @raycast/api from scratch. DO NOT implement functions exported by @raycast/utils

This should be done one piece at a time, one hook and component at a time

## Porting a new Raycast component or feature

Here is the process to follow to implement each API:

- decide which component or hook or function we are porting
- read the .d.ts of the @raycast/api package for the component or hook
- generate a new file or decide to which file to add this new API in src folder
- start by adding a signature without any actual implementation. Only a function or class or constant without any actual implementation
- try typechecking with `bun run tsc`. fix any errors that is not related to the missing implementation (like missing returns)
- then think, is the signature the same as Raycast?
- start implementing the component or function, before doing this
  - decide on what @opentui/react components to use
  - do so by reading opentui .d.ts files and see available components
  - read .d.ts to understand available styling options and attributes
- typecheck
- if the added feature is a component or adds support for a new prop for a component, add an example usage component in the src/examples directory. create a descriptive name for it in the file. use simple-{component-name} for basic implementations examples
- if the implemented feature is function or other API, add an action in the file examples/miscellaneus.tsx, add a list item for the new feature, for example "show a error toast" if we are implementing toasts
- do not add an example if our feature is already covered by other example files
- DO NOT run the examples then. instead ask me to do it. do not add these as scripts in package.json
- typecheck to make sure the example is correct

## Rules

- for return type of React components just use any
- keep types as close as possible to rayacst
- DO NOT use as any. instead try to understand how to fix the types in other ways
- to implement compound components like `List.Item` first define the type of List, using a interface, then use : to implement it and add compound components later using . and omitting the props types given they are already typed by the interface, here is an example
- DO NOT use console.log. only use logger.log instead
- <input> uses onInput not onChange. it is passed a simple string value and not an event object
- to render examples components use renderWithProviders not render
- ALWAYS bind all class methods to `this` in the constructor. This ensures methods work correctly when called in any context (callbacks, event handlers, etc). Example:

  ```typescript
  constructor(options: Options) {
    // Initialize properties
    this.prop = options.prop

    // Bind all methods to this instance
    this.method1 = this.method1.bind(this)
    this.method2 = this.method2.bind(this)
    this.privateMethod = this.privateMethod.bind(this)
  }
  ```

```typescript
interface ListType {
  (props: ListProps): any
  Item: (props: ListItemProps) => any
  Section: (props: ListSectionProps) => any
}

const List: ListType = (props) => {
  // implementation
}

List.Item = (props) => {
  // implementation
}

List.Section = (props) => {
  // implementation
}
```

## keeping the implementation compatible with raycast

the goal of this project is to use same props and api as @racyast/api so try to follow raycast types and behaviour exactly

to understand behaviour (not covered by .d.ts) you MUST read the racyast docs using commands like this one, that reads the List component docs:

curl -s https://developers.raycast.com/api-reference/user-interface/list.md

> IMPORTANT! Add the ending .md to fetch markdown! Or it will return html!

You can see the full list of raycast docs pages using

curl -s https://developers.raycast.com/sitemap-pages.xml


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [remorses/termcast](https://github.com/remorses/termcast) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
