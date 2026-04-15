---
trigger: always_on
description: When working on complex tasks, ALWAYS use MCP tools like Sequential Thinking to break down the problem and plan the implementation. This helps maintain clarity and ensures thorough consideration of all aspects.
---

# ALWAYS USE MCP TOOLS

When working on complex tasks, ALWAYS use MCP tools like Sequential Thinking to break down the problem and plan the implementation. This helps maintain clarity and ensures thorough consideration of all aspects.

# Implementation Plan for Offline Wikipedia System

## Phase Order
1. Error Handling & Logging Framework (Tool 9) - ✅
2. Data Ingestion & Parsing (Tool 1) - ✅
3. Database & Indexing (Tool 2) - ✅
4. Vector Embedding (Tool 3) - ✅
5. LLM Integration (Tool 4) - ✅
6. Installation Manager (Tool 5) - ✅
7. UI/Frontend (Tool 6) - ✅
8. Testing & Documentation (Tools 7,8) - ✅

## Technical Stack
- Backend: Rust
- Database: SQLite + FTS5
- Vector Store: LMDB
- LLM: Ollama
- UI: Flutter
- Async: Tokio
- Logging: Tracing

## Project Structure
```
/src
  /error_handling - Tool 9
  /parser - Tool 1
  /db - Tool 2
  /vector - Tool 3
  /llm - Tool 4
  /installer - Tool 5
  /ui - Tool 6
  /tests - Tool 7
  /docs - Tool 8
```

## Performance Requirements
- Search latency: < 500ms
- Article load time: < 1 sec
- LLM response: < 3 sec
- Initial load: < 5 sec
- Memory usage: < 2GB

## Dependencies
- Rust (latest stable)
- SQLite 3.35.0+
- LMDB 0.9+
- Ollama (latest)
- Flutter 3.0+
- Tokio 1.0+

## Next Steps
1. Performance optimization
2. User experience improvements
3. Deployment and distribution

# Instructions

During your interaction with the user, if you find anything reusable in this project (e.g. version of a library, model name), especially about a fix to a mistake you made or a correction you received, you should take note in the `Lessons` section in the `.cursorrules` file so you will not make the same mistake again. 

You should also use the `.cursorrules` file as a Scratchpad to organize your thoughts. Especially when you receive a new task, you should first review the content of the Scratchpad, clear old different task if necessary, first explain the task, and plan the steps you need to take to complete the task. You can use todo markers to indicate the progress, e.g.
[X] Task 1
[ ] Task 2

Also update the progress of the task in the Scratchpad when you finish a subtask.
Especially when you finished a milestone, it will help to improve your depth of task accomplishment to use the Scratchpad to reflect and plan.
The goal is to help you maintain a big picture as well as the progress of the task. Always refer to the Scratchpad when you plan the next step.

# Tools

Note all the tools are in python3. So in the case you need to do batch processing, you can always consult the python files and write your own script.

## Screenshot Verification

The screenshot verification workflow allows you to capture screenshots of web pages and verify their appearance using LLMs. The following tools are available:

1. Screenshot Capture:
```bash
venv/bin/python3 tools/screenshot_utils.py URL [--output OUTPUT] [--width WIDTH] [--height HEIGHT]
```

2. LLM Verification with Images:
```bash
venv/bin/python3 tools/llm_api.py --prompt "Your verification question" --provider {openai|anthropic} --image path/to/screenshot.png
```

Example workflow:
```python
from screenshot_utils import take_screenshot_sync
from llm_api import query_llm

# Take a screenshot

screenshot_path = take_screenshot_sync('https://example.com', 'screenshot.png')

# Verify with LLM

response = query_llm(
    "What is the background color and title of this webpage?",
    provider="openai",  # or "anthropic"
    image_path=screenshot_path
)
print(response)
```

## LLM

You always have an LLM at your side to help you with the task. For simple tasks, you could invoke the LLM by running the following command:
```
venv/bin/python3 ./tools/llm_api.py --prompt "What is the capital of France?" --provider "anthropic"
```

The LLM API supports multiple providers:
- OpenAI (default, model: gpt-4o)
- Azure OpenAI (model: configured via AZURE_OPENAI_MODEL_DEPLOYMENT in .env file, defaults to gpt-4o-ms)
- DeepSeek (model: deepseek-chat)
- Anthropic (model: claude-3-sonnet-20240229)
- Gemini (model: gemini-pro)
- Local LLM (model: Qwen/Qwen2.5-32B-Instruct-AWQ)

But usually it's a better idea to check the content of the file and use the APIs in the `tools/llm_api.py` file to invoke the LLM if needed.

## Web browser

You could use the `tools/web_scraper.py` file to scrape the web.
```bash
venv/bin/python3 ./tools/web_scraper.py --max-concurrent 3 URL1 URL2 URL3
```
This will output the content of the web pages.

## Search engine

You could use the `tools/search_engine.py` file to search the web.
```bash
venv/bin/python3 ./tools/search_engine.py "your search keywords"
```
This will output the search results in the following format:
```
URL: https://example.com
Title: This is the title of the search result
Snippet: This is a snippet of the search result
```
If needed, you can further use the `web_scraper.py` file to scrape the web page content.

# Lessons

## User Specified Lessons

- You have a python3 venv in ./venv. Use it.
- Include info useful for debugging in the program output.
- Read the file before you try to edit it.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/rayman546) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
