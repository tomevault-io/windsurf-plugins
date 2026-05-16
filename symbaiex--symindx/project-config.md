---
trigger: always_on
description: **Rule Priority:** Advanced Automation
---

# Background Agents for Cloud-Powered Development

**Rule Priority:** Advanced Automation  
**Activation:** Complex multi-step tasks and parallel workflows  
**Scope:** All development tasks suitable for agent delegation

## Overview

Cursor's Background Agents feature enables cloud-powered AI agents to work on tasks in parallel while you focus on core development. These agents can handle UI fixes, content updates, pull request creation, and complex refactoring tasks without blocking your main workflow.

## Background Agent Activation

### Settings Configuration

```json
// .cursor/background-agents.json
{
  "enabled": true,
  "maxConcurrentAgents": 5,
  "autoCreatePRs": true,
  "branchPrefix": "agent/",
  "githubIntegration": true,
  "costLimit": {
    "dailyMax": 50.00,
    "warningThreshold": 40.00
  },
  "agentTypes": {
    "uiFixes": true,
    "contentUpdates": true,
    "refactoring": true,
    "testing": true,
    "documentation": true
  }
}
```

### Authentication Setup

```bash
# Enable Background Agents in Cursor Settings
# 1. Settings → Beta → Enable Background Agents
# 2. Authenticate GitHub for PR handling
# 3. Snapshot environment for cloud agents
# 4. Configure cost limits and permissions
```

## Agent Task Templates

### UI Fix Agent Template

```typescript
// Template for delegating UI fixes to background agents
interface UIFixTask {
  type: 'ui-fix';
  description: string;
  targetFiles: string[];
  screenshot?: string;
  requirements: {
    responsive: boolean;
    accessibility: boolean;
    designSystem: string;
  };
  priority: 'low' | 'medium' | 'high';
}

// Example UI fix delegation
const uiFixTask: UIFixTask = {
  type: 'ui-fix',
  description: 'Fix mobile responsive layout for navbar component',
  targetFiles: ['website/src/components/Navigation.tsx'],
  requirements: {
    responsive: true,
    accessibility: true,
    designSystem: 'tailwind'
  },
  priority: 'medium'
};

// Cursor will automatically:
// 1. Create new branch: agent/fix-navbar-mobile
// 2. Apply responsive fixes
// 3. Test across breakpoints
// 4. Create pull request with screenshots
// 5. Notify when ready for review
```

### Content Update Agent Template

```typescript
// Template for content updates and data synchronization
interface ContentUpdateTask {
  type: 'content-update';
  description: string;
  contentType: 'documentation' | 'config' | 'assets' | 'data';
  sourceFiles: string[];
  updatePattern: 'replace' | 'merge' | 'append';
  validation: string[];
}

// Example content update
const contentTask: ContentUpdateTask = {
  type: 'content-update',
  description: 'Update AI portal configurations with new models',
  contentType: 'config',
  sourceFiles: ['mind-agents/src/portals/*/config.json'],
  updatePattern: 'merge',
  validation: ['schema-validation', 'connectivity-test']
};
```

### Refactoring Agent Template

```typescript
// Template for complex refactoring tasks
interface RefactoringTask {
  type: 'refactoring';
  description: string;
  scope: 'file' | 'module' | 'system';
  targetPattern: string;
  transformations: RefactoringTransformation[];
  preserveTests: boolean;
}

interface RefactoringTransformation {
  name: string;
  description: string;
  pattern: string;
  replacement: string;
  validation: string;
}

// Example refactoring task
const refactorTask: RefactoringTask = {
  type: 'refactoring',
  description: 'Extract common portal interface from all AI providers',
  scope: 'module',
  targetPattern: 'mind-agents/src/portals/*/index.ts',
  transformations: [
    {
      name: 'extract-interface',
      description: 'Create common BasePortal interface',
      pattern: 'export class (.+)Portal',
      replacement: 'export class $1Portal extends BasePortal',
      validation: 'typescript-compile'
    }
  ],
  preserveTests: true
};
```

## Parallel Task Execution

### Multi-Agent Workflows

```typescript
// Coordinate multiple agents for complex tasks
interface MultiAgentWorkflow {
  name: string;
  agents: AgentTask[];
  dependencies: AgentDependency[];
  mergeStrategy: 'sequential' | 'parallel' | 'hybrid';
}

interface AgentTask {
  id: string;
  type: string;
  description: string;
  estimatedTime: number;
  priority: number;
}

interface AgentDependency {
  agentId: string;
  dependsOn: string[];
  condition: 'completion' | 'approval' | 'merge';
}

// Example: Complete feature implementation
const featureWorkflow: MultiAgentWorkflow = {
  name: 'Add New Emotion Support',
  agents: [
    {
      id: 'emotion-types',
      type: 'code-generation',
      description: 'Add new emotion types to type definitions',
      estimatedTime: 300, // 5 minutes
      priority: 1
    },
    {
      id: 'emotion-logic',
      type: 'implementation',
      description: 'Implement emotion calculation logic',
      estimatedTime: 900, // 15 minutes
      priority: 2
    },
    {
      id: 'emotion-tests',
      type: 'testing',
      description: 'Generate comprehensive test suite',
      estimatedTime: 600, // 10 minutes
      priority: 3
    },
    {
      id: 'emotion-docs',
      type: 'documentation',
      description: 'Update documentation and examples',
      estimatedTime: 480, // 8 minutes
      priority: 4
    }
  ],
  dependencies: [
    {
      agentId: 'emotion-logic',
      dependsOn: ['emotion-types'],

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SYMBaiEX/SYMindX](https://github.com/SYMBaiEX/SYMindX) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-15 -->
