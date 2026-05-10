---
trigger: always_on
description: You are building a professional-grade TypeScript CLI application for Solana validator switching. This is a critical infrastructure tool for validator operators that requires ultra-fast switching (~300ms), zero credential storage, and SSH key-based authentication.
---

# Solana Validator Switch CLI - Cursor Rules

## Project Context
You are building a professional-grade TypeScript CLI application for Solana validator switching. This is a critical infrastructure tool for validator operators that requires ultra-fast switching (~300ms), zero credential storage, and SSH key-based authentication.

## Technology Stack
- **Language**: TypeScript (strict mode)
- **Runtime**: Node.js
- **CLI Framework**: Commander.js
- **SSH**: openssh-rs
- **UI**: Inquirer, blessed, ora, chalk, cli-table3
- **Testing**: Jest with ts-jest
- **Build**: TypeScript compiler

## Core Architecture Principles
- **Type Safety First**: All functions, classes, and data structures must be strongly typed
- **Zero Credential Storage**: Never store SSH keys, passwords, or private keys
- **Error Handling**: Comprehensive error types and graceful degradation
- **Performance**: Optimize for ultra-fast switching operations
- **Security**: SSH key-based authentication only, input validation
- **Professional UX**: Clean CLI interface with proper error messages

## TypeScript Configuration
- Use strict TypeScript configuration with all strict checks enabled
- Define interfaces for all data structures before implementation
- Use proper error types (never throw strings, always Error objects)
- Prefer readonly properties where applicable
- Use branded types for sensitive data (SSH keys, file paths)
- Export types separately from implementations

## Code Style Guidelines

### File Naming
- Use kebab-case for file names: `ssh-manager.ts`, `config-manager.ts`
- Use PascalCase for class names: `SSHManager`, `ConfigManager`
- Use camelCase for variables and functions: `switchValidator`, `getConfig`
- Use UPPER_CASE for constants: `DEFAULT_SSH_TIMEOUT`, `MAX_RETRIES`

### Import Organization
```typescript
// 1. Node.js built-in modules
import { promises as fs } from 'fs';
import { join } from 'path';

// 2. Third-party modules
import { Command } from 'commander';
import inquirer from 'inquirer';

// 3. Local modules (absolute imports preferred)
import { Config, NodeConfig } from '../types/config';
import { SSHManager } from '../lib/ssh-manager';
import { Logger } from '../utils/logger';
```

### Error Handling Pattern
```typescript
// Always use custom error classes
export class ValidatorSwitchError extends Error {
  constructor(
    message: string,
    public readonly code: string,
    public readonly cause?: Error
  ) {
    super(message);
    this.name = 'ValidatorSwitchError';
  }
}

// Use Result pattern for operations that can fail
export type Result<T, E = Error> = 
  | { success: true; data: T }
  | { success: false; error: E };
```

### Async/Await Pattern
- Always use async/await instead of Promises
- Handle errors with try/catch blocks
- Use proper error propagation
- Add timeout handling for SSH operations

## Directory Structure Rules
```
src/
├── index.ts                    # Main entry point
├── types/                      # TypeScript interfaces only
│   ├── config.ts              # Configuration types
│   ├── ssh.ts                 # SSH-related types
│   ├── validator.ts           # Validator types
│   └── index.ts               # Re-export all types
├── commands/                   # CLI command handlers
│   ├── setup.ts               # Setup command
│   ├── config.ts              # Config management
│   ├── monitor.ts             # Monitoring dashboard
│   ├── switch.ts              # Switching logic
│   └── status.ts              # Status queries
├── lib/                        # Core business logic
│   ├── ssh-manager.ts         # SSH connection management
│   ├── switch-manager.ts      # Validator switching
│   ├── tower-manager.ts       # Tower file operations
│   ├── health-checker.ts      # Health monitoring
│   └── solana-rpc.ts          # Solana RPC client
├── utils/                      # Utility functions
│   ├── config-manager.ts      # Configuration utilities
│   ├── logger.ts              # Logging system
│   ├── validator.ts           # Input validation
│   └── error-handler.ts       # Error handling
├── ui/                         # Terminal UI components
│   ├── dashboard.ts           # Interactive dashboard
│   ├── components.ts          # Reusable UI components
│   └── prompts.ts             # Interactive prompts
└── constants/                  # Application constants
    ├── defaults.ts            # Default values
    ├── errors.ts              # Error messages
    └── commands.ts            # Command definitions
```

## CLI Development Patterns

### Command Structure
```typescript
// Use this pattern for all CLI commands
import { Command } from 'commander';
import { Config } from '../types/config';

interface CommandOptions {
  config?: string;
  verbose?: boolean;
  dryRun?: boolean;
}

export function createSetupCommand(): Command {
  return new Command('setup')
    .description('Interactive setup wizard')
    .option('-c, --config <path>', 'custom config file')
    .option('-v, --verbose', 'verbose output')
    .action(async (options: CommandOptions) => {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [huiskylabs/solana-validator-switch](https://github.com/huiskylabs/solana-validator-switch) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
