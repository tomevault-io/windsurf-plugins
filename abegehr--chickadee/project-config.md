---
trigger: always_on
description: meta framework
---

# HonoX

**HonoX** is a simple and fast meta-framework for creating full-stack websites or Web APIs - (formerly _[Sonik](mdc:https:/github.com/sonikjs/sonik)_). It stands on the shoulders of giants; built on [Hono](mdc:https:/hono.dev), [Vite](mdc:https:/vitejs.dev), and UI libraries.

**Note**: _HonoX is currently in the "alpha stage". Breaking changes are introduced without following semantic versioning._

## Features

- **File-based routing** - You can create a large application like Next.js.
- **Fast SSR** - Rendering is ultra-fast thanks to Hono.
- **BYOR** - You can bring your own renderer, not only one using hono/jsx.
- **Islands hydration** - If you want interactions, create an island. JavaScript is hydrated only for it.
- **Middleware** - It works as Hono, so you can use a lot of Hono's middleware.

## Installing

You can install the `honox` package from the npm.

```txt
npm install hono honox
```

## Starter template

If you are starting a new HonoX project, use the `hono-create` command. Run the following and choose `x-basic` (use the arrow keys to find the option).

```txt
npm create hono@latest
```

## Get Started - Basic

Let's create a basic HonoX application using hono/jsx as a renderer. This application has no client JavaScript and renders JSX on the server side.

### Project Structure

Below is a typical project structure for a HonoX application.

```txt
.
├── app
│   ├── global.d.ts // global type definitions
│   ├── routes
│   │   ├── _404.tsx // not found page
│   │   ├── _error.tsx // error page
│   │   ├── _renderer.tsx // renderer definition
│   │   ├── merch
│   │   │   └── [...slug].tsx // matches `/merch/:category`, `/merch/:category/:item`, `/merch/:category/:item/:variant`
│   │   ├── about
│   │   │   └── [name].tsx // matches `/about/:name`
│   │   ├── blog
│   │   │   ├── index.tsx // matches /blog
│   │   │   └── (content)
│   │   │       ├── _renderer.tsx // renderer definition for routes inside this directory
│   │   │       └── [name].tsx    // matches `/blog/:name`
│   │   └── index.tsx // matches `/`
│   └── server.ts // server entry file
├── package.json
├── tsconfig.json
└── vite.config.ts
```

### `vite.config.ts`

The minimum Vite setup for development is as follows:

```ts
import { defineConfig } from 'vite'
import honox from 'honox/vite'

export default defineConfig({
  plugins: [honox()],
})
```

### Server Entry File

A server entry file is required. The file should be placed at `app/server.ts`. This file is first called by the Vite during the development or build phase.

In the entry file, simply initialize your app using the `createApp()` function. `app` will be an instance of Hono, so you can use Hono's middleware and the `showRoutes()` in `hono/dev`.

```ts
// app/server.ts
import { createApp } from 'honox/server'
import { showRoutes } from 'hono/dev'

const app = createApp()

showRoutes(app)

export default app
```

### Routes

There are three ways to define routes.

#### 1. `createRoute()`

Each route should return an array of `Handler | MiddlewareHandler`. `createRoute()` is a helper function to return it. You can write a route for a GET request with `default export`.

```tsx
// app/routes/index.tsx
// `createRoute()` helps you create handlers
import { createRoute } from 'honox/factory'

export default createRoute((c) => {
  return c.render(
    <div>
      <h1>Hello!</h1>
    </div>
  )
})
```

You can also handle methods other than GET by `export` `POST`, `PUT`, and `DELETE`.

```tsx
// app/routes/index.tsx
import { createRoute } from 'honox/factory'
import { getCookie, setCookie } from 'hono/cookie'

export const POST = createRoute(async (c) => {
  const { name } = await c.req.parseBody<{ name: string }>()
  setCookie(c, 'name', name)
  return c.redirect('/')
})

export default createRoute((c) => {
  const name = getCookie(c, 'name') ?? 'no name'
  return c.render(
    <div>
      <h1>Hello, {name}!</h1>
      <form method='POST'>
        <input type='text' name='name' placeholder='name' />
        <input type='submit' />
      </form>
    </div>
  )
})
```

#### 2. Using a Hono instance

You can create API endpoints by exporting an instance of the Hono object.

```ts
// app/routes/about/index.ts
import { Hono } from 'hono'

const app = new Hono()

// matches `/about/:name`
app.get('/:name', (c) => {
  const name = c.req.param('name')
  return c.json({
    'your name is': name,
  })
})

export default app
```

#### 3. Just return JSX

Or simply, you can just return JSX.

```tsx
// app/routes/index.tsx
export default function Home(_c: Context) {
  return <h1>Welcome!</h1>
}
```

### Renderer

Define your renderer - the middleware that does `c.setRender()` - by writing it in `_renderer.tsx`.

Before writing `_renderer.tsx`, write the Renderer type definition in `global.d.ts`.

```ts
// app/global.d.ts
import type {} from 'hono'

type Head = {
  title?: string
}

declare module 'hono' {
  interface ContextRenderer {
    (content: string | Promise<string>, head?: Head): Response | Promise<Response>
  }
}
```

The JSX Renderer middleware allows you to create a Renderer as follows:

```tsx
// app/routes/_renderer.tsx
import { jsxRenderer } from 'hono/jsx-renderer'

export default jsxRenderer(({ children, title }) => {
  return (
    <html lang='en'>
      <head>
        <meta charset='UTF-8' />

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [abegehr/chickadee](https://github.com/abegehr/chickadee) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
