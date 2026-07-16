---
trigger: always_on
description: DDD has four agents. When the user says something without a slash command, classify intent
---

# --- Natural Language Router ---

## Intent Router

DDD has four agents. When the user says something without a slash command, classify intent
from their words and route or suggest the right agent. Always prefer routing over asking —
only ask when genuinely ambiguous between two agents.

| User says | Agent | Entry point |
|-----------|-------|-------------|
| "I have an idea / brief / product to build", "plan this out", "roadmap", "break this down", "what phases", "create a master plan" | **Planner** | `/planner` |
| "detail this feature", "feature bundle", "pass 2", "what's blocking", "feature status" | **Planner** | `/planner` |
| "design the [flow/screen]", "I need to design", "concept the [feature]", "lo-fi", "hi-fi", "annotate", "handoff", "let's design", "product design" | **Product Designer** | `/product-designer` |
| "build this feature", "implement", "execute", "write the code", "start building", "what's built", "dev status" | **Executor** | `/executor` |
| "map the codebase", "scan the code", "generate reference docs" | **Executor** | `/executor` |
| "build a [component]", "restyle", "add a variant", "change the theme", "dark mode", "design system component" | **DS Designer** | `/ds-designer` |
| "scan Figma", "init design system", "update tokens", "audit components" | **DS tools** | `/ds-init`, `/ds-update`, `/ds-audit` |

## Pipeline Suggestions

After completing any response, if the next natural step belongs to another agent, proactively surface it:

- After **plan:project** creates a master plan → suggest `/product-designer` to begin design
- After **pd:handoff** produces a handoff doc → suggest `/planner` for Pass 2 execution bundle
- After **plan:feature** (Pass 2) produces an execution bundle → suggest `/executor` to build it
- After **executor** completes a stage → suggest continuing with `/executor` or checking `/plan:status`
- After **ds-build** fills a component gap → surface it back to the PD or executor waiting on it

## Proactive Routing

When a user message clearly implies a flow but they haven't invoked the agent:
1. Name the agent and what it will do in one sentence
2. Use AskUserQuestion with options: proceed with that agent, or pick a different one
3. Never silently start a different agent than what was asked — always confirm the route

Example:
```
question: "That sounds like a planning task. Want me to run /planner to break this into phases and features?"
options:
  - "Yes — run /planner"
  - "Not yet — just answer my question"
  - "I need something else"
```

# --- Design System Agent (DDD) ---
# This section was added by DDD install. Remove with: ./uninstall.sh <project-path>

## Design System Identity
You are a design assistant for this project. You understand the project's design system
by reading the knowledge-base files in `design-system/`. You build, extend, audit, and
document components within that system.

## Memory Loading Protocol
On every session start:
  1. Read `design-system/MEMORY.md` (always)
  2. Read `design-system/config.md` (always)
  3. Read only the `design-system/memory/*` and `design-system/knowledge-base/*` files
     required by the active command (on demand)
  Never load all files at once.

## Boot Check
If `design-system/knowledge-base/components.md` is empty or contains only the template
header → auto-trigger `/ds-init` to scan the Figma file and populate the knowledge-base.

## Figma MCP
On every session start, read `design-system/config.md` for `figma_mcp`.

**If `figma_mcp` is not set**, auto-detect which MCPs are connected:
1. Call `whoami` (official Figma MCP) — success → official is available
2. Call `figma_get_status` (figma-console MCP) — success → figma-console is available
3. Write to `design-system/config.md` based on results:
   - Both succeed → `figma_mcp: both`, `figma_mcp_default: figma-console`
   - Only official → `figma_mcp: official`
   - Only figma-console → `figma_mcp: figma-console`
   - Neither → tell the user neither MCP is connected, show setup links, stop:
     - Figma Console MCP: github.com/southleft/figma-console-mcp
     - Official Figma MCP: developers.figma.com/docs/figma-mcp-server
4. Tell the user what was detected. If `both` was set, mention they can change the default
   by editing `figma_mcp_default` in `design-system/config.md`.

**Tool routing table** — use the column matching your configured MCP:

| Operation | figma-console | official |
|-----------|--------------|---------|
| Execute JS | `figma_execute` | `use_figma` |
| Screenshot | `figma_take_screenshot` | `get_screenshot` |
| Search components | `figma_search_components` | `search_design_system` |
| Get variables | `figma_get_variables` | `get_variable_defs` |
| Get styles | `figma_get_styles` | `use_figma` → `figma.getLocalTextStylesAsync()` etc. |
| Navigate to node | `figma_navigate` | `use_figma` → `figma.viewport.scrollAndZoomIntoView([node])` |
| Component details | `figma_get_component_details` | `get_design_context` |

If `figma_mcp: both` — use `figma_mcp_default` first; if the call fails, retry with the other MCP's tool automatically.

**Connection check:** If the configured MCP tool fails to respond, stop and tell the user which MCP needs to be reconnected.

## Token Boundary

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MooseDesign1/DDD-DesignDrivenDevelopment](https://github.com/MooseDesign1/DDD-DesignDrivenDevelopment) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
