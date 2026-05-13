---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository Overview

This repository contains the source code of ModelFetch: a TypeScript/JavaScript SDK for building MCP (Model Context Protocol) servers with a delightful developer experience and deploying them anywhere TypeScript/JavaScript runs.

The SDK is built as a thin wrapper on top of `hono` and `@hono/mcp`, leveraging Hono's runtime-agnostic capabilities. The SDK provides runtime-specific packages to handle all runtime-specific details:

- `@modelfetch/node` - Node.js runtime support
- `@modelfetch/next` - Next.js runtime support
- `@modelfetch/bun` - Bun runtime support
- `@modelfetch/deno` - Deno runtime support
- `@modelfetch/vercel` - Vercel runtime support
- `@modelfetch/cloudflare` - Cloudflare runtime support
- `@modelfetch/netlify` - Netlify runtime support
- `@modelfetch/fastly` - Fastly runtime support
- `@modelfetch/supabase` - Supabase runtime support
- `@modelfetch/gcore` - Gcore runtime support
- `@modelfetch/aws-lambda` - AWS Lambda runtime support
- `@modelfetch/azure-functions` - Azure Functions runtime support

Each package acts as a thin wrapper around the equivalent Hono package, ensuring optimal performance and compatibility. Additionally, each package maintains a consistent API across different runtimes.

The `modelfetch` CLI provides an exceptional developer experience with:

- `dev` command for running MCP servers in development mode with hot reload
- Integration with the MCP inspector for a seamless testing and debugging experience

**Goal**: To become the world's most popular TypeScript/JavaScript SDK for building MCP servers.

## Workspace Overview

This workspace follows Nx best practices as outlined in @.cursor/rules/nx-rules.mdc

### Workspace Configurations

- Node.js version: @.nvmrc
- TypeScript configurations: @tsconfig.base.json @tsconfig.node.json @tsconfig.next.json @tsconfig.bun.json @tsconfig.deno.json
- TypeScript project references are used to improve the performance of TypeScript-related features
  - All TypeScript projects are referenced in the root @tsconfig.json
  - When adding or removing a TypeScript project as a local dependency to another TypeScript project, update the target project's `tsconfig.json` to add or remove the dependency project in its `references`
- Only TypeScript projects have ESLint configurations
- All JavaScript projects have no ESLint configurations
- All Nx projects use automatic configuration provided by Nx plugins instead of individual `project.json` files
- All local dependencies use `workspace:^` version specifier
- `dependencies` and `devDependencies` that are already available in the root @package.json should NOT be included in individual project package.json files. This includes common dependencies like `typescript`, `@types/node`, `eslint`, `prettier`, `nx`, `@nx/*`, `@swc/*`, etc.

### Core Applications

- `modelfetch-website`: ModelFetch website built with Next.js v15, Tailwind CSS v4, Fumadocs v15, and Fumadocs MDX

### Example Applications

These projects are example applications powered by ModelFetch:

- `example-node-js`: Node.js application (JavaScript)
- `example-node-ts`: Node.js application (TypeScript)
- `example-next-js`: Next.js application (JavaScript)
- `example-next-ts`: Next.js application (TypeScript)
- `example-bun-js`: Bun application (JavaScript)
- `example-bun-ts`: Bun application (TypeScript)
- `example-deno-js`: Deno application (JavaScript)
- `example-deno-ts`: Deno application (TypeScript)
- `example-vercel-js`: Vercel application (JavaScript)
- `example-vercel-ts`: Vercel application (TypeScript)
- `example-cloudflare-js`: Cloudflare application (JavaScript)
- `example-cloudflare-ts`: Cloudflare application (TypeScript)
- `example-netlify-js`: Netlify application (JavaScript)
- `example-netlify-ts`: Netlify application (TypeScript)
- `example-fastly-js`: Fastly application (JavaScript)
- `example-fastly-ts`: Fastly application (TypeScript)
- `example-supabase-js`: Supabase application (JavaScript)
- `example-supabase-ts`: Supabase application (TypeScript)
- `example-gcore-js`: Gcore application (JavaScript)
- `example-gcore-ts`: Gcore application (TypeScript)
- `example-aws-lambda-js`: AWS Lambda application (JavaScript)
- `example-aws-lambda-ts`: AWS Lambda application (TypeScript)
- `example-azure-functions-js`: Azure Functions application (JavaScript)
- `example-azure-functions-ts`: Azure Functions application (TypeScript)

### Core Libraries

- `modelfetch`: ModelFetch CLI tools (work in progress)
- `@modelfetch/core`: ModelFetch core utilities
- `@modelfetch/node`: Node.js runtime support
- `@modelfetch/next`: Next.js runtime support
- `@modelfetch/bun`: Bun runtime support
- `@modelfetch/deno`: Deno runtime support
- `@modelfetch/vercel`: Vercel runtime support
- `@modelfetch/cloudflare`: Cloudflare runtime support
- `@modelfetch/netlify`: Netlify runtime support
- `@modelfetch/fastly`: Fastly runtime support
- `@modelfetch/supabase`: Supabase runtime support
- `@modelfetch/gcore`: Gcore runtime support
- `@modelfetch/aws-lambda`: AWS Lambda runtime support
- `@modelfetch/azure-functions`: Azure Functions runtime support

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [phuctm97/modelfetch](https://github.com/phuctm97/modelfetch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
