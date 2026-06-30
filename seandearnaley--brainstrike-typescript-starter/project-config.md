---
trigger: always_on
description: Welcome to the `brainstrike-typescript-starter` monorepo! This guide is designed to help both human developers and AI agents understand how to effectively contribute to this project.
---

# Contributor Guide for AI Agents and Developers

Welcome to the `brainstrike-typescript-starter` monorepo! This guide is designed to help both human developers and AI agents understand how to effectively contribute to this project.

## Codebase Overview

This is a pnpm workspace-powered monorepo using `pnpm` as the package manager. The structure is as follows:

- **`client/`**: The React + TypeScript client application (`brainstrike-client`).
  - Built with Vite, React 19+, Material UI, and Apollo Client 3.0
  - Uses GraphQL Code Generator for type-safe hooks and operations
  - Testing with Vitest and React Testing Library
- **`server/`**: The Node.js + TypeScript GraphQL API server (`brainstrike-server`).
  - Built with Apollo Server, Express, and TypeORM
  - PostgreSQL database with migration support
  - Comprehensive testing with Vitest
  - GraphQL Code Generator for resolvers and types

## Development Environment & Tips

- **Node.js**: Use Node.js version 18.19.1 or higher (as specified in package.json engines).
- **Package Manager**: This project uses `pnpm`. Install dependencies from the root of the monorepo:
  ```bash
  pnpm install
  ```
- **Running Development Servers/Tasks**: Use pnpm workspace commands to run scripts.
  - To run both client and server in development mode:
    ```bash
    pnpm dev
    ```
  - To run only the client:
    ```bash
    pnpm dev:client
    ```
  - To run only the server:
    ```bash
    pnpm dev:server
    ```
  - To build all packages:
    ```bash
    pnpm build
    ```
  - To build a specific package:
    ```bash
    pnpm build:client
    # or
    pnpm build:server
    ```
- **Adding Dependencies**:
  - To add a dependency to the client:
    ```bash
    pnpm add <package-name> --filter brainstrike-client
    ```
  - To add a dependency to the server:
    ```bash
    pnpm add <package-name> --filter brainstrike-server
    ```
  - To add a dev dependency to the root workspace:
    ```bash
    pnpm add -D <package-name> -w
    ```
- **Package Names**:
  - Client: `brainstrike-client`
  - Server: `brainstrike-server`
  - Root: `brainstrike-monorepo`

## Contribution and Style Guidelines

- **Primary Language**: TypeScript. Adhere to strict mode.
- **Client Framework**: React 19+ with functional components and hooks, Material UI for components.
- **Server Framework**: Apollo Server with Express, TypeORM for database access.
- **Database**: PostgreSQL with TypeORM migrations.
- **Styling**: Material UI components and Emotion for styling.
- **State Management**: Apollo Client for GraphQL state management.
- **Linting & Formatting**: ESLint and Prettier are used for linting and formatting.
  - Check for issues:
    ```bash
    pnpm lint
    ```
  - Auto-fix simple problems (recommended before committing):
    `bash
pnpm lint:fix
`
    These commands run across both client and server workspaces. Ensure your code is compliant before committing.
- **File & Directory Naming**:
  - **General**: kebab-case for files and directories (e.g., `my-feature/`, `utility-functions.ts`).
  - **React Components**: PascalCase for component files (e.g., `CardList.tsx`, `UserProfile.tsx`), and PascalCase for component exports.
  - **Custom Hooks**: camelCase starting with `use` (e.g., `useCardData.ts`, `useAuth.ts`).
- **Commit Messages**: Follow Conventional Commits specification.
  - **Format**: `<type>(<scope>): <subject>`
  - **Types**: `feat`, `fix`, `docs`, `style`, `refactor`, `test`, `chore`, `ci`, `perf`, `build`, `revert`.
  - **Scopes**: `client`, `server`, `api`, `ui`, `config`, `deps`, `core`, `test`, `types`, `security`, `db`.

## Testing Instructions

- **Frameworks**:
  - Vitest: For unit and integration tests (`.test.ts`, `.integration.test.ts`).
  - React Testing Library: For React component testing.
- **Running Tests**:

  - **Unit & Integration Tests (Vitest)**:
    - Run all tests across both client and server:
      ```bash
      pnpm test
      ```
    - Run tests for a specific workspace:
      ```bash
      pnpm test:client
      # or
      pnpm test:server
      ```
    - Run tests in watch mode for a specific workspace:
      ```bash
      pnpm --filter brainstrike-client test:watch
      # or
      pnpm --filter brainstrike-server test:watch
      ```
    - From within a package's directory, you can run its specific test script:
      ```bash
      cd client
      pnpm test # Runs Vitest for the client
      ```
      ```bash
      cd server
      pnpm test # Runs Vitest for the server
      ```

- **Test Location**: Tests should be co-located with the source files they are testing (e.g., `my-component.tsx` and `my-component.test.tsx` in the same directory).
- **Type Checking**: Ensure there are no TypeScript errors.
  ```bash
  pnpm typecheck
  ```
  This command runs TypeScript type checking across both workspaces.
- **CI Pipeline**: The Continuous Integration setup can be found in `.github/workflows/`.
- **Requirement**: All tests (unit, integration) and type checks must pass. New code or changes must include corresponding tests. Fix any errors until the entire suite is green.

## Database Setup


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [seandearnaley/brainstrike-typescript-starter](https://github.com/seandearnaley/brainstrike-typescript-starter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
