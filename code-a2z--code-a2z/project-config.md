---
trigger: always_on
description: Code A2Z is a collaborative blogging platform built as a monorepo with separate client and server applications. Contributors can create, manage, and share blog posts about their projects with markdown support, customizable templates, and role-based access control.
---

# GitHub Copilot Instructions for Code A2Z

## Project Overview

Code A2Z is a collaborative blogging platform built as a monorepo with separate client and server applications. Contributors can create, manage, and share blog posts about their projects with markdown support, customizable templates, and role-based access control.

## Repository Structure

```
code-a2z/
├── client/          # React + TypeScript + Vite frontend
│   └── src/
│       ├── modules/       # Feature modules (home, editor, profile, etc.)
│       ├── shared/        # Shared utilities and components
│       │   ├── components/    # Atomic design pattern
│       │   │   ├── atoms/         # Basic UI elements
│       │   │   ├── molecules/     # Composite components
│       │   │   └── organisms/     # Complex sections
│       │   ├── hooks/         # Custom React hooks
│       │   ├── states/        # Jotai state atoms
│       │   └── utils/         # Helper functions
│       ├── infra/         # Infrastructure layer
│       │   ├── rest/          # API clients
│       │   ├── states/        # Global state
│       │   └── types/         # TypeScript types
│       ├── config/        # Configuration files
│       └── assets/        # Static assets
├── server/          # Node.js + Express + MongoDB backend
│   └── src/
│       ├── controllers/   # Request handlers by domain
│       ├── routes/        # API route definitions
│       ├── models/        # Mongoose models
│       ├── schemas/       # Mongoose schemas
│       ├── middlewares/   # Express middlewares
│       ├── utils/         # Helper functions
│       ├── config/        # Server configuration
│       ├── constants/     # Constant values
│       ├── logger/        # Winston logging
│       └── typings/       # Type definitions
└── docs/            # Project documentation
```

## Tech Stack

### Client

- **Framework**: React 19 with TypeScript
- **Build Tool**: Vite (using Rolldown)
- **State Management**: Jotai
- **UI Library**: Material-UI (MUI)
- **Routing**: React Router v7
- **Editor**: EditorJS
- **Styling**: Emotion CSS-in-JS

### Server

- **Runtime**: Node.js (>=18.0.0)
- **Framework**: Express.js
- **Database**: MongoDB with Mongoose
- **Authentication**: JWT with bcrypt
- **File Upload**: Multer + Cloudinary
- **Email**: Nodemailer + Resend
- **Security**: Helmet, HPP, sanitize-html
- **Rate Limiting**: rate-limiter-flexible
- **Logging**: Winston + Morgan

## Coding Standards

### File Naming Conventions

#### Client (TypeScript/React)

- **Components**: `kebab-case` files, `PascalCase` exports
  - `index.tsx` - Main component file
  - `render-menu.tsx` - Sub-components
  - `category-button.tsx` - Feature components
- **Hooks**: `kebab-case` with `use-` prefix
  - `use-auth.ts`
  - `use-editor.ts`
- **Utils**: `kebab-case.ts`
  - `api-interceptor.ts`
  - `date.ts`
  - `regex.ts`
- **State**: `kebab-case` files with descriptive names
  - `auth-state.ts`
  - `editor-state.ts`
- **Types**: `kebab-case.ts`
  - `user-types.ts`
  - `project-types.ts`

#### Server (JavaScript/Node.js)

- **Controllers**: `kebab-case.js` with action name
  - `signup.js`
  - `create-project.js`
  - `upload-image.js`
- **Routes**: `*.routes.js`
  - `auth.routes.js`
  - `project.routes.js`
- **Models**: `*.model.js`
  - `user.model.js`
  - `project.model.js`
- **Schemas**: `*.schema.js`
  - `user.schema.js`
  - `project.schema.js`
- **Middlewares**: `*.middleware.js` or `*.limiter.js`
  - `auth.middleware.js`
  - `auth.limiter.js`
- **Utils**: `kebab-case.js` or descriptive names
  - `response.js`
  - `regex.js`

### Code Organization

#### Client Module Structure

Each feature module follows this pattern:

```
module-name/
├── index.tsx              # Main module component
├── components/            # Module-specific components
│   ├── component-name.tsx
│   └── index.tsx
├── hooks/                 # Module-specific hooks
│   └── index.ts
├── states/                # Module-specific state
│   └── index.ts
└── constants/             # Module-specific constants
    └── index.ts
```

#### Server Domain Structure

Each domain (auth, project, user, etc.) follows this pattern:

```
domain/
├── controller-action.js   # One controller per action
└── utils/                 # Domain-specific utilities (if needed)
    └── index.js
```

### Component Patterns

#### React Components

- Use **functional components** with hooks
- Export default for main component
- Use TypeScript interfaces for props
- Follow **Atomic Design** pattern in `shared/components`
  - Atoms: Basic elements (Button, Typography, Modal)
  - Molecules: Simple combinations (InputField, SearchBar)
  - Organisms: Complex sections (Navbar, Sidebar, Comments)

Example:

```typescript
const ComponentName = ({
  prop1,
  prop2,
}: {
  prop1: string;
  prop2: number;
}) => {
  // Component logic
  return <div>...</div>;
};

export default ComponentName;
```

#### Server Controllers

- One function per file
- Use async/await
- Always use `sendResponse` utility for responses
- Add JSDoc comments for API endpoints

Example:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Code-A2Z/code-a2z](https://github.com/Code-A2Z/code-a2z) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
