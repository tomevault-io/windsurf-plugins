---
trigger: always_on
description: This guide shows how to integrate AI agents with AIO Sandbox, leveraging the Model Context Protocol (MCP) and REST APIs for powerful agent workflows.
---

# Agent Integration

This guide shows how to integrate AI agents with AIO Sandbox, leveraging the Model Context Protocol (MCP) and REST APIs for powerful agent workflows.

## Quick Start

### Basic Agent Setup

The simplest way to connect an agent to AIO Sandbox is through the MCP interface:

```python
import asyncio
import aiohttp
from typing import Dict, Any

class AIOSandboxAgent:
    def __init__(self, base_url: str = "http://localhost:8080"):
        self.base_url = base_url
        self.session = None

    async def __aenter__(self):
        self.session = aiohttp.ClientSession()
        return self

    async def __aexit__(self, exc_type, exc_val, exc_tb):
        if self.session:
            await self.session.close()

    async def execute_shell(self, command: str) -> Dict[str, Any]:
        """Execute shell command in sandbox"""
        url = f"{self.base_url}/v1/shell/exec"
        payload = {"command": command}

        async with self.session.post(url, json=payload) as response:
            return await response.json()

    async def read_file(self, file_path: str) -> Dict[str, Any]:
        """Read file content from sandbox"""
        url = f"{self.base_url}/v1/file/read"
        payload = {"file": file_path}

        async with self.session.post(url, json=payload) as response:
            return await response.json()

    async def write_file(self, file_path: str, content: str) -> Dict[str, Any]:
        """Write file content to sandbox"""
        url = f"{self.base_url}/v1/file/write"
        payload = {"file": file_path, "content": content}

        async with self.session.post(url, json=payload) as response:
            return await response.json()

# Usage example
async def main():
    async with AIOSandboxAgent() as agent:
        # Execute a command
        result = await agent.execute_shell("python --version")
        print(f"Python version: {result}")

        # Create and read a file
        await agent.write_file("/tmp/hello.py", "print('Hello from agent!')")
        await agent.execute_shell("python /tmp/hello.py")

if __name__ == "__main__":
    asyncio.run(main())
```

### MCP Integration

AIO Sandbox provides built-in MCP servers for seamless agent integration:

```python
import json
import websockets
from typing import Dict, List

class MCPClient:
    def __init__(self, mcp_url: str = "ws://localhost:8080/mcp"):
        self.mcp_url = mcp_url
        self.websocket = None

    async def connect(self):
        """Connect to MCP WebSocket interface"""
        self.websocket = await websockets.connect(self.mcp_url)

    async def list_servers(self) -> List[str]:
        """List available MCP servers"""
        message = {
            "jsonrpc": "2.0",
            "method": "servers/list",
            "id": 1
        }
        await self.websocket.send(json.dumps(message))
        response = await self.websocket.recv()
        return json.loads(response)

    async def list_tools(self, server_name: str) -> Dict:
        """List tools available in a specific server"""
        message = {
            "jsonrpc": "2.0",
            "method": "tools/list",
            "params": {"server": server_name},
            "id": 2
        }
        await self.websocket.send(json.dumps(message))
        response = await self.websocket.recv()
        return json.loads(response)

    async def call_tool(self, server_name: str, tool_name: str, arguments: Dict) -> Dict:
        """Execute a tool on a specific MCP server"""
        message = {
            "jsonrpc": "2.0",
            "method": "tools/call",
            "params": {
                "server": server_name,
                "name": tool_name,
                "arguments": arguments
            },
            "id": 3
        }
        await self.websocket.send(json.dumps(message))
        response = await self.websocket.recv()
        return json.loads(response)

# Usage example
async def mcp_example():
    client = MCPClient()
    await client.connect()

    # List available servers
    servers = await client.list_servers()
    print(f"Available servers: {servers}")

    # Use the browser server to navigate
    await client.call_tool("browser", "navigate", {
        "url": "https://example.com"
    })

    # Use the file server to create a file
    await client.call_tool("file", "write", {
        "path": "/tmp/scraped_data.html",
        "content": "<html>...</html>"
    })
```

## Advanced Workflows

### Multi-Tool Agent Workflow

Here's an example of a complex agent that combines browser automation, file operations, and code execution:

```python
class WebScrapingAgent(AIOSandboxAgent):
    async def scrape_and_analyze(self, url: str, analysis_script: str):
        """Complete workflow: scrape website, save data, and analyze"""

        # Step 1: Navigate to website and extract content
        await self.navigate_browser(url)
        content = await self.extract_page_content()

        # Step 2: Save content to file
        data_file = "/tmp/scraped_data.html"
        await self.write_file(data_file, content)

        # Step 3: Create analysis script
        script_file = "/tmp/analyze.py"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [agent-infra/sandbox](https://github.com/agent-infra/sandbox) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
