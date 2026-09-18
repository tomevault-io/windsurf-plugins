---
trigger: always_on
description: This rule defines the comprehensive file organization and project structure for zen-flow-ui. Follow this structure when creating new files, organizing components, or refactoring the codebase.
---

# Zen Flow UI Project Structure

## Overview

This rule defines the comprehensive file organization and project structure for zen-flow-ui. Follow this structure when creating new files, organizing components, or refactoring the codebase.

## Root Directory Structure

```
zen-flow-ui/
├── .cursor/                    # Cursor IDE rules and configurations
│   └── rules/                  # Cursor rules for project guidelines
├── .github/                    # GitHub workflows and templates
│   ├── ISSUE_TEMPLATE/         # Issue templates
│   └── workflows/              # CI/CD workflows
├── cli/                        # CLI tools and utilities
│   ├── index.cjs              # Main CLI entry point
│   ├── commands/              # Individual CLI commands
│   ├── templates/             # Project templates
│   └── utils/                 # CLI utility functions
├── docs/                      # Documentation files
│   ├── GETTING_STARTED.md     # Quick start guide
│   ├── CONTRIBUTING.md        # Contribution guidelines
│   ├── DEPLOYMENT.md          # Deployment instructions
│   ├── MIGRATION_GUIDES/      # Version migration guides
│   └── API_REFERENCE/         # Component API documentation
├── examples/                  # Usage examples and demos
│   ├── App.tsx               # Demo application
│   ├── components/           # Example component usage
│   └── templates/            # Starter templates
├── scripts/                  # Build and automation scripts
│   ├── build.js             # Build script
│   ├── setup-tailwind.js    # Tailwind configuration helper
│   ├── generate-exports.js  # Auto-generate export files
│   └── migrate.js           # Migration utilities
├── src/                     # Main source code
│   ├── components/          # React components
│   ├── hooks/              # Custom React hooks
│   ├── lib/                # Utility libraries and core functions
│   ├── styles/             # Global styles and CSS
│   └── types/              # TypeScript type definitions
├── tests/                  # Test utilities and global test setup
│   ├── setup.ts           # Test environment setup
│   ├── utils.ts           # Test utilities
│   └── mocks/             # Mock implementations
├── .eslintrc.json         # ESLint configuration
├── .gitignore            # Git ignore patterns
├── jest.config.js        # Jest testing configuration
├── package.json          # Package configuration and dependencies
├── postcss.config.js     # PostCSS configuration
├── README.md             # Project overview and basic documentation
├── rollup.config.js      # Rollup build configuration
├── tailwind.config.js    # Tailwind CSS configuration
└── tsconfig.json         # TypeScript configuration
```

## Source Code Structure

### Components Directory (`src/components/`)

```
src/components/
├── ui/                           # Core UI components
│   ├── Accordion/               # Accordion component group
│   │   ├── index.ts            # Export barrel
│   │   ├── Accordion.tsx       # Main component
│   │   ├── AccordionItem.tsx   # Sub-component
│   │   ├── types.ts           # Type definitions
│   │   └── Accordion.test.tsx  # Unit tests
│   ├── Alert/                  # Alert component group
│   ├── Button/                 # Button component group
│   │   ├── index.ts           # Export barrel
│   │   ├── Button.tsx         # Main button component
│   │   ├── ButtonGroup.tsx    # Button group component
│   │   ├── types.ts          # Button-specific types
│   │   └── Button.test.tsx    # Button tests
│   ├── Card/                  # Card component group
│   ├── DataTable/             # Data table component group
│   ├── Dialog/                # Dialog component group
│   ├── Form/                  # Form component group
│   │   ├── index.ts          # Export barrel
│   │   ├── Input.tsx         # Input component
│   │   ├── Select.tsx        # Select component
│   │   ├── Textarea.tsx      # Textarea component
│   │   ├── RadioGroup.tsx    # Radio group component
│   │   ├── Toggle.tsx        # Toggle/switch component
│   │   ├── Slider.tsx        # Slider component
│   │   └── types.ts          # Form-related types
│   ├── Layout/               # Layout components
│   │   ├── index.ts         # Export barrel
│   │   ├── Container.tsx    # Container component
│   │   ├── Grid.tsx         # Grid system
│   │   ├── Stack.tsx        # Stack layout
│   │   └── Flex.tsx         # Flex layout
│   ├── Navigation/          # Navigation components
│   │   ├── index.ts        # Export barrel
│   │   ├── Breadcrumb.tsx  # Breadcrumb navigation
│   │   ├── Tabs.tsx        # Tab navigation
│   │   ├── Command.tsx     # Command palette
│   │   └── types.ts        # Navigation types
│   ├── Overlay/            # Overlay components
│   │   ├── index.ts       # Export barrel
│   │   ├── Modal.tsx      # Modal component
│   │   ├── Popover.tsx    # Popover component
│   │   ├── Tooltip.tsx    # Tooltip component
│   │   └── types.ts       # Overlay types
│   ├── Feedback/          # Feedback components
│   │   ├── index.ts      # Export barrel
│   │   ├── Progress.tsx  # Progress indicators

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ingpoc/zen-flow-ui](https://github.com/ingpoc/zen-flow-ui) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-18 -->
