---
trigger: always_on
description: This document provides guidelines for AI coding agents working on the Ackee codebase.
---

# Agent Guide for Ackee

This document provides guidelines for AI coding agents working on the Ackee codebase.

## Project Overview

Ackee is a self-hosted Node.js analytics tool built with:

- **Backend**: Node.js (ESM modules), Express, Apollo Server (GraphQL), Mongoose (MongoDB)
- **Frontend**: React (with createElement as `h`), Apollo Client, SCSS
- **Build**: Custom build script (`build.js`), Rosid handlers
- **Testing**: AVA test framework
- **Code Quality**: ESLint + Prettier (via @electerious configs)

## Commands

### Build

```bash
npm run build              # Production build
npm run build:pre          # Development build (BUILD_ENV=pre)
npm start                  # Build and start server
```

### Development

```bash
npm run dev                # Start with nodemon (auto-rebuild + restart)
npm run server             # Start server without building
```

### Testing

```bash
npm test                   # Run lint + all tests
npm run lint               # ESLint + Prettier check only
ava                        # Run all tests without linting
ava test/path/to/file.js   # Run a single test file
ava test/**/*domains*.js   # Run tests matching pattern
ava --watch                # Run in watch mode
```

### Code Quality

```bash
npm run eslint             # Check JavaScript with ESLint
npm run prettier -- --check # Check formatting
npm run format             # Auto-fix ESLint + Prettier issues
```

### Health Check

```bash
npm run healthcheck        # Run health check script
```

## Code Style Guidelines

### General Principles

- Use **ES modules** (`.js` files with `type: "module"` in package.json)
- No TypeScript - pure JavaScript with JSDoc comments where needed
- Functional programming style preferred
- Keep code simple, readable, and minimal

### Imports

- Use `.js` extensions in all import paths
- Group imports logically: external deps → internal modules → utils
- Use named exports for utilities, default exports for main components/resolvers

```javascript
import { randomUUID as uuid } from 'node:crypto'
import Domain from '../models/Domain.js'
import sortByProp from '../utils/sortByProp.js'
```

### File Naming

- **Backend**: camelCase for files (e.g., `domains.js`, `requireAuth.js`)
- **Frontend Components**: PascalCase (e.g., `Input.js`, `Dashboard.js`)
- **Frontend Hooks**: camelCase with `use` prefix (e.g., `useDomains.js`)
- **Constants**: camelCase files (e.g., `routes.js`, `intervals.js`)

### Formatting

- Uses Prettier via `@electerious/prettier-config`
- Tabs for indentation (configured in Prettier)
- Single quotes for strings
- Trailing commas in multi-line structures
- **Do not manually format** - run `npm run format` instead

### React Patterns

- Use `createElement as h` instead of JSX
- Define PropTypes for all components
- Use functional components with hooks
- Custom hooks follow `use*` naming convention

```javascript
import { createElement as h } from 'react'
import PropTypes from 'prop-types'

const Component = (props) => {
  return h('div', { className: 'example' }, props.children)
}

Component.propTypes = {
  children: PropTypes.node,
}

export default Component
```

### GraphQL Patterns

- Use `gql` template tag from `@apollo/client`
- Define fragments in separate files
- Mutations return `{ success, payload }` structure

```javascript
const QUERY = gql`
  query fetchDomains {
    domains {
      ...domainFields
    }
  }
  ${domainFields}
`
```

### Error Handling

- Use `KnownError` class for user-facing errors
- Catch and transform ValidationErrors from Mongoose
- Always handle promise rejections
- Use `signale` for logging (not `console.log`)

```javascript
try {
  entry = await domains.add(input)
} catch (error) {
  if (error.name === 'ValidationError') {
    throw new KnownError(messages(error.errors))
  }
  throw error
}
```

### Middleware Pattern

- Resolvers use `pipe()` utility to compose middleware
- Common middleware: `requireAuth`, `blockDemoMode`

```javascript
createDomain: pipe(requireAuth, blockDemoMode, async (parent, { input }) => {
  const entry = await domains.add(input)
  return { payload: entry, success: true }
})
```

### Database Patterns

- Export named functions for CRUD operations
- Use `response()` transformer to shape data
- Use `enhance()` pattern for consistent transformations

```javascript
export const get = async (id) => {
  const enhance = (entry) => {
    return entry == null ? entry : response(entry)
  }
  return enhance(await Domain.findOne({ id }))
}
```

### Testing with AVA

- Use `test.serial()` for tests that depend on execution order
- Import AVA as `test from 'ava'`
- Use `test.before`, `test.after.always`, `test.beforeEach`, `test.afterEach.always`
- Test context (`t.context`) stores shared state (e.g., tokens)
- Organize tests in folders matching source structure

```javascript
import test from 'ava'
import { api, cleanup, fillDatabase, gql } from './_utils.js'

test.beforeEach(fillDatabase)
test.afterEach.always(cleanupDatabase)

test.serial('create domain', async (t) => {
  const { json } = await api(base, body, t.context.token.id)
  t.true(json.data.createDomain.success)
  t.is(json.data.createDomain.payload.title, expectedTitle)
})
```

## ESLint Rules


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [electerious/Ackee](https://github.com/electerious/Ackee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
