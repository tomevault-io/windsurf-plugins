---
trigger: always_on
description: This is an AI-powered architecture documentation generator that analyzes codebases in any programming language and generates comprehensive markdown documentation using LangChain, LLMs (Anthropic Claude, OpenAI, Google Gemini), and agentic workflows.
---

# Copilot Instructions for architecture-doc-generator

## Project Overview

This is an AI-powered architecture documentation generator that analyzes codebases in any programming language and generates comprehensive markdown documentation using LangChain, LLMs (Anthropic Claude, OpenAI, Google Gemini), and agentic workflows.

**Core Technologies**: TypeScript, LangChain LCEL, Anthropic Claude, LangSmith tracing, Multi-agent architecture

## Response Guidelines

### Documentation Files

**CRITICAL RULE**: Do NOT create markdown documentation files (`.md` files) to summarize changes, fixes, or implementations UNLESS explicitly requested by the user.

**Examples of what NOT to do**:

- ❌ Creating `EMPTY_FILES_FIX.md` after fixing a bug
- ❌ Creating `LANGSMITH_TRACE_HIERARCHY.md` after implementing tracing
- ❌ Creating `LOGGING_MIGRATION_COMPLETE.md` after migration
- ❌ Creating `LOGGER_SINGLETON_REFACTOR.md` after refactoring
- ❌ Creating ANY summary `.md` file after changes/fixes/features
- ❌ Creating documentation files to "document what was done"

**What to do instead**:

- ✅ Make the code changes
- ✅ Provide a concise summary in the chat (2-3 sentences max)
- ✅ Show verification results (build/test/lint status)
- ✅ ONLY create documentation files when user explicitly asks: "document this", "create a guide", "add documentation for X", "write a README about Y"

**Why this matters**:

- Users want changes done, not documented
- Summaries clutter the repository
- Chat responses are sufficient for change tracking
- Documentation should be intentional, not automatic

### Code Style

- **Indentation**: 2 spaces
- **Import Order**: Alphabetical within groups (external → type imports → local)
- **Naming**: camelCase for variables/functions, PascalCase for classes/types/interfaces
- **Comments**: JSDoc for public APIs, inline comments for complex logic only

## Architecture Patterns

### Agent Development Pattern

All agents follow this structure:

```
src/agents/
├── agent.interface.ts            # Base agent interface
├── agent-registry.ts             # Agent registration and discovery
├── base-agent-workflow.ts        # Base class with logging and refinement
├── file-structure-agent.ts       # HIGH priority - Project organization
├── dependency-analyzer-agent.ts  # HIGH priority - Dependency analysis
├── architecture-analyzer-agent.ts # HIGH priority - Architecture design
├── pattern-detector-agent.ts     # MEDIUM priority - Design patterns
├── flow-visualization-agent.ts   # MEDIUM priority - Control/data flows
├── schema-generator-agent.ts     # MEDIUM priority - Data models
└── security-analyzer-agent.ts    # MEDIUM priority - Security vulnerabilities (NEW!)
```

**Agent Implementation Pattern** (uses BaseAgentWorkflow with LangGraph):

All agents extend `BaseAgentWorkflow` which provides:

- ✅ LangGraph self-refinement workflow (analyzeInitial → evaluateClarity → generateQuestions → retrieveFiles → refineAnalysis)
- ✅ Token tracking and budget management
- ✅ LangSmith tracing with proper runNames
- ✅ Agent-owned file generation via `generateFiles()`

**Required implementations**:

```typescript
export class MyCustomAgent extends BaseAgentWorkflow implements Agent {
  // 1. Agent metadata
  public getMetadata(): AgentMetadata {
    return {
      name: 'my-custom-agent',
      version: '1.0.0',
      description: 'What the agent does',
      priority: AgentPriority.MEDIUM,
      capabilities: {
        supportsParallel: false,
        requiresFileContents: true,
        estimatedTokens: 3000,
      },
      tags: ['tag1', 'tag2'],
    };
  }

  // 2. Execution checks
  public async canExecute(context: AgentContext): Promise<boolean> {
    return context.files.length > 0; // Your condition
  }

  public async estimateTokens(context: AgentContext): Promise<number> {
    return 2000 + context.files.length * 5; // Your estimation
  }

  // 3. Agent name (for tracing)
  protected getAgentName(): string {
    return this.getMetadata().name;
  }

  // 4. Prompts for LLM
  protected async buildSystemPrompt(context: AgentContext): Promise<string> {
    return `You are analyzing ${context.projectPath}...`;
  }

  protected async buildHumanPrompt(context: AgentContext): Promise<string> {
    return `Analyze these files:\n${context.files.join('\n')}`;
  }

  // 5. Parse LLM output
  protected async parseAnalysis(analysis: string): Promise<Record<string, unknown>> {
    // Parse JSON or structured output
    return JSON.parse(analysis);
  }

  // 6. Generate summary
  protected generateSummary(data: Record<string, unknown>): string {
    return `Found ${Object.keys(data).length} insights`;
  }

  // 7. Format markdown (backwards compat)
  protected async formatMarkdown(
    data: Record<string, unknown>,
    state: typeof AgentWorkflowState.State,
  ): Promise<string> {
    return `# Analysis\n\n${JSON.stringify(data, null, 2)}`;
  }

  // 8. Generate files (NEW: agent-owned file generation)
  protected async generateFiles(
    data: Record<string, unknown>,
    state: typeof AgentWorkflowState.State,
  ): Promise<AgentFile[]> {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [techdebtgpt/architecture-doc-generator](https://github.com/techdebtgpt/architecture-doc-generator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
