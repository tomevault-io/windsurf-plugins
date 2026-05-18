---
trigger: always_on
description: This is a Wasp full-stack web application project. Always consider the following background information and rules when planning and implementing app features.
---

# General Info

This is a Wasp full-stack web application project. Always consider the following background information and rules when planning and implementing app features.

## Quick Reference

### Common Patterns
- Define app structure in `main.wasp` or `main.wasp.ts`
- Define data models in `schema.prisma`
- Group feature code in `src/features/{featureName}`
- Group feature config definitions (e.g. routes, pages, operations, etc.) into sections within the Wasp config file using the `//#region` directive.
  - e.g. `//#region {FeatureName}` ... `//#endregion`
- Use operations (queries/actions) for client-server communication
- Import from 'wasp/...' not '@wasp/...'
- Document features in `./ai/docs` with:
  - One markdown file per feature (e.g. `./ai/docs/{featureName}.md`)
  - Operations specifications and data models
  - User workflows and business logic
  - Update documentation when implementing feature changes
- Reference the relevant `./ai/docs` files when writing or modifying feature code

### Common Issues
- If imports aren't working, check path format ('wasp/...' not '@wasp/...')
- If entities aren't available in operations, check they're listed in the operation definition
- If auth isn't working, ensure auth config is properly set up in main.wasp

## Background Information

### What is Wasp

- Wasp (Web Application SPecification language) is a declarative, statically typed, domain-specific language (DSL) for building modern, full-stack web applications.
- Unlike traditional frameworks that are sets of libraries, Wasp is a simple programming language that understands web app concepts and generates code for you.
- Wasp integrates with React (frontend), Node.js (backend), and Prisma (database ORM) to create full-stack web applications with minimal boilerplate.
- The Wasp compiler reads your declarative configuration and generates all the necessary code for a working web application.

### Wasp Project Structure

- A Wasp project consists of a `main.wasp` (or `main.wasp.ts`) file in the root directory that defines the app's configuration.
- The `schema.prisma` file in the root directory defines your database models.
- Your custom code lives in the `src/` directory, which contains client-side and server-side code.
- Wasp generates additional code that connects everything together when you run your app.
- Feature code should be organized in `src/features/{featureName}` directories.

### The main.wasp File

- The `main.wasp` file is the central configuration file that defines your application structure.
- It contains declarations for app settings, pages, routes, authentication, database entities, and operations (queries and actions).
- Example structure:
  ```
  app myApp {
    wasp: {
      version: "^0.16.0"
    },
    title: "My App",
  }
  
  route HomeRoute { path: "/", to: HomePage }
  page HomePage {
    component: import { HomePage } from "@src/client/pages/HomePage.tsx"
  }
  
  query getTasks {
    fn: import { getTasks } from "@src/server/queries.js",
    entities: [Task]
  }
  ```

### Wasp Database and Entities

- Wasp uses Prisma for database access, with models defined in `schema.prisma`.
- Prisma models automatically become Wasp Entities that can be used in operations.
- Wasp reads the Prisma schema to understand your data model and generate appropriate code.
- Example Prisma model:
  ```
  model Task {
    id          Int      @id @default(autoincrement())
    description String
    isDone      Boolean  @default(false)
    user        User     @relation(fields: [userId], references: [id])
    userId      Int
  }
  ```

### Wasp Operations

- Operations are how Wasp handles client-server communication.
- Queries (read operations) and Actions (write operations) are defined in the main.wasp file.
- Operations automatically handle data fetching, caching, and updates.
- Operations reference entities to establish proper access patterns and dependencies.
- Example query and action:
  ```
  query getTasks {
    fn: import { getTasks } from "@src/server/operations",
    entities: [Task]
  }
  
  action createTask {
    fn: import { createTask } from "@src/server/operations",
    entities: [Task]
  }
  ```

### Wasp Auth

- Wasp provides built-in authentication with minimal configuration.
- Auth can be configured with username/password, social providers (Google, GitHub, etc.), or verified email and password .
- Wasp generates all necessary auth routes, middleware, and UI components.
- Example auth configuration:
  ```
  app myApp {
    // ... other config
    auth: {
      userEntity: User,
      methods: {
        usernameAndPassword: {},
        google: {}
      },
      onAuthFailedRedirectTo: "/login"
    }
  }
  ```

### Advanced Features

#### Jobs and Workers

- Wasp supports background jobs with the `job` declaration in main.wasp.
- Jobs can be scheduled to run at specific intervals or triggered programmatically.
- Example:
  ```
  job emailSender {
    executor: PgBoss,
    perform: {
      fn: import { sendEmail } from "@src/server/jobs/emailSender.js"
    },
    entities: [User, Email]
  }
  ```

#### API Routes

- Custom API endpoints can be defined with the `api` declaration.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wasp-lang/vibe-coding-video](https://github.com/wasp-lang/vibe-coding-video) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-17 -->
