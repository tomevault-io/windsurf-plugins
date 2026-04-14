---
trigger: always_on
description: **fweep** is a front-end web application for creating maps of interactive fiction parser games (like Zork). It is built with TypeScript and React.
---

# Copilot Instructions

## Project Overview

**fweep** is a front-end web application for creating maps of interactive fiction parser games (like Zork). It is built with TypeScript and React.

## Process documentation

Read migration-process.md for information about how to handle migrations of the map data format, and how to write migration scripts when making breaking changes to the data model.

Read release-process.md for information about how to prepare a release.

## Special commands to the agent

- `@release <version>`: Prepare a release branch for the specified version, following the release checklist and guidelines in release-process.md. This includes drafting release notes, running automated validation, performing manual smoke tests, and opening a PR to merge the release branch back to main when ready.

## Tech Stack

- **Language:** TypeScript
- **Platform:** Web (browser-based, front-end only)

## Coding Conventions

- Use TypeScript.
- Enable and preserve strict TypeScript settings.
- Prefer `const` over `let`; avoid `var`.
- Use named exports over default exports.
- Use descriptive variable and function names.
- Keep functions small and focused on a single responsibility.
- Use async/await over raw Promises where applicable.
- Prefer explicit types for public APIs and core domain models.

## File & Folder Structure

- Keep source code in `frontend/src/`.
- Keep tests in a `frontend/__tests__/` directory.
- Use kebab-case for file names (e.g., `map-editor.tsx`).

## Testing

- Write unit tests for utility functions and core logic.
- Test files should be named `*.test.ts`, `*.spec.ts`, `*.test.tsx`, or `*.spec.tsx` as appropriate.
- Write test-first when adding new features or fixing bugs. 
- Use Test-Driven Development (TDD) principles: write a failing test, implement the feature, then refactor.
- Keep tests in the centralized `frontend/__tests__/` directory unless there is a strong reason to do otherwise.
- Use Jest (with ts-jest, ESM mode) and React Testing Library.
- Use `jest.fn<FunctionSignature>()` (single type parameter with the full function type) instead of the legacy `jest.fn<ReturnType, ArgsType>()` two-parameter form.
- Import `jest` from `@jest/globals` in test files that need `jest.fn`, `jest.spyOn`, etc., since Jest globals are not automatically available in ESM mode.

## Git

- Write clear, concise commit messages in imperative mood (e.g., "Add room linking feature").
- Keep commits focused on a single change.

## Agent Guidance

- When creating new features, start by understanding the existing code structure before making changes.
- Prefer small, incremental changes over large rewrites.
- Always check for and fix TypeScript and React errors after making changes.
- When unsure about a design choice, ask before proceeding.

## Domain Definitions

- **Interactive Fiction:** A genre of games where players interact with the game world through text input, often involving puzzles and exploration.
- **Parser Game:** A type of interactive fiction game where players input text commands to interact with the game world.
- **Parser:** The component that processes user input and translates it into actions within the game world.
- **Map:** A visual representation of the rooms and connections in an interactive fiction game.
- **Map Editor:** The main interface of fweep where users can create and edit their maps.
- **Room:** A location in the interactive fiction game, which can have connections to other rooms.
- **Connection:** A link between two rooms, which can be directional (one-way) or bidirectional (two-way).
- **Direction:** A label for traversing from one room to another, such as "north", "southeast", "up", "in", "out", etc. Some games may have custom directions beyond the standard compass directions. A room maps directions to connections, and multiple directions may map to the same connection.
- **Source Room:** The room from which a one-way connection originates.
- **Target Room:** The room to which a one-way connection leads.
- **Item:** An object that can be placed in a room, which may have properties and interactions.
- **Scenery:** An item that cannot be picked up by the player but can be interacted with or described in the game world. 
- **NPC (Non-Player Character):** A character in the game that is not controlled by the player, which can have interactions and behaviors.
- **Puzzle:** A challenge or obstacle in the game that requires the player to solve it to progress.
- **Inventory:** A collection of items that the player has collected during the game.
- **Container:** An item that can hold other items, such as a box or a chest.
- **Supporter:** An item that can support other items, such as a table or a shelf.
- **Dark Room:** A room that is initially dark and requires a light source to see its contents.
- **Light Source:** An item that can illuminate a dark room, such as a lamp or a torch.

## Major Features

- **Room Creation:** Users can create rooms and specify their properties (name, description, etc.).
- **Connection Creation:** Users can create connections between rooms, specifying one or more room-direction bindings and the connection type (one-way or two-way).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rileypb) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
