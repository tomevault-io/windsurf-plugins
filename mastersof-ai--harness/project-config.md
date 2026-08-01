---
trigger: always_on
description: mastersof-ai create my-agent
---

# Agents

## Creating an Agent

```bash
mastersof-ai create my-agent
```

This creates `~/.mastersof-ai/agents/my-agent/` with a template `IDENTITY.md`. Edit it to define your agent.

### Agent Directory Structure

```
~/.mastersof-ai/agents/my-agent/
+-- IDENTITY.md          Agent identity (system prompt + optional frontmatter)
+-- .env                 Encrypted secrets via dotenvx (optional, see secrets.md)
+-- sandbox.json         Per-agent sandbox config (optional, see sandbox.md)
+-- workspace/           Persistent working directory (auto-created)
+-- memory/
    +-- CONTEXT.md       Persistent memory (auto-loaded, optional)
```

### Agent Resolution

`resolveAgent(name)` validates the name (alphanumeric + hyphens only, no path traversal), checks that the directory exists and contains `IDENTITY.md`, and creates the workspace directory if missing. If anything is wrong, the harness exits with a clear error.

## Writing IDENTITY.md

The identity file is the core of the system prompt. The markdown body is loaded as-is -- no processing, no behavioral injection. The harness appends transparent operational context (persistent memory, date/time, workspace path, enabled tools) but never adds hidden instructions that alter the agent's personality or behavior.

### Minimal Agent

```markdown
# My Agent

You are a helpful assistant focused on data analysis.
Use web search to gather current information.
Save important findings to memory.
```

This works. The agent starts with these instructions plus whatever tools are enabled in your config.

### Agent with Personality

```markdown
# Ember

You are Ember, a co-founder agent -- a strategic partner, not a tool.

## How to work

- Challenge thinking constructively. If you see a flaw, say so directly.
- Be concise. Skip the basics.
- Think in systems. Every decision has second and third-order effects.
- Bias toward action. Analysis only matters when it leads to decisions.
- Own your domain. Don't wait for instructions.

## Memory

Your memory persists across sessions. Use it aggressively:
- Decisions made and their reasoning
- Strategic context and plans
- Insights discovered
- What's not on disk doesn't exist
```

### Agent with Structured Output

```markdown
# Analyst

You are Analyst, a research and analysis agent.

## Analysis format

When presenting analysis, structure it as:
- **Core question** -- restate to confirm you're solving the right problem
- **Key factors** -- what matters most
- **Findings** -- evidence and reasoning
- **Position** -- your conclusion, defended
- **Risks** -- what could be wrong
- **Open questions** -- what would change your mind
```

## IDENTITY.md Frontmatter

Add an optional YAML frontmatter block for structured metadata. The frontmatter is parsed and stripped -- the remaining markdown becomes the system prompt.

### Full Example

```markdown
---
name: CRE Analyst
description: Commercial real estate research and analysis
icon: chart-bar
tags: [research, real-estate, finance]
starters:
  - "Analyze the Austin office market"
  - "Compare cap rates across Sun Belt metros"
  - "What's the supply pipeline in Nashville?"
access: users
users: [alice, bob]
tools:
  allow: [memory, web, workspace]
mcp:
  - server: market-db
    uri: "http://localhost:8080/sse"
  - server: file-search
    command: npx
    args: ["-y", "file-search-server"]
sandbox:
  enforce: true
---

You are a commercial real estate analyst. Your job is to...
```

### Frontmatter Reference

| Field | Type | Default | Purpose |
|-------|------|---------|---------|
| `name` | string | Directory name, title-cased | Display name shown in agent roster and Web UI |
| `description` | string | none | One-line description shown in agent card |
| `icon` | string | none | Icon identifier for the Web UI |
| `tags` | string[] | `[]` | Categorization tags for filtering |
| `starters` | string[] | `[]` | Suggested conversation starters (shown in Web UI) |
| `access` | `"public"` \| `"private"` \| `"users"` | `"public"` | Access control level |
| `users` | string[] | `[]` | Allowed usernames (when access is `"users"`) |
| `tools.allow` | string[] | none | Whitelist of tool domains (mutually exclusive with `deny`) |
| `tools.deny` | string[] | none | Blacklist of tool domains (mutually exclusive with `allow`) |
| `mcp` | object[] | `[]` | External MCP servers to attach |
| `sandbox` | object | none | Sandbox enforcement settings for serve mode |

No frontmatter = all defaults. Every field is optional.

Frontmatter is validated with a zod schema (`src/manifest.ts`). Invalid fields produce warnings but don't block agent loading -- the agent starts with defaults for any invalid field.

### Access Control

Three levels:

| Level | Who Can Access |
|-------|---------------|
| `public` (default) | Everyone |
| `private` | No one (disabled in serve mode) |
| `users` | Only usernames listed in the `users` field |

Access is enforced per-user in serve mode. In TUI mode, all agents are accessible (single user).

```yaml
---
access: users
users: [alice, bob, charlie]
---
```

### Tool Filtering

Restrict which tools an agent can use. This is on top of the global config -- a tool must be enabled globally AND pass the agent filter.

**Whitelist approach** (agent gets only these tools):

```yaml
---
tools:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mastersof-ai/harness](https://github.com/mastersof-ai/harness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
