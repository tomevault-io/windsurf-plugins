---
trigger: always_on
description: **Version 1.0.2** - TypeScript ESM project for a Model Context Protocol (MCP) server that integrates with FacturaScripts ERP system, providing comprehensive access to business, accounting, and administrative data.
---

# MCP FacturaScripts

**Version 1.0.2** - TypeScript ESM project for a Model Context Protocol (MCP) server that integrates with FacturaScripts ERP system, providing comprehensive access to business, accounting, and administrative data.

## Project Structure

The project follows a **modular architecture** that groups related functionality for better organization and maintainability:

- `src/index.ts` - Main MCP server entry point
- `src/env.ts` - Environment validation using Zod
- `src/fs/client.ts` - Axios-based FacturaScripts API client
- `src/types/facturascripts.ts` - TypeScript interfaces for all FacturaScripts entities
- `src/utils/filterParser.ts` - Dynamic filtering and sorting utilities

### Modular Organization

```
src/modules/
├── core-business/          # Essential business entities
│   ├── clientes/           # Customer management
│   ├── productos/          # Product catalog
│   ├── proveedores/        # Supplier management
│   └── stocks/             # Inventory management
├── sales-orders/           # Sales and order processing
│   ├── pedidoclientes/     # Customer orders
│   ├── facturaclientes/    # Customer invoices
│   ├── presupuestoclientes/# Customer quotes
│   ├── albaranclientes/    # Customer delivery notes
│   └── line-items/         # All document line items
├── purchasing/             # Procurement and supplier operations
│   ├── facturaproveedores/ # Supplier invoices
│   ├── albaranproveedores/ # Supplier delivery notes
│   └── productoproveedores/# Products by supplier
├── accounting/             # General accounting
│   ├── asientos/           # Accounting entries
│   ├── cuentas/            # Chart of accounts
│   ├── diarios/            # Accounting journals
│   ├── ejercicios/         # Fiscal years
│   └── conceptopartidas/   # Entry concepts
├── finance/                # Financial management
│   ├── cuentabancos/       # Bank accounts
│   ├── cuentabancoclientes/# Customer bank accounts
│   ├── cuentabancoproveedores/# Supplier bank accounts
│   ├── cuentaespeciales/   # Special accounts
│   └── divisas/            # Currencies
├── configuration/          # System configuration
│   ├── almacenes/          # Warehouses
│   ├── agentes/            # Sales agents
│   ├── formapagos/         # Payment methods
│   ├── impuestos/          # Tax rates
│   ├── familias/           # Product families
│   ├── fabricantes/        # Manufacturers
│   └── [14 more modules]   # Complete configuration coverage
├── system/                 # System administration
│   ├── apiaccess/          # API access control
│   ├── apikeyes/           # API key management
│   ├── logmessages/        # System logs
│   └── [4 more modules]    # System management
├── communication/          # Communications
│   ├── emailnotifications/ # Email templates
│   ├── emailsentes/        # Email history
│   └── contactos/          # Contact management
└── geographic/             # Geographic data
    ├── ciudades/           # Cities
    ├── codigopostales/     # Postal codes
    └── empresas/           # Company locations
```

### Module Structure

Each module follows a consistent pattern:
```
module-name/
├── resource.ts    # MCP resource implementation
├── tool.ts        # Claude Desktop tool definition
└── index.ts       # Module exports
```

This modular architecture provides:
- **Better Organization**: Related functionality grouped together
- **Easier Maintenance**: Changes isolated to specific modules
- **Cleaner Code**: Smaller, focused files instead of large monoliths
- **Enhanced Testability**: Individual modules can be tested in isolation
- **Scalability**: Easy to add new modules or modify existing ones

## Configuration

Environment variables (see `.env.example`):
- `FS_BASE_URL` - Base URL of your FacturaScripts instance
- `FS_API_VERSION` - API version (default: 3)
- `FS_API_TOKEN` - API authentication token

## Available Scripts

- `npm run dev` - Run development server (auto-builds, runs JavaScript)
- `npm run build` - Build TypeScript to dist/
- `npm run start` - Run built server (same as mcp)
- `npm run mcp` - Build and run MCP server (recommended for MCP Inspector)
- `npm run dev:ts` - Run TypeScript directly with tsx (development only, not for MCP Inspector)
- `npm run test` - Run tests with Vitest
- `npm run test:watch` - Run tests in watch mode
- `npm run test:ui` - Run tests with Vitest UI
- `npm run test:run` - Run tests once and exit

## MCP Inspector Usage

The server includes a robust entry point (`mcp-server.js`) that automatically builds the project and runs the JavaScript version for maximum compatibility.

**✅ Recommended for MCP Inspector**:
```bash
# Using MCP Inspector
npx @modelcontextprotocol/inspector npm run mcp

# Or directly with node
node mcp-server.js

# Or any of these npm scripts
npm start
npm run dev
npm run mcp
```

**✅ All scripts now use the built JavaScript version** - The project automatically builds and runs the compiled code to avoid ESM module resolution issues.

**⚠️ Development only**: TypeScript source execution
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cristotodev/MCP-Facturascripts](https://github.com/cristotodev/MCP-Facturascripts) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
