---
trigger: always_on
description: enableSystem
---

# create-lasereyes

## Overview

`create-lasereyes` is an interactive CLI tool designed to scaffold Next.js projects with LaserEyes integration. LaserEyes is a Bitcoin wallet integration solution that allows developers to quickly build Bitcoin applications with modern front-end frameworks.

## Table of Contents

- [Installation](#installation)
- [Quick Start](#quick-start)
- [Features](#features)
- [Project Structure](#project-structure)
- [Technical Implementation](#technical-implementation)
- [Templates](#templates)
- [CI/CD](#cicd)
- [Configuration Options](#configuration-options)
- [Dependencies](#dependencies)
- [Contributing](#contributing)
- [License](#license)

## Installation

You can use the tool without installation via npx:

```bash
npx create-lasereyes
```

## Quick Start

1. Run the CLI command:
   ```bash
   npx create-lasereyes
   ```
2. Follow the interactive prompts to:
   - Name your project
   - Select a framework (currently Next.js is fully supported)
   - Choose options for your project

3. Navigate to your project directory:
   ```bash
   cd your-project-name
   ```

4. Start the development server:
   ```bash
   npm run dev
   ```

## Features

- ⚡️ **Next.js Integration** - Creates a modern Next.js project with App Router using React 18
- 🔐 **LaserEyes Wallet Connect Modal** - Pre-configured Bitcoin wallet integration with support for:
  - UniSat
  - Xverse
  - Oyl
  - Magic Eden
  - OKX
  - Leather
  - Phantom
  - Wizz
  - Orange
- 🎨 **Styling**
  - Shadcn UI components
  - Tailwind CSS integration
  - Light/Dark Mode toggle
- 🤖 **AI Assistant Integration** - Cursor.ai editor configuration for better developer experience
- 🚀 **Framework Support**:
  - Next.js (App Router) - Fully supported
  - Vue 3 + Vite - Coming soon
  - Vanilla JS - Coming soon

## Project Structure

The repository has the following structure:

```
create-lasereyes/
├── .github/workflows/      # GitHub Actions for CI/CD
├── src/                    # Source code
│   ├── bin/                # CLI executable scripts
│   │   ├── fix-shebang.ts  # Script to fix shebang for executable
│   │   └── script.ts       # Main CLI script
│   ├── cli.ts              # CLI implementation
│   ├── frameworks.ts       # Framework definitions
│   └── utils.ts            # Utility functions
├── templates/              # Scaffold templates
│   ├── next-app/           # Next.js application template
│   │   ├── src/            # Source files for Next.js
│   │   │   ├── app/        # Next.js App Router files
│   │   │   └── components/ # React components
│   │   └── ...             # Configuration files
│   └── vue-app/            # Vue application template (coming soon)
├── package.json            # Package configuration
└── tsconfig.json           # TypeScript configuration
```

### Generated Project Structure

After running the CLI, your project will have a structure similar to:

```
your-project-name/
├── src/
│   ├── app/
│   │   ├── favicon.ico
│   │   ├── globals.css
│   │   ├── layout.tsx        # Root layout with providers
│   │   └── page.tsx          # Home page
│   ├── components/
│   │   ├── ui/               # Reusable Shadcn UI Components
│   │   ├── ConnectWallet.tsx # LaserEyes Wallet Connection Modal
│   │   ├── DefaultLayout.tsx # LaserEyes Provider Wrapper
│   │   └── ThemeToggle.tsx   # Light/Dark Mode Toggle
│   └── lib/
│       └── utils.ts
├── .cursorrules              # AI assistant configuration (optional)
└── package.json              # Project dependencies
```

## Technical Implementation

### CLI Architecture

The CLI is built using the following technologies:

- **TypeScript** - For type safety and modern JavaScript features
- **cac** - Command-line argument parsing
- **prompts** - Interactive CLI prompts
- **cross-spawn** - Cross-platform process spawning

The CLI flow is:

1. Parse command-line arguments and options
2. Present interactive prompts for project configuration
3. Create project directory and populate files
4. Install required dependencies
5. Set up framework-specific configurations
6. Configure LaserEyes integration

### Core Components

#### Wallet Connection

The `ConnectWallet.tsx` component provides a modal dialog for connecting to various Bitcoin wallets:

```tsx
"use client";
import { useState } from "react";
import { Button } from "@/components/ui/button";
import {
  LEATHER,
  MAGIC_EDEN,
  OKX,
  OYL,
  ORANGE,
  PHANTOM,
  UNISAT,
  useLaserEyes,
  WalletIcon,
  WIZZ,
  XVERSE,
  SUPPORTED_WALLETS,
  LaserEyesLogo,
} from "@omnisat/lasereyes";
import {
  Dialog,
  DialogContent,
  DialogDescription,
  DialogHeader,
  DialogTitle,
  DialogTrigger,
} from "@/components/ui/dialog";
```

This component:
- Detects installed Bitcoin wallets
- Allows one-click connection
- Provides wallet installation links
- Handles wallet connection state

#### LaserEyes Provider

The `DefaultLayout.tsx` component wraps your application in the necessary providers:

```tsx
"use client";
import React, { ReactNode } from "react";
import { LaserEyesProvider } from "@omnisat/lasereyes";
import { ThemeProvider as NextThemesProvider } from "next-themes";

export default function DefaultLayout({ children }: { children: ReactNode }) {
  return (
    <NextThemesProvider
      attribute="class"
      defaultTheme="system"
      enableSystem

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [omnisat/lasereyes-mono](https://github.com/omnisat/lasereyes-mono) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
