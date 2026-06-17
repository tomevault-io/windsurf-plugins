---
trigger: always_on
description: This guide covers how to integrate `computer-use-mcp` into AI agent frameworks and agentic workflows. Works on **macOS**, **Windows**, and **Linux**.
---

# Using computer-use-mcp with AI Agents

This guide covers how to integrate `computer-use-mcp` into AI agent frameworks and agentic workflows. Works on **macOS**, **Windows**, and **Linux**.

## Tool priority guidance

Desktop computer use should be your **last resort**. Always prefer more precise tools:

1. **Connector / integration** — structured APIs, MCP connectors, app-specific plugins
2. **Shell / filesystem** — terminal commands, file I/O, CLI tools
3. **Browser automation** — Playwright, Puppeteer, Browser Use for web pages
4. **Desktop computer use** — this package, for native desktop apps, simulators, installers, modal dialogs, and UI-only workflows

Desktop control works for anything on screen, but structured tools are faster, more reliable, and easier to recover from.

## Quick setup for any agent

The server speaks standard MCP over stdio. Start it with:

```bash
npx @zavora-ai/computer-use-mcp
```

Any agent framework with MCP support can connect to it immediately.

## Codex

Codex reads local MCP servers from `~/.codex/config.toml`:

```toml
[mcp_servers.computer-use]
command = "npx"
args = ["--yes", "--prefer-offline", "@zavora-ai/computer-use-mcp"]
```

You can confirm the server is available with:

```bash
codex mcp list
```

## Claude (Anthropic)

### Claude Desktop

Add to your Claude Desktop config:
- **macOS:** `~/Library/Application Support/Claude/claude_desktop_config.json`
- **Windows:** `%APPDATA%\Claude\claude_desktop_config.json`

```json
{
  "mcpServers": {
    "computer-use": {
      "command": "npx",
      "args": ["--yes", "--prefer-offline", "@zavora-ai/computer-use-mcp"]
    }
  }
}
```

Restart Claude Desktop. Claude will automatically use the tools when asked to interact with your computer.

**Example prompts:**
- *"Take a screenshot and tell me what's on my screen"*
- *"Open Notepad, write a short poem, and save it to the desktop"* (Windows)
- *"Open Safari, go to github.com, and find the trending repositories"* (macOS)
- *"List all virtual desktops and create a new one"*

### Claude API (programmatic)

```typescript
import Anthropic from '@anthropic-ai/sdk'
import { createComputerUseServer } from '@zavora-ai/computer-use-mcp'
import { connectInProcess } from '@zavora-ai/computer-use-mcp/client'

// Start the MCP server in-process
const server = createComputerUseServer()
const mcpClient = await connectInProcess(server)

// List available tools to pass to Claude
const tools = await mcpClient.listTools()

const anthropic = new Anthropic()

// Agent loop
async function runAgent(task: string) {
  const messages: any[] = [{ role: 'user', content: task }]

  while (true) {
    const response = await anthropic.messages.create({
      model: 'claude-opus-4-5',
      max_tokens: 4096,
      tools: tools.map(t => ({
        name: t.name,
        description: t.description,
        input_schema: { type: 'object', properties: {} }
      })),
      messages,
    })

    if (response.stop_reason === 'end_turn') break

    // Execute tool calls
    const toolResults = []
    for (const block of response.content) {
      if (block.type === 'tool_use') {
        const result = await mcpClient.callTool(block.name, block.input as any)
        toolResults.push({
          type: 'tool_result',
          tool_use_id: block.id,
          content: result.content,
        })
      }
    }

    messages.push({ role: 'assistant', content: response.content })
    if (toolResults.length) {
      messages.push({ role: 'user', content: toolResults })
    }
  }

  await mcpClient.close()
}

await runAgent('Open Calculator and compute 123 * 456')
```

## OpenAI Agents SDK

```typescript
import OpenAI from 'openai'
import { createComputerUseServer } from '@zavora-ai/computer-use-mcp'
import { connectInProcess } from '@zavora-ai/computer-use-mcp/client'

const server = createComputerUseServer()
const mcpClient = await connectInProcess(server)
const openai = new OpenAI()

// Wrap MCP tools as OpenAI function tools
const tools = (await mcpClient.listTools()).map(t => ({
  type: 'function' as const,
  function: {
    name: t.name,
    description: t.description ?? '',
    parameters: { type: 'object', properties: {}, additionalProperties: true },
  },
}))

async function runAgent(task: string) {
  const messages: any[] = [{ role: 'user', content: task }]

  while (true) {
    const response = await openai.chat.completions.create({
      model: 'gpt-4o',
      messages,
      tools,
      tool_choice: 'auto',
    })

    const msg = response.choices[0].message
    messages.push(msg)

    if (!msg.tool_calls?.length) break

    for (const call of msg.tool_calls) {
      const args = JSON.parse(call.function.arguments)
      const result = await mcpClient.callTool(call.function.name, args)
      messages.push({
        role: 'tool',
        tool_call_id: call.id,
        content: JSON.stringify(result.content),
      })
    }
  }

  await mcpClient.close()
}

await runAgent('Take a screenshot and describe what you see')
```

## LangChain / LangGraph

```typescript
import { ChatAnthropic } from '@langchain/anthropic'
import { createComputerUseServer } from '@zavora-ai/computer-use-mcp'

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zavora-ai/computer-use-mcp](https://github.com/zavora-ai/computer-use-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
