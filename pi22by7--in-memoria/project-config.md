---
trigger: always_on
description: This file provides instructions for AI agents (Claude Code, GitHub Copilot, Cursor, etc.) on how to effectively use the In Memoria MCP server for intelligent codebase navigation and analysis.
---

# AI Agent Instructions for In Memoria MCP

This file provides instructions for AI agents (Claude Code, GitHub Copilot, Cursor, etc.) on how to effectively use the In Memoria MCP server for intelligent codebase navigation and analysis.

## Quick Start Checklist

Every new session, follow this pattern:

- [ ] 1. Call `get_project_blueprint()` to get instant context
- [ ] 2. Check `learningStatus` in the blueprint response
- [ ] 3. If `recommendation === 'learning_recommended'`, call `auto_learn_if_needed()`
- [ ] 4. Use the blueprint to understand tech stack, entry points, and key directories
- [ ] 5. Leverage feature maps and semantic search for navigation

## Tool Reference Card

### 🎯 Most Important Tools (Use These First)

| Tool | When to Use | Key Feature |
|------|-------------|-------------|
| `get_project_blueprint` | **Every session start** | Instant context: tech stack, entry points, architecture, learning status |
| `auto_learn_if_needed` | When learning recommended | Smart learning with automatic staleness detection |
| `predict_coding_approach` | Before implementing | Get approach + file routing + patterns in one call |
| `search_codebase` | Finding code | Semantic (meaning), text (keywords), or pattern search |
| `analyze_codebase` | Understanding files/dirs | Token-efficient analysis with top concepts/patterns |

### 📊 Complete Tool List (10 Core + 3 Monitoring)

#### Core Intelligence Tools (10)

1. **`analyze_codebase`** - Analyze files or directories
   ```typescript
   { path: string, includeFileContent?: boolean }
   ```
   Returns: Language, concepts (top 10), patterns (top 5), complexity

2. **`search_codebase`** - Smart search (semantic/text/pattern)
   ```typescript
   { query: string, type?: 'semantic'|'text'|'pattern', limit?: number }
   ```
   Returns: Scored results with context

3. **`learn_codebase_intelligence`** - Deep learning
   ```typescript
   { path: string, force?: boolean }
   ```
   Returns: Blueprint, concepts learned, patterns discovered

4. **`get_project_blueprint`** - Instant project context ⭐
   ```typescript
   { path?: string, includeFeatureMap?: boolean }
   ```
   Returns: Tech stack, entry points, key dirs, feature map, **learning status**

5. **`get_semantic_insights`** - Query learned concepts
   ```typescript
   { query?: string, conceptType?: string, limit?: number }
   ```
   Returns: Concepts, relationships, usage contexts

6. **`get_pattern_recommendations`** - Pattern suggestions
   ```typescript
   { problemDescription: string, currentFile?: string, includeRelatedFiles?: boolean }
   ```
   Returns: Patterns, examples, confidence, related files

7. **`predict_coding_approach`** - Implementation guidance
   ```typescript
   { problemDescription: string, context?: object, includeFileRouting?: boolean }
   ```
   Returns: Approach, patterns, complexity, target files

8. **`get_developer_profile`** - Coding style and conventions
   ```typescript
   { includeRecentActivity?: boolean, includeWorkContext?: boolean }
   ```
   Returns: Naming conventions, structural patterns, expertise

9. **`contribute_insights`** - Record architectural decisions
   ```typescript
   { type: string, content: object, confidence: number, sourceAgent: string }
   ```
   Returns: Success, insight ID

10. **`auto_learn_if_needed`** - Smart auto-learning ⭐
    ```typescript
    { path?: string, force?: boolean, skipLearning?: boolean, includeSetupSteps?: boolean }
    ```
    Returns: Action taken, intelligence status, setup steps

#### Monitoring Tools (3 - for debugging)

11. **`get_system_status`** - System health check
12. **`get_intelligence_metrics`** - Concept/pattern metrics
13. **`get_performance_status`** - Performance diagnostics

## Common Use Cases

### Use Case 1: Starting Fresh in a New Codebase

```typescript
// Step 1: Get the lay of the land
const blueprint = await mcp.get_project_blueprint({
  path: '.',
  includeFeatureMap: true
});

console.log('Tech Stack:', blueprint.techStack);
console.log('Entry Points:', blueprint.entryPoints);
console.log('Key Directories:', blueprint.keyDirectories);

// Step 2: Learn if needed
if (blueprint.learningStatus.recommendation !== 'ready') {
  await mcp.auto_learn_if_needed({
    path: '.',
    includeProgress: true
  });
}

// Step 3: You now have full context and intelligence!
```

### Use Case 2: Implementing a New Feature

```typescript
// Step 1: Get implementation approach with file routing
const approach = await mcp.predict_coding_approach({
  problemDescription: 'Add user password reset functionality',
  context: {
    feature: 'authentication',
    relatedFiles: ['src/auth/login.ts']
  },
  includeFileRouting: true
});

// Step 2: Get pattern recommendations for consistency
const patterns = await mcp.get_pattern_recommendations({
  problemDescription: 'Password reset with email validation',
  currentFile: approach.fileRouting.suggestedStartPoint,
  includeRelatedFiles: true
});

// Step 3: Search for similar implementations
const examples = await mcp.search_codebase({
  query: 'email validation auth',
  type: 'semantic',
  limit: 5
});


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pi22by7/In-Memoria](https://github.com/pi22by7/In-Memoria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
