---
trigger: always_on
description: A Model Context Protocol (MCP) server that gives LM Studio's Qwen3 (or any local LLM) full coding agent capabilities including 80+ tools for file operations, command execution, git, web search, memory, planning, and a full skills system.
---

# Qwen3 MCP Server

A Model Context Protocol (MCP) server that gives LM Studio's Qwen3 (or any local LLM) full coding agent capabilities including 80+ tools for file operations, command execution, git, web search, memory, planning, and a full skills system.

## Two Server Modes

### HTTP Mode (Browser Chat)
```batch
start-chat.bat
# Open http://localhost:3847/chat.html
```

### MCP Mode (LM Studio Direct)
Add to `~/.lmstudio/mcp-servers.json`:
```json
{
  "mcpServers": {
    "qwen3-mcp": {
      "command": "node",
      "args": ["<YOUR_PATH>/src/index.js"],
      "cwd": "<YOUR_PATH>"
    }
  }
}
```

Replace `<YOUR_PATH>` with the actual installation path.

## Project Structure

```
qwen3-mcp/
├── src/
│   ├── index.js           # MCP server (stdio, uses @modelcontextprotocol/sdk)
│   └── tools/             # Tool implementations
│       ├── filesystem.js  # read_file, write_file, list_directory, etc.
│       ├── edit.js        # edit_file, insert_at_line, replace_lines
│       ├── bash.js        # execute_command, execute_background
│       ├── git.js         # git_status, git_commit, git_push, etc.
│       ├── search.js      # glob_search, grep_search, find_definition
│       ├── web.js         # web_search, web_image_search, web_fetch
│       ├── memory.js      # memory_store, memory_recall, scratchpad_*
│       ├── planning.js    # plan_create, plan_status, plan_step_complete
│       ├── tasks.js       # task_add, task_list, task_update
│       ├── thinking.js    # think, reason, evaluate_options
│       ├── context.js     # conversation_log, conversation_context
│       ├── interaction.js # ask_user, confirm, present_choices
│       ├── media.js       # read_image, read_pdf, take_screenshot
│       ├── notebook.js    # notebook_read, notebook_edit_cell
│       ├── comfyui.js     # 32 ComfyUI workflow tools
│       ├── github-blog.js # Jekyll blog tools
│       └── skills.js      # list_skills, load_skill, install_skill
├── frontend/
│   ├── server.js          # HTTP server (port 3847) with all tools
│   ├── chat.html          # Browser chat interface
│   └── index.html         # Image viewer interface
├── docs/                  # GitHub Pages static site (auto-generated)
│   ├── index.html         # Built from README.md by build-docs.cjs
│   ├── .nojekyll          # Tells GitHub Pages to serve as static HTML
│   └── assets/
│       ├── css/style.css  # Dark theme matching marduk191.github.io
│       ├── js/main.js     # Mobile menu toggle
│       └── images/banner.jpg  # Hero banner image
├── scripts/
│   └── build-docs.cjs     # Converts README.md → docs/index.html
├── .github/
│   └── workflows/
│       └── build-docs.yml # Auto-rebuilds docs on README.md changes
├── skills/                # Installed agent skills (16)
│   ├── chrome-extension/  # Chrome extension development (MV3)
│   ├── code-review/       # Code review methodology
│   ├── comfyui-nodes/     # ComfyUI custom node development (V1 + V3 API, v2.1.0)
│   ├── comfyui-workflow/  # ComfyUI workflow creation (SD1.5/SDXL/SD3.5/Flux)
│   ├── differential-review/ # Security-focused diff review
│   ├── docx/              # Word document creation (Anthropic)
│   ├── frontend-design/   # Frontend UI/UX
│   ├── github-blog/       # Jekyll blog for GitHub Pages
│   ├── mcp-builder/       # Build MCP servers
│   ├── modern-python/     # Python tooling (Trail of Bits)
│   ├── react-best-practices/ # React patterns (Vercel)
│   ├── shadcn-ui/         # Modern component library
│   ├── static-analysis/   # CodeQL, Semgrep, SARIF
│   ├── testing-handbook-skills/ # Fuzzers, sanitizers
│   ├── web-artifacts-builder/ # HTML/React prototypes
│   └── web-design-guidelines/ # UI/UX fundamentals
├── start-chat.bat         # Start HTTP server
├── stop-chat.bat          # Stop server
├── restart-chat.bat       # Restart server
└── install-skill.bat      # Install skills from GitHub
```

## Available Tools (80+)

### File Operations
| Tool | Description |
|------|-------------|
| `read_file` | Read file contents with line numbers (params: `file_path`, `offset`, `limit`) |
| `write_file` | Write/create files (params: `file_path`, `content`) |
| `edit_file` | Find and replace text in files (params: `file_path`, `old_string`, `new_string`) |
| `list_directory` | List directory contents |
| `create_directory` | Create directories (recursive) |
| `delete_file` | Delete files |
| `move_file` | Move/rename files |
| `copy_file` | Copy files |
| `file_info` | Get file metadata (size, dates) |
| `get_working_directory` | Get current working directory |
| `set_working_directory` | Set working directory |

### Edit Tools
| Tool | Description |
|------|-------------|
| `edit_file` | Find/replace text |
| `insert_at_line` | Insert at specific line |
| `replace_lines` | Replace line range |
| `append_to_file` | Append to file |
| `prepend_to_file` | Prepend to file |

### Search Tools
| Tool | Description |
|------|-------------|
| `glob_search` | Find files by pattern (params: `pattern`, `cwd`) |
| `grep_search` | Search file contents with regex (params: `pattern`, `path`) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marduk191/qwen3_mcp](https://github.com/marduk191/qwen3_mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
