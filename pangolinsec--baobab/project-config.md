---
trigger: always_on
description: Instructions for Claude Code sessions in this repository.
---

# CLAUDE.md

Instructions for Claude Code sessions in this repository.

## Build & Dev Environment

**Never run node, npx, or npm on the host.** All commands must run inside Docker containers.

```bash
# Dev server (hot reload on port 5173)
docker compose up

# Run a one-off command
docker compose run --rm app <command>

# Type-check
docker compose run --rm app npx tsc --noEmit
```

### Adding dependencies

1. Edit `package.json` on the host directly (don't `npm install` inside a container).
2. Rebuild the image: `docker compose build --no-cache`
3. Recreate containers with fresh node_modules volume: `docker compose up -V`

The anonymous `/app/node_modules` volume is separate per ephemeral container — `docker compose run --rm` won't persist installs.

## Project Overview

Baobab is a tree-based conversation UI for LLMs. Users branch from any assistant response to explore multiple threads, with full linear context preserved from root to leaf. Supports multiple LLM providers (Anthropic, OpenAI, Azure Foundry, Gemini, OpenRouter, HuggingFace, Ollama). See `README.md` for the full description and feature list.

**Key technologies**: React 19, TypeScript, Vite 6, Tailwind CSS v4, Zustand v5, Dexie.js v4 (IndexedDB), @xyflow/react v12 (React Flow), dagre, react-router-dom 7, lucide-react.

## Architecture

```
src/
  types/
    index.ts               # TreeNode, Conversation, Project, AppSettings,
                           #   CoverageScore, ThinkingBlock
    research.ts            # ResearchRun, ResearchPlan, ResearchSubTask
  db/database.ts           # Dexie IndexedDB schema (v11, 7 tables)
  store/
    useTreeStore.ts        # Tree CRUD, node selection, reply target, streaming
    useSettingsStore.ts    # API keys, models, theme, provider configs
    useProjectStore.ts     # Projects and knowledge files
    useResearchStore.ts    # Research run state
  api/
    providers/             # LLMProvider interface + 7 provider implementations
      registry.ts          #   Provider registry and model dispatch
      types.ts             #   ProviderSendParams, ToolDefinition, shared types
      anthropic.ts         #   Anthropic (Claude)
      openai.ts            #   OpenAI (GPT, o-series)
      openai-chat-completions.ts  # Shared Chat Completions streaming (OpenAI + Azure)
      openai-responses-api.ts     # Responses API for reasoning models (o-series)
      azure.ts             #   Azure Foundry (per-deployment config)
      gemini.ts            #   Google Gemini
      openrouter.ts        #   OpenRouter
      huggingface.ts       #   HuggingFace Inference
      ollama.ts            #   Ollama (local)
      callProvider.ts      #   Shared provider call wrapper
    tools.ts               # Tool definitions and executors (web_search, read_file)
    search.ts              # Web search dispatch (DuckDuckGo, Tavily, Bing)
  agents/
    scorer.ts              # Coverage scoring against target terms
    research/
      researchRunner.ts    # Research orchestration pipeline
      planner.ts           # LLM-based research plan generation
      subAgent.ts          # Per-subtask tool-calling agent
      synthesizer.ts       # Combines findings into final report
  lib/
    tree.ts                # Path-to-root, React Flow graph builder, dagre layout
    messageBuilder.ts      # Build API messages from node path (thinking blocks, tool calls)
    pricing.ts             # Cost calculations, live pricing fetch
    generateTitle.ts       # LLM-based conversation title generation
    merge.ts               # Branch merge logic
    summarize.ts           # Branch summarization
    search.ts              # Full-text search over nodes
    fileReferences.ts      # @file mention parsing
  hooks/
    useStreamingResponse.ts  # Send flow: user node -> stream -> finalize
    useTreeLayout.ts         # Memoized dagre layout
    useContextMenu.ts        # Right-click context menu logic
  components/
    layout/                # MainLayout, Sidebar, SidebarConvItem
    pages/                 # LandingPage, ConversationView, SettingsPage,
                           #   ProjectDetailPage
    tree/                  # TreeView, MessageNode, NodeDetailPanel,
                           #   ContextMenu, MergeDialog, SummarizeDialog,
                           #   DuplicateEditModal, ManualNodeModal
    thread/                # ThreadView, ThreadMessage, BranchIndicator
    chat/                  # ChatInput, FileMentionDropdown
    research/              # ResearchConfigModal, ResearchView, ResearchReport
    project/               # ProjectDialog, ProjectFileList, FileUploadButton
    settings/              # ManageModelsDialog
    shared/                # ModelSelector, TagInput, ProjectAssignDropdown,
                           #   ReasoningBlocksSection
```

### Routes

| Path | Component | Purpose |
|------|-----------|---------|
| `/` | `LandingPage` | Empty state / new conversation CTA |
| `/c/:conversationId` | `ConversationView` | Tree + thread views, chat input, panels |
| `/settings/:section?` | `SettingsPage` | Settings sections (general, providers, prompts, etc.) |
| `/project/:projectId` | `ProjectDetailPage` | Project files, conversations, knowledge mode |

### Data flow


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pangolinsec/baobab](https://github.com/pangolinsec/baobab) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
