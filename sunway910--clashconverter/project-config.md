---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**ClashConverter** is a client-side proxy configuration converter built with Next.js 16. It transforms various proxy protocols (SS, SSR, Vmess, Trojan, Hysteria, VLESS, HTTP, SOCKS5) into Clash YAML or Sing-Box JSON formats.

**Key Design Principle**: Pure frontend static service - user inputs are never stored on backend servers for privacy. All processing happens client-side.

**Task Hook** Execute `pnpm test && pnpm build` after each task item completed, exec `git commit` with `Co-Authored-By: Claude <noreply@anthropic.com>` after `pnpm test && pnpm build`, don't exec `git push` without permissions.


## Development Commands

```bash
pnpm dev      # Start development server on port 3000
pnpm build    # Build for production
pnpm start    # Start production server
pnpm run lint # Run ESLint
npx tsc --noEmit # TypeScript type check

# Testing
pnpm test     # Run all tests
pnpm test:ui  # Run tests with UI
pnpm test:coverage  # Run tests with coverage

# Cloudflare deployment
pnpm build:cf # Build for Cloudflare Workers
pnpm deploy:cf # Deploy to Cloudflare
pnpm preview   # Preview Cloudflare build locally
```

## Technology Stack

- **Framework**: Next.js 16.1.5 with App Router
- **Language**: TypeScript 5.6+ (strict mode)
- **Styling**: Tailwind CSS v3 with shadcn/ui components
- **Icons**: Lucide React
- **Code Editor**: CodeMirror 6 with YAML/JSON language support
- **Internationalization**: next-intl v4
- **Notifications**: Sonner (toast)
- **Theme**: next-themes
- **Deployment**: @opennextjs/cloudflare for Cloudflare Workers
- **Component**: shadcn@latest
- **Validation**: zod v4 for runtime schema validation
- **YAML**: yaml library for YAML parsing/generation

## Architecture

### Path Aliases (configured in tsconfig.json and components.json)
- `@/*` → `./` (root directory)
- `@/components` → Components directory (for shadcn/ui)
- `@/lib` → Library directory

### Component System
- Uses shadcn@latest with RSC (React Server Components) enabled
- Style: "new-york" with "stone" base color
- CSS variables enabled for theming (supports dark/light mode)
- When adding shadcn components: use `npx shadcn@latest add <component>`

## Core Features

### Protocol Support
The app supports 9 proxy protocols:
1. **SS** (Shadowsocks) - `ss://base64#name`
2. **SSR** (ShadowsocksR) - `ssr://base64#name`
3. **VMess** - `vmess://base64(json)#name`
4. **VLESS** - `vless://uuid@server:port?params#name`
5. **Trojan** - `trojan://password@server:port#name`
6. **Hysteria** - `hysteria://server:port?params#name`
7. **Hysteria2** - `hysteria2://password@server:port/?params#name`
8. **HTTP** - `http://user:pass@server:port#name`
9. **SOCKS5** - `socks5://server:port#name`

### Format Support

#### Input Formats
- **Proxy Links** - URI format (ss://, vmess://, etc.)
- **Clash YAML** - Complete configuration files
- **Sing-Box JSON** - JSON configuration format

#### Output Formats
- **Proxy Links** - Shareable URI format
- **Clash Meta (Mihomo)** - Full protocol support
- **Clash Premium** - Limited protocol support (no VLESS/Hysteria)
- **Sing-Box JSON** - No SSR/SOCKS5 support
- **Loon** - iOS client, SS/SSR/VMess/Trojan only (INI format)

### Key Implementation Details

#### Core Architecture (`lib/core/`)
- **Factory Pattern**: `FormatFactory` creates appropriate parsers/generators
- **Registry Pattern**: Auto-initializes all supported formats and protocol adapters on startup
- **Base Classes**: `BaseFormatGenerator` provides common functionality
- **Interfaces**: Strict typing with `IFormatParser` and `IFormatGenerator`
- **Error Handling**: Custom error classes with structured error codes

#### Adapter Pattern (`lib/adapters/`)
Each protocol has its own adapter class implementing `IProtocolAdapter`:
- `toClashJson(node)`: Convert ProxyNode to Clash JSON format
- `toSingBoxJson(node)`: Convert ProxyNode to Sing-Box JSON format
- `toLink(node)`: Convert ProxyNode to shareable link

Benefits:
- Clean abstraction for protocol-specific logic
- Consistent interface for all protocols
- Easy to extend with new protocols
- Eliminates code duplication

#### Type System (`lib/types/`)
- **Strong Typing**: Discriminated union types for all 9 protocols (no `[key: string]: any`)
- **Zod Validation**: Runtime schema validation with `validateProxyNode()`, `safeValidateProxyNode()`
- **Type Guards**: Protocol-specific type guards (`isSSProxy`, `isVMessProxy`, etc.)

```typescript
// Type-safe proxy node (discriminated union)
type ProxyNode =
  | SSProxyNode
  | SSRProxyNode
  | VMessProxyNode
  | VLESSProxyNode
  | TrojanProxyNode
  | HysteriaProxyNode
  | Hysteria2ProxyNode
  | HTTPProxyNode
  | SOCKS5ProxyNode;
```

#### Parser Logic (`lib/parsers/`)
- `parseProxyLink()`: Attempts to parse a single link using all protocol parsers
- `parseMultipleProxies()`: Parses multiple lines, returns `{ proxies, unsupported }`
- Uses yaml library for YAML parsing (replaced custom parser)
- Zod validation for all parsed nodes

#### Generator Logic (`lib/generators/`)
- **Clash YAML Generator**: Uses yaml library + adapters for complete Clash YAML output

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sunway910/clashconverter](https://github.com/sunway910/clashconverter) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
