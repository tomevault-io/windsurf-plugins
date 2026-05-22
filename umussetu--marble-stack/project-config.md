---
trigger: always_on
description: This is a Vite Remix project in React using Typescript, Ant Design and Tailwind and tRPC React query.
---

# Frontend Documentation

This is a Vite Remix project in React using Typescript, Ant Design and Tailwind and tRPC React query.

## Pages

Pages located at "/app/routes/\_logged.**.tsx" are accessible for logged users.
Other pages located at "/app/routes/**.tsx" are public.

# Authorization Policies on Server

Policies are defined using zenstack on zmodels "/models/models.zmodel".
It automatically protect auto-generated CRUD models and the default Prisma client.

You can use the `@@allow` decorator to define policies.

Example for a model "Foo" at "/models/models.zmodel":

```zmodel
model Foo {
  id               String    @id @default(uuid())
  name             String?
  email            String?
  isVerified       Boolean?

  @@allow('all', auth().roles?[name == 'admin'])
  @@allow('read', true)
  @@allow('create', auth().id == this.id)
  @@allow('update', auth().id == this.id)
  @@allow('delete', auth().id == this.id && this.isVerified)
}
```

Explanations:

- `auth()` is the authenticated user, you have access to the same properties as the user model.
- `this` is the current model (here Foo)
- `@@allow('read', [condition])` grant read policy if the condition is true. It will also affect how findMany results are filtered.
- `@@allow('update', [condition])` grant update policy if the condition is true.
- `@@allow('delete', [condition])` grant delete policy if the condition is true.
- `@@allow('all', [condition])` grant all policies if the condition is true
- `auth().roles?[name == 'admin']` - you can access relations. In this example, we checking if a user, has a role with the name 'admin'

# Core Utilities

Core utilities, hooks and business logic are located at "/app/core"

- "app/core/context" is where the user context store is setup. It can be used in the application to access information about the logged user.
- "app/core/helpers" is where small utilities to manipulate dates or typescript operations are stored.
- "app/core/hooks" contains hooks for external libraries and API routes that are not using tRPC.

## Sensitive files

- "app/core/authentication" is where the authentication is setup.
- "app/core/configuration" is where the common environment variables are provided.
- "app/core/database" is where the Prisma database is setup for the backend.
- "app/core/trpc" is where the trpc client and server are setup.

Do not updates these files except if asked to do so.

# Design System

The project uses Ant Design for all UI components. On top of that, you can use tailwind className to style and position the UI components.

The design system is setup in the "/app/designSystem" folder.

The following folders and files can safely be updated:

- "/app/designSystem/layouts" contains re-usable layouts components for pages.
- "/app/designSystem/theme/theme.tsx" is where the Ant Design theme can be customised.
- "/app/designSystem/ui" is where custom components that are not part of Ant Design but useful for the whole application are stored.

## Sensitive files

Except if explicitly asked, you must avoid touching the following folders and files:

- "/app/designSystem/core" contains top level components such as HTML and main tags.
- "/app/designSystem/provider.tsx" is where we apply all design system providers.
- "/app/designSystem/style" contains global scss files.

[CHUNK-SEPARATOR]

## How to create a UI component

1. Create your component in "/app/designSystem/ui/[ComponentName]/index.tsx"
2. Code the component

```tsx
// Typical example

type Props = {
  key1: string
  key2: string
  key3: string
}

export const ComponentName: React.FC<Props> = ({ key1, key2, key3 }) => {
  return <>...</>
}
```

Tips:

- We usually prefer to export the `const` directly so the component has a consistent import in other files `import { ComponentName } from '@/designSystem/ui/ComponentName'`.
- You can use tailwind className to style the UI components.
- To import Title, Text and Paragraph from Ant Design, you need to import \`import { Typography } from 'antd'\` and then do \`const { Title, Text, Paragraph} = Typography\`. There are no other ways.

## Session

The context available from `import { AuthenticationServer } from '@/core/authentication/server'` is an object with the following structure:

```ts
type Context = {
  database: PrismaClient // The default prisma client to use, recommended as there are policies triggered in case the action is not permitted for the user.
  databaseUnprotected: PrismaClient // The prisma client without policies, useful when you don't need the policies.
  session: {
    user?: {
      id: string
      email: string
      name: string
      globalRole: string
    }
  }
}

const context = await AuthenticationServer.getHttpContext({ request })

console.log(context.session.user)
```

## Database Interaction

```ts
const context = await AuthenticationServer.getHttpContext({ request })

const userFetched = await context.database.user.findUnique({
  data: { id: context.session.user.id },
})

const filteredUsers = await context.database.user.findMany() // This automatically apply authorization policies to filter the results

const allUsers = await context.databaseUnprotected.user.findMany() // This bypass all the authorization policies
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [umussetu/marble-stack](https://github.com/umussetu/marble-stack) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
