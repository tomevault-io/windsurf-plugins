---
trigger: always_on
description: This repository includes **In Memoria**, an intelligent MCP (Model Context Protocol) server that provides codebase intelligence through semantic analysis, pattern recognition, and smart navigation.
---

# GitHub Copilot Instructions for In Memoria

This repository includes **In Memoria**, an intelligent MCP (Model Context Protocol) server that provides codebase intelligence through semantic analysis, pattern recognition, and smart navigation.

## ⚠️ CRITICAL: Path Parameter Usage

**ALWAYS provide absolute paths to In-Memoria tools. NEVER rely on default values.**

### Why This Matters

In MCP server context, `process.cwd()` is unpredictable and may point to the wrong directory (like `/home/user` instead of your project). Always specify the path explicitly to avoid analyzing the wrong codebase or creating databases in unexpected locations.

```typescript
// ❌ WRONG - May use incorrect directory
await use_mcp_tool('in-memoria', 'get_project_blueprint', {
  includeFeatureMap: true
});

// ✅ CORRECT - Always specify path explicitly
await use_mcp_tool('in-memoria', 'get_project_blueprint', {
  path: '/absolute/path/to/project',  // Use workspace root
  includeFeatureMap: true
});
```

### Getting the Project Path

- In VS Code: Use `${workspaceFolder}` or workspace root API
- Ensure it's an **absolute path** (starts with `/` on Unix, `C:\` on Windows)
- Be **consistent** across all tool calls in a session
- Verify the path exists before calling tools

### Path Convention for All Tools

Every tool that accepts a `path` parameter should receive:
- **Absolute paths** to the project root directory
- **Same path** throughout the entire session
- **No relative paths** like `.` or `./src` (resolve them first)

---

## Core Capabilities

In Memoria learns from codebases and provides:
- **Instant project context** - Tech stack, entry points, key directories, architecture overview
- **Semantic search** - Find code by meaning, not just keywords
- **Pattern recognition** - Discover coding patterns and best practices
- **Smart file routing** - Navigate to relevant files from vague requests
- **Coding approach predictions** - Get implementation suggestions based on learned patterns

## How to Use In Memoria MCP Tools

### 🚀 Quick Start Pattern

**ALWAYS start new sessions with:**
```typescript
// IMPORTANT: Get the absolute project path first
const projectPath = '/absolute/path/to/project'; // or ${workspaceFolder}

// 1. Get instant project blueprint
const blueprint = await use_mcp_tool('in-memoria', 'get_project_blueprint', {
  path: projectPath,  // Always provide path!
  includeFeatureMap: true
});

// Check if learning is needed
if (blueprint.learningStatus.recommendation === 'learning_recommended') {
  // 2. Auto-learn from codebase if needed
  await use_mcp_tool('in-memoria', 'auto_learn_if_needed', {
    path: projectPath,  // Same path throughout session
    includeProgress: true
  });
}
```

This eliminates cold-start exploration and gives you instant context.

### 📊 The 10 Core Tools (Use These Frequently)

#### 1. **analyze_codebase** - Comprehensive Analysis
Use for: Understanding files or directories
```typescript
const projectPath = '/absolute/path/to/project';

// Analyze a specific file
await use_mcp_tool('in-memoria', 'analyze_codebase', {
  path: `${projectPath}/src/components/Header.tsx`
});

// Analyze entire directory
await use_mcp_tool('in-memoria', 'analyze_codebase', {
  path: `${projectPath}/src`
});
```
Returns: Languages, frameworks, complexity, top concepts, top patterns (token-efficient)

#### 2. **search_codebase** - Smart Search
Use for: Finding code by meaning, text, or patterns
```typescript
// Semantic search (finds by meaning)
await use_mcp_tool('in-memoria', 'search_codebase', {
  query: 'authentication logic',
  type: 'semantic',
  limit: 10
});

// Text search (fast keyword matching)
await use_mcp_tool('in-memoria', 'search_codebase', {
  query: 'fetchUserData',
  type: 'text'
});
```

#### 3. **get_project_blueprint** - Instant Context
Use for: Cold-start elimination, understanding project structure
```typescript
await use_mcp_tool('in-memoria', 'get_project_blueprint', {
  path: '/absolute/path/to/project',  // ALWAYS provide path
  includeFeatureMap: true
});
```
Returns: Tech stack, entry points, key directories, feature-to-file mapping, **learning status**

#### 4. **predict_coding_approach** - Implementation Guidance
Use for: Getting suggestions before writing code
```typescript
await use_mcp_tool('in-memoria', 'predict_coding_approach', {
  problemDescription: 'Add user profile editing feature',
  context: { currentFile: 'src/pages/profile.tsx' },
  includeFileRouting: true
});
```
Returns: Recommended approach, patterns, complexity estimate, target files

#### 5. **get_pattern_recommendations** - Pattern Suggestions
Use for: Maintaining consistency with existing code
```typescript
await use_mcp_tool('in-memoria', 'get_pattern_recommendations', {
  problemDescription: 'Create new API endpoint for user search',
  currentFile: 'src/api/routes/users.ts',
  includeRelatedFiles: true
});
```
Returns: Patterns to follow, examples, confidence scores, related files

#### 6. **get_semantic_insights** - Explore Concepts
Use for: Understanding what the codebase knows
```typescript
await use_mcp_tool('in-memoria', 'get_semantic_insights', {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pi22by7/In-Memoria](https://github.com/pi22by7/In-Memoria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
