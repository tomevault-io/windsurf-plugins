---
trigger: always_on
description: description: Guidelines and best practices for building e-commerce website on EdgeOne Pages based on Shopify and Next.js, including edge functions, and real-world examples
---

---
description: 
globs: 
alwaysApply: true
---
---
description: 
globs: 
alwaysApply: true
---
---
description: Guidelines and best practices for building e-commerce website on EdgeOne Pages based on Shopify and Next.js, including edge functions, and real-world examples
globs: **/*.{ts,tsx,js,jsx,toml,json}
---

<ProviderContext version="1.0" provider="edgeone">
  ## General
  - the `.edgeone` folder is not for user code. It should be added to the .gitignore list
  - front end dev server and edge function dev server are two different servers.

  # Guidelines

  - There are 1 type of compute systems you can write code for:
    - Edge functions - usually used for code that must modify requests before hitting the server or modifying responses before returning to users.
  - Environment variables are available for storing secrets, API keys, and other values that you want to control external to the code or are too sensitive to put in the code.

  ### Technology Stack Description
  - Frontend framework: Next.js (must use SSG, i.e., Static Site Generation; SSR is not allowed)
  - Component library: shadcn/ui
  - Styles: It is recommended to use Tailwind CSS V4, document: https://tailwindcss.com/docs/installation/using-postcss
  - Type system: TypeScript
  - Edge functions: EdgeOne Functions, all API logic must be placed in the functions directory and follow EdgeOne specifications
  - E-commerce headless API: Use (Shopify Storefront API)[https://shopify.dev/docs/storefronts/headless/building-with-the-storefront-api/]

  ### Syntax and Project Specifications
  1. Next.js Configuration
     - Only SSG (getStaticProps, getStaticPaths) is allowed; SSR (getServerSideProps) is prohibited.
       - next.config should add   `output: "export"` to ensure the project is SSG.
     - All pages and components must use TypeScript (.tsx).
     - Routing follows Next.js file routing conventions.
     - Static assets are placed in the public directory.
     - Next.js min version 15.3.1.
     - All components that imports "useState" need has `"use client";` at the beginning of the file.
  2. shadcn/ui Component Library
     - Components must be imported from @/components/ui or @/components.
     - Direct modification of components inside node_modules is prohibited; extend via custom components if needed.
     - Component styles must use Tailwind CSS.
  3. Code Style
     - TypeScript must be used, with complete type declarations.
     - Variables, functions, and component names use camelCase; React components use PascalCase.
     - Strictly use ES6+ syntax, prohibit var, prefer const.
     - Use Prettier for code formatting, recommended to use with ESLint.
     - File and directory names use lowercase and kebab-case.
  4. EdgeOne Functions Specifications
     - How to write EdgeOne functions: see `EdgeOne Pages compute` below.
     - You should auto generate local environments for local EdgeOne Functions: NEXT_PUBLIC_DEV=true, NEXT_PUBLIC_API_URL_DEV=http://localhost:8088/ and FRONT_END_URL_DEV=`http://localhost:${FRONTEND DEV SERVER PORT}/`
     - Front end request EdgeOne functions API with `fetch`.
       - When env.DEV equals true, fetch url format is `${env.VITE_API_URL_DEV}/path-to-api`
       - When env.DEV equals true, redirect url in EdgeOne functions is `${env.FRONT_END_URL_DEV}/path-to-redirect`
     - Front end pages **can not** share same path with functions.
  5. Security and Environment
     - All sensitive information must be managed via environment variables; hardcoding is prohibited.
     - It is forbidden to expose any keys, tokens, or other sensitive information in frontend code.
  6. Language
     - Use English as default language when write code and comments.
  7. E-commerce Specifications
     - Use Shopify Storefront API as data source. 
       - Storefront API Document: https://shopify.dev/docs/storefronts/headless/building-with-the-storefront-api/
     - Website includes: index page, products page, signup/login/logout, cart, order management, checkout page.
       - Index page, contact page, blog page, products page are staticly generated. Sku stock number are dynamic from edge function.
       - Pages get other dynamic infomation from edge functions, edge functions call Storefront API.

  ## EdgeOne Pages compute

  - NEVER put any type of edge function in the public or publish directory
  - DO NOT change the default functions or edge functions directory unless explicitly asked to.
  - ALWAYS verify the correct directory to place functions or edge functions into

  ### Edge Functions
  - ALWAYS use the latest format of an edge function structure.

  - DO NOT put global logic outside of the exported function unless it is wrapped in a function definition
  - ONLY use vanilla javascript if there are other ".js" files in the functions directory.
  - ALWAYS use typescript if other functions are typescript or if there are no existing functions.
  - The first argument is a custom EdgeOne context object.
    - EdgeOne context object provides a "request" property, a web platform Request object that represents the incoming HTTP request

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [rohitrrnad-maker/enterprise-website-template](https://github.com/rohitrrnad-maker/enterprise-website-template) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-29 -->
