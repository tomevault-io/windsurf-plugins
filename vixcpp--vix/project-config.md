---
trigger: always_on
description: `vix agent` runs the Vix AI agent from the CLI.
---

# vix agent

`vix agent` runs the Vix AI agent from the CLI.

Use it when you want to ask a local model a question, analyze a project workspace, or scan a workspace before deeper analysis.

```bash
vix agent ask "Explain Vix.cpp in simple words"
```

## Overview

`vix agent` provides AI-assisted development workflows for Vix.cpp.

It supports three commands:

```txt
vix agent ask
vix agent analyze
vix agent scan
```

The agent can:

```txt
answer questions
analyze a local project
scan workspace files
read project files when allowed
use a local model provider such as Ollama
cache responses
persist run history and memory
optionally run safe local commands
```

The default provider comes from the environment or falls back to Ollama.

## Usage

```bash
vix agent ask <prompt> [options]
vix agent analyze [workspace] [prompt] [options]
vix agent scan [workspace] [options]
```

## Basic examples

```bash
vix agent ask "Explain Vix.cpp in simple words"

vix agent ask "Explain Vix.cpp" --timeout 120000

vix agent ask "Explain this code" \
  --model qwen2.5-coder:1.5b \
  --timeout 120000

vix agent analyze .

vix agent scan .

vix agent ask "Run vix tests if useful" --allow-process
```

## Subcommands

| Command             | Purpose                                        |
| ------------------- | ---------------------------------------------- |
| `vix agent ask`     | Ask the agent a direct question.               |
| `vix agent analyze` | Analyze a workspace and explain the project.   |
| `vix agent scan`    | Scan workspace files and print a file summary. |

## `vix agent ask`

Use `ask` for direct prompts.

```bash
vix agent ask "Explain Vix.cpp in simple words"
```

The prompt is required.

Everything after `ask` becomes the input prompt unless it is an option.

Example:

```bash
vix agent ask "What does this project do?"
```

With workspace:

```bash
vix agent ask "Explain this repository" --workspace .
```

With model and timeout:

```bash
vix agent ask "Explain this code" \
  --model qwen2.5-coder:1.5b \
  --timeout 120000
```

## `vix agent analyze`

Use `analyze` when you want the agent to inspect a local project.

```bash
vix agent analyze .
```

If no prompt is given, Vix uses this default analysis prompt:

```txt
Analyze this project and explain the most important parts.
```

You can also pass a custom prompt after the workspace:

```bash
vix agent analyze . "Explain the build system and main modules"
```

Or use `--workspace`:

```bash
vix agent analyze --workspace . "Explain this project architecture"
```

## Analyze mode behavior

Analyze mode gives the agent project-analysis context.

It tells the agent to focus on:

```txt
local C++ project architecture
modules
folders
build system
CLI commands
runtime components
how the pieces fit together
```

It also tells the agent not to invent unrelated technologies.

This makes `vix agent analyze` better suited for real project understanding than a generic chat prompt.

## `vix agent scan`

Use `scan` to inspect which files the agent can see in a workspace.

```bash
vix agent scan .
```

Output includes:

```txt
workspace path
number of files
number of skipped files
whether scan was truncated
file list with sizes
```

Example output shape:

```txt
Scanning agent
  provider: ollama
  model: llama3
  timeout: 120000ms
  workspace: .

agent [============================] done
✔ Scanned workspace

summary:
  workspace: .
  files: 42
  skipped: 8
  truncated: no

files:
  • CMakeLists.txt (1200 bytes)
  • src/main.cpp (900 bytes)
  • include/vix/app.hpp (2400 bytes)
```

`scan` is useful before `analyze` when you want to confirm that the workspace is being read correctly.

## Workspace

The default workspace is:

```txt
.
```

You can set it with:

```bash
vix agent ask "Explain this project" --workspace .
```

or:

```bash
vix agent ask "Explain this project" -w .
```

For `analyze` and `scan`, you can also pass the workspace positionally:

```bash
vix agent analyze .
vix agent scan .
```

## Provider

Set the provider with:

```bash
vix agent ask "Explain Vix.cpp" --provider ollama
```

If not provided, Vix loads the provider from the environment.

Default behavior:

```txt
VIX_AGENT_PROVIDER or ollama
```

## Model

Set the model with:

```bash
vix agent ask "Explain Vix.cpp" --model llama3
```

For a lighter local demo:

```bash
ollama pull qwen2.5-coder:1.5b

vix agent ask "Explain Vix.cpp" \
  --model qwen2.5-coder:1.5b \
  --timeout 120000
```

If not provided, Vix loads the model from the environment.

Default behavior:

```txt
VIX_AGENT_MODEL or llama3
```

## Model URL

Set the model endpoint with:

```bash
vix agent ask "Explain Vix.cpp" \
  --model-url http://127.0.0.1:11434
```

If not provided, Vix loads the endpoint from:

```txt
VIX_AGENT_MODEL_URL
```

When the endpoint is set, Vix prints it in the agent header.

## Timeout

Set the request timeout in milliseconds:

```bash
vix agent ask "Explain Vix.cpp" --timeout 120000
```

The timeout must be a positive integer value.

Examples:

```bash
vix agent ask "Explain this project" --timeout 60000
vix agent analyze . --timeout 180000
vix agent ask "Explain this code" --timeout 300000
```

For slow local CPU models, use a larger timeout:

```bash
vix agent ask "Explain this project" --timeout 300000
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vixcpp/vix](https://github.com/vixcpp/vix) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
