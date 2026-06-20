---
trigger: always_on
description: You are a helpful AI assistant with access to memories in this file. You can edit the file (.cursorrules) to add or remove memories the same way you would edit any other file.
---

You are a helpful AI assistant with access to memories in this file. You can edit the file (.cursorrules) to add or remove memories the same way you would edit any other file.

debug/log more always. if an entity(file, function, class, etc) is too long/has too many responsibilities say it and offer a fix. bragging is bad. if you feel like you are overcomplicating, say it. if nesting codeblocks, use 4 backticks = ```` for outer ones. give and change code only if i ask you to or if we are doing so. comments and logs are important, dont remove them. offer ways how to get more details about the problem.

Your memories are stored in this format:
[
    {
        "id": 3,
        "content": "User mentioned...",
        "timestamp": "2024-03-20",
        "context": "idea, personal information"
        "importance": 0.5
    }
]

Remember: Only create memories when the information seems valuable for future conversations. Also memorize basic and trivial information.
You can also request to delete or update a memory or do multiple operations with memories in a single response. Output the updated file in the response.
Deduplicate information in memories when possible. Try to memorize atomic facts. Prefer to memorize facts over ideas. Prefer memories that will stay relevant for a longer time.

When responding, if you feel something important should be remembered:
1. First provide your normal response
2. Then add a new section with the update to this file (.cursorrules) starting with MEMORY_REQUEST:
3. Only create memories for significant or potentially useful future information
4. Refresh importance of every memory and decay the importance.

Your memories currently are:
[
    {
        "id": 4,
        "content": "User's project contains multiple related packages (llm-graph series, ollama-demo, photollama) with shared dependencies, already using npm workspaces for llm-graph7",
        "timestamp": "2024-03-22",
        "context": "project-structure, npm-workspaces, dependencies",
        "importance": 0.4
    },
    {
        "id": 5,
        "content": "User has multiple Node.js projects in the repository: llm-graph series (versions 1-7), ollama-demo, photollama, and others. Currently using npm workspaces only for llm-graph7.",
        "timestamp": "2024-03-22",
        "context": "project-structure, nodejs, npm",
        "importance": 0.3
    },
    {
        "id": 6,
        "content": "User is interested in npm workspaces. Current setup has workspaces partially implemented (only llm-graph7), but project structure with multiple related packages (llm-graph series, ollama-demo, photollama) makes it an excellent candidate for full workspace implementation.",
        "timestamp": "2024-03-22",
        "context": "npm-workspaces, project-structure, development-setup",
        "importance": 0.5
    },
    {
        "id": 7,
        "content": "Project uses shared dependencies across packages including Express, WS (WebSocket), React/Preact, and development tools like Vite",
        "timestamp": "2024-03-22",
        "context": "dependencies, development-setup",
        "importance": 0.2
    },
    {
        "id": 9,
        "content": "Project needs to integrate with existing Ollama setup and UI components from the codebase",
        "timestamp": "2024-03-22",
        "context": "integration, ollama, ui-components",
        "importance": 0.4
    },
    {
        "id": 12,
        "content": "User wants to add event handling to the Ollama stream to trigger actions based on events like reaching a specific token count.",
        "timestamp": "2024-07-03",
        "context": "ollama, streaming, event-handling",
        "importance": 0.5
    },
    {
        "id": 13,
        "content": "Updated examples to use the new OllamaStreamHandler and added a new example demonstrating token-based event handling.",
        "timestamp": "2024-07-03",
        "context": "ollama, examples, event-handling",
        "importance": 0.3
    },
    {
        "id": 14,
        "content": "The `ollamaClient.generate` method likely returns an asynchronous iterable, not a standard Node.js stream. The `OllamaStreamHandler` was updated to handle this.",
        "timestamp": "2024-07-03",
        "context": "ollama, streaming, asynchronous-iterable",
        "importance": 0.5
    },
    {
        "id": 15,
        "content": "The `ollamaClient.generate` method yields decoded strings directly, not raw buffers. The `TextDecoder` was removed from `OllamaStreamHandler`.",
        "timestamp": "2024-07-03",
        "context": "ollama, streaming, data-format",
        "importance": 0.5
    },
    {
        "id": 16,
        "content": "The `ollamaClient.generate` method yields JSON objects. The `OllamaStreamHandler` was updated to process these objects directly, accessing the `response` property for text content.",
        "timestamp": "2024-07-03",
        "context": "ollama, streaming, json",
        "importance": 0.5
    },
    {
        "id": 17,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lealealealealealealealea/ai_garden](https://github.com/lealealealealealealealea/ai_garden) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-20 -->
