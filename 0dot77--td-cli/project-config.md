---
trigger: always_on
description: - Commit and push after every meaningful change (new feature, bug fix, refactor)
---

# td-cli Development Guidelines

## Git Workflow
- Commit and push after every meaningful change (new feature, bug fix, refactor)
- Write concise commit messages in English
- Always push to origin/main after committing

## CLI Command Reference (for Claude)

### Workflow — typical agent loop
```bash
td-cli status                              # 1. Check connection
td-cli context --depth 2                   # 2. Get full project summary (tree, families, harness history)
td-cli exec -f scene.py --verify /project1 --screenshot /project1/render1
                                           # 3. Execute + verify + capture preview to .tmp/preview.png
td-cli harness observe /project1 --depth 2 # 4. Deep inspect (graph, data flow, issues)
td-cli harness verify /project1 --assert '[{"kind":"parameterValue","path":"/project1/noise1","name":"roughness","min":0.1}]'
                                           # 5. Assert expected state
td-cli harness rollback <id>               # 6. Undo if needed
```

### Connection & Discovery
| Command | Description |
|---------|-------------|
| `td-cli status` | Check TD connection |
| `td-cli context [--depth N]` | Project summary: tree, families, activity, harness |
| `td-cli instances` | List running TD instances |
| `td-cli describe [path]` | AI-friendly network description |

### Operators
| Command | Description |
|---------|-------------|
| `td-cli ops list [path] [--depth N] [--family TYPE]` | List operators |
| `td-cli ops create <type> <parent> [--name N] [--x X] [--y Y]` | Create operator |
| `td-cli ops delete <path>` | Delete operator |
| `td-cli ops info <path>` | Operator details |
| `td-cli ops rename <path> <new-name>` | Rename |
| `td-cli ops copy <src> <parent>` | Copy |
| `td-cli ops move <src> <parent>` | Move |
| `td-cli ops clone <src> <parent>` | Clone |
| `td-cli ops search <parent> <pattern> [--family TYPE]` | Search |

### Parameters
| Command | Description |
|---------|-------------|
| `td-cli par get <op> [names...]` | Read parameters |
| `td-cli par set <op> <name> <val> [...]` | Set parameters (key-value pairs) |
| `td-cli par pulse <op> <name>` | Pulse button parameter |
| `td-cli par reset <op> [names...]` | Reset to default |
| `td-cli par expr <op> <name> [expression]` | Get/set expression |
| `td-cli par export <op>` | Export all as JSON |
| `td-cli par import <op> <json>` | Import from JSON |

### Connections
| Command | Description |
|---------|-------------|
| `td-cli connect <src> <dst> [--src-index N] [--dst-index N]` | Wire operators |
| `td-cli disconnect <src> <dst>` | Unwire |

### Execution
| Command | Description |
|---------|-------------|
| `td-cli exec "<code>"` | Execute Python in TD |
| `td-cli exec -f <file>` | Execute from file |
| `td-cli exec ... --verify <path>` | + verify node graph after exec |
| `td-cli exec ... --screenshot <path>` | + capture TOP to `.tmp/preview.png` |

### Data Access
| Command | Description |
|---------|-------------|
| `td-cli dat read <path>` | Read DAT content |
| `td-cli dat write <path> <content> [-f file]` | Write DAT |
| `td-cli chop info <path>` | Channel info |
| `td-cli chop channels <path>` | List channels |
| `td-cli chop sample <path> [--channel NAME]` | Sample value |
| `td-cli sop info <path>` | Geometry info |
| `td-cli sop points <path>` | Point data |
| `td-cli pop info <path>` | POP info |
| `td-cli pop points <path> [--attr P]` | POP point data |
| `td-cli pop bounds <path>` | Bounding box |
| `td-cli table rows <path>` | Read rows |
| `td-cli table cell <path> <row> <col> [--value V]` | Read/write cell |

### Visual & Media
| Command | Description |
|---------|-------------|
| `td-cli screenshot [path] [-o file]` | Capture TOP as PNG |
| `td-cli media info <path>` | TOP metadata |
| `td-cli media export <path> <file>` | Export media |
| `td-cli watch [path] [--interval ms]` | Real-time monitor |

### Harness (Agent Loop)
| Command | Description |
|---------|-------------|
| `td-cli harness capabilities` | List supported features |
| `td-cli harness observe [path] [--depth N]` | Capture state snapshot |
| `td-cli harness verify [path] [--assert JSON]` | Run assertions |
| `td-cli harness apply <path> [--goal TEXT] [--op JSON]` | Apply operations with rollback |
| `td-cli harness rollback <id>` | Restore prior state |
| `td-cli harness history [--limit N]` | List iterations |

### Project & Timeline
| Command | Description |
|---------|-------------|
| `td-cli project info` | Project metadata |
| `td-cli project save [path]` | Save project |
| `td-cli timeline [info\|play\|pause]` | Timeline control |
| `td-cli timeline seek <time>` | Jump to frame |
| `td-cli cook node <path>` | Force cook operator |

### Templates & Docs
| Command | Description |
|---------|-------------|
| `td-cli pop av [--root path] [--name NAME]` | Build audio-reactive POP scene |
| `td-cli shaders list [--cat CAT]` | List shader templates |
| `td-cli shaders apply <name> <top>` | Apply shader to GLSL TOP |
| `td-cli docs <operator>` | Offline operator docs |
| `td-cli docs search <keyword>` | Search operators |
| `td-cli docs api [class]` | Python API reference |

### Batch & Network
| Command | Description |
|---------|-------------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [0dot77/td-cli](https://github.com/0dot77/td-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-27 -->
