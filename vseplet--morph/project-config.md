---
trigger: always_on
description: This document provides structured information for LLM agents to effectively
---

# Morph — Documentation for AI Agents

This document provides structured information for LLM agents to effectively
build web applications using Morph.

## Overview

Morph is a **server-side rendering library** for building web UIs with HTMX and
Hono. Key characteristics:

- **Runtime:** Deno, Bun, or Node.js
- **No build step:** TypeScript runs directly
- **Server-rendered:** All components execute on the server
- **HTMX-powered:** Partial page updates without client JavaScript

## Project Setup

### File Structure (Minimal)

```
project/
├── deno.json        # or package.json for Node/Bun
└── main.ts          # entry point
```

### File Structure (Recommended)

```
project/
├── deno.json
├── main.ts
├── components/
│   ├── layout.ts    # wrapper, navigation
│   ├── pages/       # page components
│   └── partials/    # HTMX-updatable components
└── tests/
    └── *.test.ts
```

### Dependencies

```json
// deno.json
{
  "imports": {
    "@hono/hono": "jsr:@hono/hono@4",
    "@vseplet/morph": "jsr:@vseplet/morph"
  }
}
```

## Core Concepts

### 1. Components

Components are functions that return HTML templates.

```ts
import { component, html } from "@vseplet/morph";

// Basic component (no props)
const header = component(() =>
  html`
    <header>
      <h1>My App</h1>
    </header>
  `
);

// Component with typed props
const userCard = component<{ name: string; email: string }>((props) =>
  html`
    <div class="user-card">
      <h3>${props.name}</h3>
      <p>${props.email}</p>
    </div>
  `
);

// Async component (can fetch data)
const userList = component(async (props) => {
  const users = await fetchUsers();
  return html`
    <ul>
      ${users.map((u) => userCard({ name: u.name, email: u.email }))}
    </ul>
  `;
});
```

### 2. Available Props in Components

Every component receives `MorphPageProps`:

```ts
interface MorphPageProps {
  request: Request; // Raw HTTP request
  route: string; // Current route path
  params: Record<string, string>; // URL params (:id -> params.id)
  query: Record<string, string>; // Query string (?foo=bar -> query.foo)
  headers: Record<string, string>; // Request headers
  hx: () => string; // Returns hx-get attribute for self-refresh
}
```

Example usage:

```ts
const page = component((props) =>
  html`
    <div>
      <p>URL: ${props.request.url}</p>
      <p>User ID: ${props.params.id}</p>
      <p>Search: ${props.query.q ?? "none"}</p>
      <p>Auth: ${props.headers.authorization ?? "none"}</p>
    </div>
  `
);
```

### 3. Templates and Interpolation

```ts
// Strings and numbers
html`
  <p>Count: ${42}</p>
`; // -> <p>Count: 42</p>

// Nested templates
html`
  <div>${html`
    <span>nested</span>
  `}</div>
`;

// Arrays (auto-joined)
html`
  <ul>${items.map((i) =>
    html`
      <li>${i}</li>
    `
  )}</ul>
`;

// Conditionals
html`
  <div>${isAdmin
    ? html`
      <button>Delete</button>
    `
    : ""}</div>
`;

// Components
html`
  <div>${userCard({ name: "Alice", email: "a@b.com" })}</div>
`;

// Falsy values: null, undefined, false render as empty string
// IMPORTANT: 0 renders as "0" (not empty)
html`
  <p>${0}</p>
`; // -> <p>0</p>
html`
  <p>${null}</p>
`; // -> <p></p>
```

### 4. Styling with `styled`

```ts
import { styled } from "@vseplet/morph";

// Creates unique class name, CSS collected in <head>
const buttonClass = styled`
  padding: 8px 16px;
  background: blue;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;

  &:hover {
    background: darkblue;
  }
`;

const button = component<{ label: string }>((props) =>
  html`
    <button class="${buttonClass}">${props.label}</button>
  `
);
```

### 5. Meta (Title, Headers, Status)

```ts
import { meta } from "@vseplet/morph";

const page = component(() =>
  html`
    ${meta({
      title: "Page Title", // <title> tag
      statusCode: 200, // HTTP status
      statusText: "OK", // HTTP status text
      headers: { // Response headers
        "X-Custom": "value",
        "Cache-Control": "no-cache",
      },
      head: `<link rel="icon" href="/favicon.ico">`, // Inject into <head>
      bodyStart: `<div id="top"></div>`, // Start of <body>
      bodyEnd: `<script src="/analytics.js"></script>`, // End of <body>
    })}
    <h1>Content</h1>
  `
);
```

### 6. Client-Side JavaScript

```ts
import { fn, js, onclick, script } from "@vseplet/morph";

const page = component(() =>
  html`
    <div>
      <!-- Inline JS block (added to end of body) -->
      ${js`console.log("Page loaded");`}

      <!-- Function converted to script -->
      ${fn(() => {
        document.querySelector("#btn").addEventListener("click", () => {
          alert("Clicked!");
        });
      })}

      <!-- Inline onclick attribute -->
      <button ${onclick(() => alert("Hello"))}>Click me</button>

      <!-- Script tag in HTML -->
      ${script(() => console.log("Inline script"))}
    </div>
  `
);
```

## Application Setup

### Minimal App

```ts
import { Hono } from "@hono/hono";
import { component, html, morph } from "@vseplet/morph";

const homePage = component(() =>
  html`
    <h1>Hello!</h1>
  `
);

const app = new Hono().all(
  "/*",
  (c) => morph.page("/", homePage).fetch(c.req.raw),
);

Deno.serve(app.fetch);
```

### App with Layout

```ts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vseplet/morph](https://github.com/vseplet/morph) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-21 -->
