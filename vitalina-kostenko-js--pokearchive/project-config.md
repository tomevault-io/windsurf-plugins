---
trigger: always_on
description: * **Layers** - standardized levels of abstraction
---


# Server Architecture Guide

## Overview

* **Layers** - standardized levels of abstraction
* **Slices** - feature-based divisions within layers
* **Segments** - technical purpose divisions within slices

```
┌─────────────┬────────────┬────────────┐
│   Layers    │   Slices   │  Segments  │
├─────────────┼────────────┼────────────┤
│   routes    │      -     │      +     │
│   modules   │      +     │      +     │
│   widget    │      +     │      +     │
│   features  │      +     │      +     │
│   entities  │      +     │      +     │
│   shared    │      -     │      +     │
└─────────────┴────────────┴────────────┘
```


## Complete Project Structure

```
src/
├── server.ts                                       # Main entry point - Fastify app initialization
├── payload.config.ts                               # Payload CMS configuration
├── payload-types.ts                                # Payload CMS generated types
├── app/
│   ├── (payload)/                                  # Layer - Payload CMS admin panel
│   │   ├── [[...segments]]/                        # Catch-all route for Payload admin
│   │   ├── api/                                    # Payload API routes
│   │   ├── migrations/                             # Database migrations
│   │   ├── seed/                                   # Database seed data
│   │   ├── layout.tsx                              # Admin layout
│   │   ├── custom.scss                             # Admin custom styles
│   │   └── importMap.js                            # Payload import map
│   ├── routes/                                     # Layer - API routing
│   │   ├── index.ts                                # Main router export
│   │   └── server.routes.ts                        # Route definitions and module registration
│   ├── modules/                                    # Layer - Main business logic
│   │   ├── module-name/                            # Slice
│   │   │   ├── module-name.module.ts               # Route registration and validation
│   │   │   ├── module-name.service.ts              # Business logic
│   │   │   └── index.ts
│   │   └── ...
│   ├── widget/                                     # Layer - Complex reusable functionality
│   │   └── widget-name/                            # Slice
│   │       ├── widget-name.service.ts              # Widget logic
│   │       ├── widget-name.interface.ts            # Type definitions (Optional)
│   │       ├── widget-name.constant.ts             # Constants (Optional)
│   │       └── index.ts
│   ├── features/                                   # Layer - Reusable implementations
│   │   ├── feature-name/                           # Slice
│   │   │   ├── feature-name.service.ts             # Feature logic
│   │   │   ├── feature-name.interface.ts           # Type definitions (Optional)
│   │   │   ├── feature-name.constant.ts            # Constants (Optional)
│   │   │   └── index.ts
│   │   └── ...
│   ├── entities/                                   # Layer - Business entities
│   │   ├── collections/                            # Slice - Payload CMS collections
│   │   │   ├── entity-name.collection.ts           # Collection definition
│   │   │   ├── ...
│   │   │   └── index.ts
│   │   ├── dto/                                    # Slice - Data transfer objects
│   │   │   ├── entity-name.dto.ts                  # Zod validation schemas
│   │   │   ├── ...
│   │   │   └── index.ts
│   │   └── index.ts
│   └── shared/                                     # Layer - Reusable code
│       ├── components/                             # Segment - Shared components
│       ├── constant/                               # Segment - Global constants
│       │   ├── constant-name.constant.ts
│       │   └── index.ts
│       ├── decorators/                             # Segment - Payload CMS decorators/hooks
│       │   └── hook-name.hook.ts
│       ├── hooks/                                  # Segment - Custom hooks
│       │   ├── hook-name.hook.ts
│       │   └── index.ts
│       ├── interface/                              # Segment - Global interface definitions
│       │   ├── interface-name.interface.ts
│       │   └── index.ts
│       ├── middleware/                             # Segment - Middlewares
│       ├── service/                                # Segment - Shared services
│       │   ├── service-name.service.ts
│       │   └── index.ts
│       └── util/                                   # Segment - Utility functions
│           ├── util-name.util.ts
│           └── index.ts
├── config/                                         # Application configuration
│   ├── env.config.ts                               # Environment variables validation
│   ├── server.config.ts                            # Server configuration (cors, rate-limit, etc.)
│   ├── locale.config.ts                            # Locale configuration
│   ├── swagger.config.ts                           # Swagger documentation configuration
│   └── index.ts
└── pkg/                                            # External packages/utilities
    ├── auth/                                       # Authentication (plugin, service, constants)
    ├── cache/                                      # Cache client (Redis)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/vitalina-kostenko-js) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
