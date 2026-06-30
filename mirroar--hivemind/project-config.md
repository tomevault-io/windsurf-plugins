---
trigger: always_on
description: **Hivemind** is an advanced AI bot for the game Screeps (https://screeps.com), written in **TypeScript**. The bot autonomously manages rooms, resources, expansion, construction, mining, power harvesting, commodities, and trading. It is designed to run on both private and official Screeps servers. The codebase contains ~12,000 lines of TypeScript across 201 source files.
---

# Hivemind - Copilot Coding Agent Instructions

## Project Overview
**Hivemind** is an advanced AI bot for the game Screeps (https://screeps.com), written in **TypeScript**. The bot autonomously manages rooms, resources, expansion, construction, mining, power harvesting, commodities, and trading. It is designed to run on both private and official Screeps servers. The codebase contains ~12,000 lines of TypeScript across 201 source files.

**Key Details:**
- **Language:** TypeScript (transpiled to JavaScript via Rollup)
- **Runtime:** Node.js v16 (required - project will NOT build on Node v20+)
- **Size:** Medium (~200 TypeScript files, 12k+ lines)
- **Target Platform:** Screeps game servers
- **Build Tool:** Rollup with TypeScript plugin
- **Linter:** XO (ESLint wrapper)
- **Package Manager:** npm

---

## Critical Build Requirements

### **ALWAYS use Node.js v16**
The project requires Node.js v16 and **will fail to build** with Node v20 or later due to incompatible dependencies (tslib, rollup-plugin-typescript2). If you encounter build errors like "Package subpath './package.json' is not defined by exports", you are using the wrong Node version.

**To switch to Node 16:**
```bash
source /home/runner/.nvm/nvm.sh && nvm install 16 && nvm use 16
```

**Verification:**
```bash
node --version  # Should output v16.x.x
```

### **Required Local Configuration Files**
The build will fail without these files. They must be created **before** building:

```bash
cp settings.local.example.ts src/settings.local.ts
cp relations.local.example.ts src/relations.local.ts
```

These files allow users to customize bot behavior and diplomatic relations. They are in `.gitignore` and should NOT be committed.

---

## Build & Test Commands

**Always run commands in this exact order:**

### 1. Install Dependencies
```bash
npm install
```
**Time:** ~5-10 seconds (after npm cache is populated)  
**Notes:** You may see deprecation warnings (urix, resolve-url, chokidar) and security vulnerabilities - these are pre-existing and can be ignored.

### 2. Build the Project
```bash
npm run build
```
**Equivalent to:** `rollup -c`  
**Time:** ~14-16 seconds  
**Output:** Creates `dist/main.js` and `dist/main.js.map.js`

**Expected warnings during build:**
- Deprecation warning about tslib package exports - **safe to ignore**
- Circular dependency warnings about `cost-matrix.ts`, `room-defense.ts`, `nav-mesh.ts`, and `room-intel.ts` - **these are known and do not break functionality**
- "No destination specified - code will be compiled but not uploaded" - **normal** (upload requires `screeps.json` config)

**Build will fail if:**
- Node version is not 16
- Local settings files are missing (see above)

### 3. Lint the Code
```bash
npm test
```
**Equivalent to:** `xo`  
**Time:** ~3-5 seconds

**Important:** The codebase currently has **4000+ linting errors**. These are pre-existing issues. When making changes:
- Only fix linting errors **directly related** to your changes
- Do NOT attempt to fix all linting errors globally
- Focus on making your new code follow the linting rules

**Common XO rules to follow:**
- Use `===` instead of `==`
- Avoid `@typescript-eslint/no-unsafe-*` violations
- Follow brace-style: `stroustrup` (else/catch on new line)
- Use `'error'` log level for comma-dangle: `'always-multiline'`

---

## Project Structure

### Root Directory Files
- `package.json` - Dependencies and npm scripts
- `tsconfig.json` - TypeScript compiler configuration
- `rollup.config.js` - Build configuration (Rollup bundler)
- `.nvmrc` - Node version specification (v16)
- `README.md` - User documentation
- `CHANGELOG.md` - Version history
- `snippets.js` - Console commands for manual bot interaction (not imported by code)
- `Gruntfile.example.js` - Example Grunt config for deploying to Screeps servers
- `settings.local.example.ts` - Example settings customization
- `relations.local.example.ts` - Example diplomatic relations config
- `.gitignore` - Excludes `src/*.local.ts`, `dist/`, `node_modules/`, `screeps.json`

### Source Code Organization (`/src`)

**Main Entry Point:**
- `src/main.ts` - Application entry point, imports all processes and prototypes

**Core Bot Logic:**
- `src/hivemind.ts` - Core kernel/process scheduler
- `src/utilities.ts` - General utility functions
- `src/settings-manager.ts` - Settings management (imports `settings.local.ts`)
- `src/relations.ts` - Diplomatic relations (imports `relations.local.ts`)
- `src/settings.default.ts` - Default bot settings

**Key Subdirectories:**
- `src/process/` - Game loop processes (cleanup, creeps, resources, rooms, trade, strategy, etc.)
- `src/process/strategy/` - High-level strategy processes (expand, scout, mining, power, deposits, intershard)
- `src/role/` - Creep role behaviors (brawler, builder, hauler, harvester, etc.)
- `src/spawn-role/` - Creep spawning logic per role
- `src/room/` - Room management (defense, planner, operation)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Mirroar/hivemind](https://github.com/Mirroar/hivemind) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-29 -->
