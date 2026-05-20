---
trigger: always_on
description: Writing Svelte code with Go backend and adapter-static using Tailwind CSS, DaisyUI, and Soul AST-generated API endpoints
---


# SvelteKit with Go Backend and Adapter-Static

In this project, we use **Tailwind CSS** for utility-first styling and **DaisyUI** as a component library built on top of Tailwind CSS, providing a lightweight and customizable UI framework. The frontend is built with SvelteKit and paired with a Go backend using the **Soul AST system**, which generates API endpoints found in `./src/lib/api`. The `adapter-static` package is used to generate static files, served by the Go backend alongside the dynamic API routes.

## Go Backend Integration with Soul AST

The Go backend is powered by the **Soul AST system**, which generates API endpoints automatically. These endpoints are exposed via TypeScript clients in `./src/lib/api`, including functions like `RegisterPost`, `LoginPost`, and others, as defined in `endpoints.ts`. The SvelteKit frontend is configured with `adapter-static` to produce static files, which the Go server delivers alongside the Soul-generated API.

### Setup Instructions

1. **Install dependencies using pnpm**:
   Initialize your SvelteKit project and install required packages:
   ```bash
   pnpm init
   pnpm create svelte@latest my-svelte-go-app
   cd my-svelte-go-app
   pnpm install
   pnpm add -D @sveltejs/adapter-static tailwindcss postcss autoprefixer daisyui
   ```

2. **Set up Tailwind CSS and DaisyUI**:
   - Generate Tailwind configuration:
     ```bash
     pnpm dlx tailwindcss init -p
     ```
   - Update `tailwind.config.js`:
     ```javascript
     /** @type {import('tailwindcss').Config} */
     export default {
       content: ['./src/**/*.{html,js,svelte,ts}'],
       theme: {
         extend: {},
       },
       plugins: [require('daisyui')],
       daisyui: {
         themes: ["light", "dark", "cupcake"], // Customize as needed
       },
     };
     ```
   - Create `src/app.css`:
     ```css
     @tailwind base;
     @tailwind components;
     @tailwind utilities;
     ```
   - Import it in `src/routes/+layout.svelte`:
     ```svelte
     <script>
       import '../app.css';
     </script>
     <slot />
     ```

3. **Configure adapter-static in `svelte.config.js`**:
   ```javascript
   import adapter from '@sveltejs/adapter-static';
   import { vitePreprocess } from '@sveltejs/vite-plugin-svelte';

   /** @type {import('@sveltejs/kit').Config} */
   const config = {
     preprocess: vitePreprocess(),
     kit: {
       adapter: adapter({
         fallback: 'index.html', // Fallback for SPA routing
         pages: 'build',         // Output directory
         assets: 'build',        // Output directory
       }),
       paths: {
         base: '', // Adjust if deploying to a subdirectory
       },
     },
   };

   export default config;
   ```

4. **Configure the Go backend**:
   - The Soul AST system has already generated the backend handlers in `internal/handler/`. Use them in `main.go`:
     ```go
     package main

     import (
       "net/http"
       "github.com/gin-gonic/gin"
       "your-app/internal/handler" // Adjust to your module path
     )

     func main() {
       router := gin.Default()

       // Serve static files from the SvelteKit build directory
       router.Static("/", "./build")

       // Register Soul-generated API handlers
       handler.RegisterHandlers(router)

       // SPA fallback for client-side routing
       router.NoRoute(func(c *gin.Context) {
         c.File("./build/index.html")
       })

       router.Run(":8080")
     }
     ```
   - Ensure `ast/` files (e.g., `main.api`, `api/handlers.api`) are defined and `make gen` has been run to generate the API.

5. **Build and Run**:
   - Build the SvelteKit frontend:
     ```bash
     pnpm build
     ```
   - Run the Go backend:
     ```bash
     go run main.go
     ```
   - Access the app at `http://localhost:8080`.

## Accessing Soul-Generated API Endpoints

The Soul AST system generates TypeScript API clients in `./src/lib/api`. Import and use these endpoints directly in your Svelte components. Below are examples using the provided endpoints:

### Example: User Authentication

```svelte
/// file: src/routes/login/+page.svelte
<script lang="ts">
  import { api } from '@api'; // Soul-generated API endpoints
  let email = $state('');
  let password = $state('');
  let error = $state<string | null>(null);
  let success = $state(false);

  async function handleLogin() {
    try {
      const response = await api.LoginPost({ email, password });
      if (response.success) {
        success = true;
      }
    } catch (e) {
      error = e.message;
    }
  }
</script>

<div class="container p-4 mx-auto">
  <h1 class="mb-4 text-2xl font-bold">Login</h1>
  {#if success}
    <p class="text-success">Logged in successfully!</p>
  {:else}
    <form onsubmit|preventDefault={handleLogin} class="space-y-4">
      <input type="email" bind:value={email} placeholder="Email" class="w-full input input-bordered" />
      <input type="password" bind:value={password} placeholder="Password" class="w-full input input-bordered" />
      {#if error}
        <p class="text-error">{error}</p>
      {/if}
      <button type="submit" class="btn btn-primary">Login</button>
    </form>
  {/if}
</div>
```

### Example: Fetching User Profile

```svelte

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [localrivet/gomcp](https://github.com/localrivet/gomcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
