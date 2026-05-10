---
trigger: always_on
description: This repository is a Cloudflare Worker that provides a REST API layer for generative AI inference and completion. It serves as a schema-driven HTTP API gateway, integrating with OpenAI and Google GenAI backends to provide a unified interface for AI model inference. The worker manages completion requests, supports multimodal input (text and images), and handles provider abstraction for multiple AI services. Built with Hono framework, it provides type-safe endpoints with comprehensive validation a
---

# Worker API Agent Instructions

## Project Overview

This repository is a Cloudflare Worker that provides a REST API layer for generative AI inference and completion. It serves as a schema-driven HTTP API gateway, integrating with OpenAI and Google GenAI backends to provide a unified interface for AI model inference. The worker manages completion requests, supports multimodal input (text and images), and handles provider abstraction for multiple AI services. Built with Hono framework, it provides type-safe endpoints with comprehensive validation and error handling.

## Tech Stack

- **Language:** TypeScript (strict mode, ESNext)
- **Framework:** Hono (for Cloudflare Workers)
- **Authentication:** Bearer token authentication
- **Validation:** Zod schemas for request/response validation
- **Middleware:** CORS, compression, body limits, secure headers, pretty JSON
- **AI Providers:** OpenAI, Google GenAI (via @google/genai and openai packages)
- **Runtime:** Cloudflare Workers
- **Formatting/Linting:** Biome (spaces, double quotes, recommended rules)
- **Build Tools:** tsx, Wrangler
- **Automation:** Makefile, pnpm scripts
- **Environment:** dotenv (.dev.vars, .prod.vars)
- **Package Manager:** pnpm

## Project Structure

```
.
├── src/
│   ├── clients/              # Client implementations for AI providers
│   │   ├── baseClient.ts     # Base client interface
│   │   └── googleGenAIClient.ts
│   ├── dtos/                 # Data transfer objects and Zod schemas
│   │   ├── request.ts        # Request validation schemas
│   │   ├── response.ts       # Response validation schemas
│   │   └── internal.ts       # Internal DTOs
│   ├── routes/               # Route handlers
│   │   ├── health.ts         # Health check endpoint
│   │   └── completion.ts     # AI completion endpoint
│   ├── services/             # Business logic services
│   │   ├── inferenceService.ts  # Main inference orchestration
│   │   └── providers/        # AI provider implementations
│   │       ├── baseInferenceProvider.ts  # Abstract base class
│   │       └── googleGenAIProvider.ts    # Google GenAI implementation
│   ├── middlewares/          # Custom middleware
│   │   └── apiKeyProvider.ts  # API key resolution middleware
│   ├── enums/                # Type-safe enumerations
│   │   ├── provider.ts       # AI provider types
│   │   ├── model.ts          # Supported model names
│   │   ├── role.ts           # Message role types
│   │   └── ...
│   ├── types/                # TypeScript type definitions
│   │   ├── inference.ts      # Inference-related types
│   │   └── completionContext.ts
│   ├── utils/                # Utility functions
│   │   ├── retry.ts          # Retry logic for API calls
│   │   └── imageUtils.ts     # Image validation utilities
│   └── index.ts              # Main application entry point
├── biome.json                # Biome formatting/linting config
├── tsconfig.json             # TypeScript config (strict, path aliases)
├── wrangler.jsonc            # Cloudflare Workers configuration
├── Makefile                  # CLI shortcuts for common tasks
├── package.json              # Scripts, dependencies
├── .dev.vars/.prod.vars      # Environment variables
└── README.md                 # Usage and deployment instructions
```

## Environment Configuration

- **Development:** Runs on port 8788 (configurable in `wrangler.jsonc`)
- **Production:** Deployed as Cloudflare Worker

### Setup Instructions

1. **Install dependencies:**

   `make init`

2. **Configure environment:**

   Create `.dev.vars` file with required variables:

   ```
   BEARER_TOKEN=your_bearer_token_for_api_security
   GOOGLE_AI_STUDIO_API_KEY=your_google_ai_studio_api_key
   CLOUDFLARE_ACCOUNT_ID=your_cloudflare_account_id
   CLOUDFLARE_AI_GATEWAY_ID=your_gateway_id
   CLOUDFLARE_AI_GATEWAY_BASE_URL=https://gateway.ai.cloudflare.com/v1
   ```

   For production, set secrets via Wrangler:

   ```bash
   wrangler secret put BEARER_TOKEN
   wrangler secret put GOOGLE_AI_STUDIO_API_KEY
   ```

3. **Development:**

   `make dev` - Runs development server on port 8788

4. **Deploy:**

   `make deploy` - Deploys to Cloudflare Workers

## Common Commands

The following Makefile commands are available for development, formatting, testing, and deployment:

| Command                | Description                                 |
|------------------------|---------------------------------------------|
| `make init`            | Initialize the project (install dependencies) |
| `make update`          | Update dependencies to latest versions      |
| `make dev`             | Run development server with hot reloading   |
| `make format`          | Format the codebase using Biome             |
| `make lint`            | Lint the codebase using Biome               |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [louisbrulenaudet/genai-api](https://github.com/louisbrulenaudet/genai-api) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
