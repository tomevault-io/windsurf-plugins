---
trigger: always_on
description: **Rule Priority:** Advanced Integration
---

# Git Hooks Integration with Cursor AI Agents

**Rule Priority:** Advanced Integration  
**Activation:** Git operations and commit workflows  
**Scope:** All repository files and Git operations

## Overview

Cursor v1.2+ enables powerful Git commit hook integration where AI agents can automatically respond to Git events, performing validation, testing, and quality checks without manual intervention. This creates a seamless development workflow where code quality is enforced automatically.

## Pre-Commit Hook Patterns

### Automatic Code Quality Validation

```bash
# .git/hooks/pre-commit (or use husky/lint-staged)
#!/bin/bash

# Let Cursor AI agent handle all validation
echo "🤖 Cursor AI Agent: Running pre-commit validation..."

# TypeScript type checking
npm run type-check || {
    echo "❌ TypeScript errors detected. Let Cursor AI fix them..."
    # Cursor agent will see this output and automatically fix type errors
    exit 1
}

# ESLint validation
npm run lint || {
    echo "❌ ESLint errors detected. Let Cursor AI fix them..."
    # Cursor agent will automatically apply lint fixes
    exit 1
}

# Test suite
npm run test || {
    echo "❌ Tests failing. Let Cursor AI investigate and fix..."
    exit 1
}

echo "✅ All pre-commit checks passed!"
```

### SYMindX-Specific Pre-Commit Validation

```typescript
// scripts/pre-commit-validation.ts
import { validateAgentConfig } from '../mind-agents/src/utils/validation';
import { checkMemoryProviders } from '../mind-agents/src/memory/validation';
import { validatePortalConfigs } from '../mind-agents/src/portals/validation';

async function preCommitValidation(): Promise<void> {
  console.log('🧠 SYMindX Pre-Commit Validation');
  
  // Validate agent configurations
  await validateAgentConfig();
  
  // Check memory provider configurations
  await checkMemoryProviders();
  
  // Validate AI portal configurations
  await validatePortalConfigs();
  
  // Ensure all emotion modules are properly typed
  await validateEmotionSystem();
  
  console.log('✅ SYMindX validation complete');
}
```

## Post-Commit Hook Automation

### Automatic Documentation Updates

```bash
# .git/hooks/post-commit
#!/bin/bash

# Let Cursor AI agent handle post-commit tasks
echo "🤖 Cursor AI Agent: Running post-commit automation..."

# Update documentation if code changes
if git diff --name-only HEAD~1 HEAD | grep -E '\.(ts|tsx)$'; then
    echo "📚 Code changes detected. Updating documentation..."
    # Cursor agent will automatically update docs
fi

# Update type definitions
if git diff --name-only HEAD~1 HEAD | grep -E 'src/types/'; then
    echo "🔧 Type changes detected. Regenerating exports..."
    npm run build:types
fi

# Trigger deployment if on main branch
if [ "$(git branch --show-current)" = "main" ]; then
    echo "🚀 Main branch updated. Preparing deployment..."
fi
```

### Character System Updates

```typescript
// scripts/post-commit-character-sync.ts
import { syncCharacterDefinitions } from '../mind-agents/src/characters/sync';

async function postCommitCharacterSync(): Promise<void> {
  const changedFiles = process.env.CHANGED_FILES?.split(',') || [];
  
  // Check if character definitions were modified
  const characterFiles = changedFiles.filter(file => 
    file.includes('characters/') && file.endsWith('.json')
  );
  
  if (characterFiles.length > 0) {
    console.log('🎭 Character definitions updated. Syncing...');
    await syncCharacterDefinitions(characterFiles);
    console.log('✅ Character sync complete');
  }
}
```

## Branch-Specific Hook Workflows

### Development Branch Hooks

```bash
# Branch-specific pre-commit (development branches)
if [[ $(git branch --show-current) =~ ^(feature|bugfix|hotfix)/ ]]; then
    echo "🔧 Development branch detected. Running extended validation..."
    
    # Extended type checking for experimental features
    npm run type-check:strict
    
    # Performance regression testing
    npm run test:performance
    
    # Memory leak detection for agent modules
    npm run test:memory-leaks
fi
```

### Production Branch Hooks

```bash
# Production branch pre-commit (main/production)
if [[ $(git branch --show-current) =~ ^(main|production)$ ]]; then
    echo "🏭 Production branch detected. Running full validation suite..."
    
    # Full test suite including integration tests
    npm run test:full
    
    # Security vulnerability scanning
    npm audit --audit-level=moderate
    
    # Build validation
    npm run build
    
    # Bundle size analysis
    npm run analyze:bundle
fi
```

## AI Agent Hook Integration

### Cursor Agent Response Patterns

```typescript
// Integration with Cursor AI agents for hook responses
interface HookResponse {
  hookType: 'pre-commit' | 'post-commit' | 'pre-push';
  success: boolean;
  errors: Array<{
    file: string;
    line: number;
    message: string;
    fixable: boolean;
  }>;
  autoFixes: string[];
}

// Cursor agent will automatically:
// 1. Parse hook output
// 2. Identify fixable issues
// 3. Apply fixes automatically
// 4. Re-run hooks until success
// 5. Commit fixes with descriptive messages
```

### Hook Configuration

```json
// .cursor/git-hooks.json
{
  "preCommit": {
    "enabled": true,
    "autoFix": true,
    "checks": [
      "typescript",
      "eslint", 
      "tests",
      "formatting",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
