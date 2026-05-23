---
trigger: always_on
description: https://github.com/punkpeye/fastmcp
---

https://github.com/punkpeye/fastmcp


Skip to content
Navigation Menu
punkpeye
fastmcp

Type / to search
Code
Issues
3
Pull requests
Actions
Security
Insights
Owner avatar
fastmcp
Public
punkpeye/fastmcp
Go to file
t
Name		
punkpeye
punkpeye
fix: gracefully handle failure to shutdown
754ff55
 · 
last week
.github/workflows
fix: correct ci configuration
3 months ago
src
fix: gracefully handle failure to shutdown
last week
.gitignore
release fastmcp
3 months ago
LICENSE
release fastmcp
3 months ago
README.md
fix: rename auth to session
last week
eslint.config.js
release fastmcp
3 months ago
jsr.json
fix: limit included files
3 months ago
package.json
feat: support MCP client authentication
last week
pnpm-lock.yaml
feat: support MCP client authentication
last week
tsconfig.json
chore: lint using tsc
3 months ago
vitest.config.js
feat: allow to retrieve current list of clients
2 months ago
Repository files navigation
README
MIT license
FastMCP
A TypeScript framework for building MCP servers capable of handling client sessions.

Note

For a Python implementation, see FastMCP.

Features
Simple Tool, Resource, Prompt definition
Authentication
Sessions
Image content
Logging
Error handling
SSE
CORS (enabled by default)
Progress notifications
Typed server events
Prompt argument auto-completion
Sampling
Automated SSE pings
Roots
CLI for testing and debugging
Installation
npm install fastmcp
Quickstart
import { FastMCP } from "fastmcp";
import { z } from "zod";

const server = new FastMCP({
  name: "My Server",
  version: "1.0.0",
});

server.addTool({
  name: "add",
  description: "Add two numbers",
  parameters: z.object({
    a: z.number(),
    b: z.number(),
  }),
  execute: async (args) => {
    return String(args.a + args.b);
  },
});

server.start({
  transportType: "stdio",
});
That's it! You have a working MCP server.

You can test the server in terminal with:

git clone https://github.com/punkpeye/fastmcp.git
cd fastmcp

npm install

# Test the addition server example using CLI:
npx fastmcp dev src/examples/addition.ts
# Test the addition server example using MCP Inspector:
npx fastmcp inspect src/examples/addition.ts
SSE
You can also run the server with SSE support:

server.start({
  transportType: "sse",
  sse: {
    endpoint: "/sse",
    port: 8080,
  },
});
This will start the server and listen for SSE connections on http://localhost:8080/sse.

You can then use SSEClientTransport to connect to the server:

import { SSEClientTransport } from "@modelcontextprotocol/sdk/client/sse.js";

const client = new Client(
  {
    name: "example-client",
    version: "1.0.0",
  },
  {
    capabilities: {},
  },
);

const transport = new SSEClientTransport(new URL(`http://localhost:8080/sse`));

await client.connect(transport);
Core Concepts
Tools
Tools in MCP allow servers to expose executable functions that can be invoked by clients and used by LLMs to perform actions.

server.addTool({
  name: "fetch",
  description: "Fetch the content of a url",
  parameters: z.object({
    url: z.string(),
  }),
  execute: async (args) => {
    return await fetchWebpageContent(args.url);
  },
});
Returning a string
execute can return a string:

server.addTool({
  name: "download",
  description: "Download a file",
  parameters: z.object({
    url: z.string(),
  }),
  execute: async (args) => {
    return "Hello, world!";
  },
});
The latter is equivalent to:

server.addTool({
  name: "download",
  description: "Download a file",
  parameters: z.object({
    url: z.string(),
  }),
  execute: async (args) => {
    return {
      content: [
        {
          type: "text",
          text: "Hello, world!",
        },
      ],
    };
  },
});
Returning a list
If you want to return a list of messages, you can return an object with a content property:

server.addTool({
  name: "download",
  description: "Download a file",
  parameters: z.object({
    url: z.string(),
  }),
  execute: async (args) => {
    return {
      content: [
        { type: "text", text: "First message" },
        { type: "text", text: "Second message" },
      ],
    };
  },
});
Returning an image
Use the imageContent to create a content object for an image:

import { imageContent } from "fastmcp";

server.addTool({
  name: "download",
  description: "Download a file",
  parameters: z.object({
    url: z.string(),
  }),
  execute: async (args) => {
    return imageContent({
      url: "https://example.com/image.png",
    });

    // or...
    // return imageContent({
    //   path: "/path/to/image.png",
    // });

    // or...
    // return imageContent({
    //   buffer: Buffer.from("iVBORw0KGgoAAAANSUhEUgAAAAEAAAABCAQAAAC1HAwCAAAAC0lEQVR42mNkYAAAAAYAAjCB0C8AAAAASUVORK5CYII=", "base64"),
    // });

    // or...
    // return {
    //   content: [
    //     await imageContent(...)
    //   ],
    // };
  },
});
The imageContent function takes the following options:

url: The URL of the image.
path: The path to the image file.
buffer: The image data as a buffer.
Only one of url, path, or buffer must be specified.

The above example is equivalent to:

server.addTool({
  name: "download",
  description: "Download a file",
  parameters: z.object({
    url: z.string(),
  }),
  execute: async (args) => {
    return {
      content: [
        {
          type: "image",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Meeting-BaaS/meeting-mcp](https://github.com/Meeting-BaaS/meeting-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
