---
trigger: always_on
description: **Generated:** 2026-04-18 | **Commit:** 2892ca4a | **Branch:** dev
---

# oh-my-opencode — OpenCode Plugin

**Generated:** 2026-04-18 | **Commit:** 2892ca4a | **Branch:** dev

## OVERVIEW

OpenCode plugin (npm: `oh-my-opencode`, dual-published as `oh-my-openagent` during transition) extending Claude Code with 11 agents, 52 lifecycle hooks, 26 tools, 3-tier MCP system (built-in + .mcp.json + skill-embedded), Hashline LINE#ID edit tool, IntentGate classifier, and Claude Code compatibility. 1766 TypeScript source files, 377k LOC, 104 barrel index.ts files. Entry: `src/index.ts` → 5-step init (loadConfig → createManagers → createTools → createHooks → createPluginInterface).

## STRUCTURE

```
oh-my-opencode/
├── src/
│   ├── index.ts              # Plugin entry: default export `pluginModule`, shape `{ id, server }`
│   ├── plugin-config.ts      # JSONC multi-level config: user → project → defaults (Zod v4)
│   ├── agents/               # 11 agents (Sisyphus, Hephaestus, Oracle, Librarian, Explore, Atlas, Prometheus, Metis, Momus, Multimodal-Looker, Sisyphus-Junior)
│   ├── hooks/                # 52 lifecycle hooks across dedicated modules and standalone files
│   ├── tools/                # 26 tools across 16 directories (includes Hashline edit with LINE#ID content hashing)
│   ├── features/             # 19 feature modules (background-agent, skill-loader, tmux, MCP-OAuth, skill-mcp-manager, etc.)
│   ├── shared/               # 170+ utility files (barrel-exported, logger → /tmp/oh-my-opencode.log)
│   ├── config/               # Zod v4 schema system (32 files)
│   ├── cli/                  # CLI: install, run, doctor, mcp-oauth (Commander.js)
│   ├── mcp/                  # 3 built-in remote MCPs (websearch, context7, grep_app)
│   ├── plugin/               # 10 OpenCode hook handlers + 52 hook composition
│   ├── plugin-handlers/      # 6-phase config loading pipeline
│   └── openclaw/             # Bidirectional external integration (Discord/Telegram/webhook/command)
├── packages/                 # 11 platform-specific compiled binaries (darwin/linux/windows, AVX2 + baseline variants)
├── script/                   # Build/publish automation (singular, not scripts/)
├── .sisyphus/                # AI agent workspace (rules, plans, tasks, notepads)
└── .local-ignore/            # Dev-only test fixtures + PR worktrees
```

## INITIALIZATION FLOW

```
pluginModule.server(input, options)
  ├─→ loadPluginConfig()         # JSONC parse → project/user merge → Zod validate → migrate
  ├─→ createManagers()           # TmuxSessionManager, BackgroundManager, SkillMcpManager, ConfigHandler
  ├─→ createTools()              # SkillContext + AvailableCategories + ToolRegistry (26 tools)
  ├─→ createHooks()              # 3-tier: Core(43) + Continuation(7) + Skill(2) = 52 hooks
  └─→ createPluginInterface()    # 10 OpenCode hook handlers → PluginInterface
```

## 10 OPENCODE HOOK HANDLERS

| Handler | Purpose |
|---------|---------|
| `config` | 6-phase: provider → plugin-components → agents → tools → MCPs → commands |
| `tool` | 26 registered tools |
| `chat.message` | First-message variant, session setup, keyword detection (ultrawork/search/analyze) |
| `chat.params` | Anthropic effort level, think mode, runtime fallback override |
| `chat.headers` | Copilot x-initiator header injection |
| `event` | Session lifecycle (created, deleted, idle, error), openclaw dispatch, runtime fallback |
| `tool.execute.before` | Pre-tool hooks (file guard, label truncator, rules injector, prometheus md-only) |
| `tool.execute.after` | Post-tool hooks (output truncation, comment checker, hashline read enhancer) |
| `experimental.chat.messages.transform` | Context injection, thinking block validation, tool pair validation |
| `experimental.session.compacting` | Context + todo preservation during compaction |

## WHERE TO LOOK

| Task | Location | Notes |
|------|----------|-------|
| Add new agent | `src/agents/` + `src/agents/builtin-agents/` | Follow createXXXAgent factory pattern |
| Add new hook | `src/hooks/{name}/` + register in `src/plugin/hooks/create-*-hooks.ts` | Match event type to tier |
| Add new tool | `src/tools/{name}/` + register in `src/plugin/tool-registry.ts` | Follow createXXXTool factory |
| Add new feature module | `src/features/{name}/` | Standalone module, wire in plugin/ |
| Add new MCP | `src/mcp/` + register in `createBuiltinMcps()` | Remote HTTP only (tier 1 of 3) |
| Add new skill | `src/features/builtin-skills/skills/` | Implement BuiltinSkill interface |
| Add new command | `src/features/builtin-commands/` | Template in templates/ |
| Add new CLI command | `src/cli/cli-program.ts` | Commander.js subcommand |
| Add new doctor check | `src/cli/doctor/checks/` | Register in checks/index.ts |
| Modify config schema | `src/config/schema/` + update root schema | Zod v4, add to OhMyOpenCodeConfigSchema |
| Add new category | `src/tools/delegate-task/constants.ts` | DEFAULT_CATEGORIES + CATEGORY_MODEL_REQUIREMENTS |
| Debug provider errors | `src/hooks/runtime-fallback/` | Reactive error recovery (distinct from model-fallback) |
| External notifications | `src/openclaw/` | Bidirectional Discord/Telegram/webhook integration |
| Skill-embedded MCP | `src/features/skill-mcp-manager/` | Tier 3 MCPs (stdio + HTTP, per-session) |

## MULTI-LEVEL CONFIG

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [code-yeongyu/oh-my-openagent](https://github.com/code-yeongyu/oh-my-openagent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
