---
trigger: always_on
description: These guidelines define the operational principles and capabilities of an AI agent (e.g., Gemini) interacting with Next.js projects within the Firebase Studio environment. The goal is to enable an efficient, automated, and error-resilient application design and development workflow that leverages the full power of the Next.js framework.
---

# **AI Development Guidelines for Next.js in Firebase Studio**

These guidelines define the operational principles and capabilities of an AI agent (e.g., Gemini) interacting with Next.js projects within the Firebase Studio environment. The goal is to enable an efficient, automated, and error-resilient application design and development workflow that leverages the full power of the Next.js framework.

## **Environment & Context Awareness**

The AI operates within the Firebase Studio development environment, which provides a Code OSS-based IDE and a pre-configured environment for Next.js development.

* **Project Structure (App Router):** The AI assumes a standard Next.js project structure using the App Router.
  * `/app`: The core directory for file-based routing.
  * `layout.tsx`: The root layout.
  * `page.tsx`: The page UI for a route.
  * `/components`: For reusable UI components.
  * `/lib`: For utility functions and libraries.
* **`dev.nix` Configuration:** The AI is aware of the `.idx/dev.nix` file for environment configuration, which includes `pkgs.nodejs` and other necessary tools.
* **Preview Server:** Firebase Studio provides a running preview server. The AI **will not** run `next dev`, but will instead monitor the output of the already running server for real-time feedback.
* **Firebase Integration:** The AI can integrate Firebase services, following standard procedures for Next.js projects, including using the Firebase Admin SDK in server-side code.

## Firebase MCP

When requested for Firebase add the following the server configurations to .idx/mcp.json. Just add the following and don't add anything else.

{
    "mcpServers": {
        "firebase": {
            "command": "npx",
            "args": [
                "-y",
                "firebase-tools@latest",
                "experimental:mcp"
            ]
        }
    }
}

## **Code Modification & Dependency Management**

The AI is empowered to modify the codebase autonomously based on user requests. The AI is creative and anticipates features that the user might need even if not explicitly requested.

* **Core Code Assumption:** The AI will primarily work with React components (`.tsx` or `.jsx`) within the `/app` directory. It will create new routes, layouts, and components as needed.
* **Package Management:** The AI will use `npm` or `yarn` for package management.
* **Next.js CLI:** The AI will use the Next.js CLI for common development tasks:
  * `npm run build`: To build the project for production.
  * `npm run lint`: To run ESLint and check for code quality issues.

## **Next.js Core Concepts (App Router)**

### **Server Components by Default**

The AI understands that components in the `/app` directory are React Server Components (RSCs) by default.

* **Data Fetching:** The AI will perform data fetching directly in Server Components using `async/await`, colocating data access with the component that uses it.
* **"use client" Directive:** For components that require interactivity, state, or browser-only APIs, the AI will use the `"use client"` directive to mark them as Client Components.
* **Best Practice:** Keep Client Components as small as possible and push them to the leaves of the component tree to minimize the client-side JavaScript bundle.

### **File-based Routing**

The AI will manage routing by creating folders and `page.tsx` files within the `/app` directory.

* **Layouts (`layout.tsx`):** Define shared UI for a segment and its children.
* **Pages (`page.tsx`):** Define the unique UI of a route.
* **Loading UI (`loading.tsx`):** Create instant loading states that show while a route segment loads.
* **Error Handling (`error.tsx`):** Isolate errors to specific route segments.

### **Server Actions**

For data mutations (e.g., form submissions), the AI will use Server Actions to call server-side functions directly from components.

* **Definition:** The AI will define an `async` function with the `"use server"` directive.
* **Invocation:** Actions will be invoked using the `action` prop on a `<form>` element or from custom event handlers.
* **Security:** Server Actions are the preferred way to handle mutations as they provide built-in protection against POST-only requests.

*Example of a simple Server Action:*

```ts
// app/actions.ts
'use server'

import { z } from 'zod'

const schema = z.object({
  email: z.string().email(),
})

export async function-save-email(prevState: any, formData: FormData) {
  const validatedFields = schema.safeParse({
    email: formData.get('email'),
  })

  if (!validatedFields.success) {
    return {
      errors: validatedFields.error.flatten().fieldErrors,
    }
  }
  // Save email to database...
  return { message: 'Success!' }
}
```

## **Automated Error Detection & Remediation**

A critical function of the AI is to continuously monitor for and automatically resolve errors.

* **Post-Modification Checks:** After every code modification, the AI will:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [chimalongy/ai-image-generator](https://github.com/chimalongy/ai-image-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
