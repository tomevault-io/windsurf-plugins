---
trigger: always_on
description: Guia de referência para o Claude Code trabalhar neste repositório.
---

# Orbit — CLAUDE.md

Guia de referência para o Claude Code trabalhar neste repositório.

---

## O que é o Orbit

Orbit é um **dashboard desktop para gerenciar múltiplas sessões de AI agents em paralelo**, construído com Tauri 2 (Rust + Svelte). Suporta Claude Code, Codex, OpenCode, Gemini CLI e Copilot CLI. Permite criar sessões, acompanhar output em tempo real, visualizar diffs de arquivos, tasks e tokens consumidos. Inclui um **servidor MCP embutido** que permite agentes externos criarem e monitorarem sessões via JSON-RPC.

- Plataformas: **Windows 10 1903+**, **Ubuntu 22.04+** (e outras distros Linux com webkit2gtk 4.1), **macOS** (Intel e Apple Silicon)
- Identificador: `com.josefernando.orbit`
- Repositório: `github.com/xinnaider/orbit`

---

## Stack

| Camada | Tecnologia |
|--------|-----------|
| Desktop framework | Tauri 2.x |
| Backend | Rust 1.85 (MSRV) |
| Frontend | SvelteKit 2.9 + Svelte 5 + TypeScript 5.6 |
| Bundler | Vite 6 (porta 1420) |
| Banco de dados | SQLite via rusqlite 0.31 |
| Testes Rust | cargo test (integração, sem mocks de DB) |
| Testes TS | Vitest 2 |
| Lint TS/Svelte | ESLint 9 + eslint-plugin-svelte |
| Lint Rust | cargo clippy (-D warnings) |
| Format TS/Svelte | Prettier 3.8 + prettier-plugin-svelte |
| Format Rust | rustfmt (tauri/rustfmt.toml) |

---

## Estrutura de diretórios

```
agent-dashboard-v2/
├── tauri/                      # Backend Rust / Tauri
│   ├── src/
│   │   ├── main.rs             # Entry point mínimo
│   │   ├── lib.rs              # Inicialização do app Tauri, plugins, IPC handlers
│   │   ├── models.rs           # Tipos centrais: Session, JournalEntry, AgentStatus, etc.
│   │   ├── commands.rs         # Comandos Tauri auxiliares (diff, tasks, slash commands, files)
│   │   ├── journal_reader.rs   # Parser JSONL do output do Claude Code (incremental)
│   │   ├── agent_tree.rs       # Leitura de metadados de subagentes (.meta.json)
│   │   ├── diff_builder.rs     # Diff de versões de arquivo (Myers LCS)
│   │   ├── mcp_transport.rs     # Servidor IPC local (named pipe / Unix socket) para MCP
│   │   ├── mcp_proxy.rs        # Proxy stdio↔IPC + standalone fallback do orbit-mcp
│   │   ├── ipc/
│   │   │   ├── session.rs      # Comandos Tauri de sessão (create, stop, list, send_message)
│   │   │   ├── mcp.rs          # Handler JSON-RPC MCP com 7 tools (embedded no Tauri)
│   │   │   └── project.rs      # Comandos Tauri de projeto (create, list)
│   │   └── services/
│   │       ├── database.rs     # Wrapper SQLite com migrations automáticas
│   │       ├── session_manager.rs  # Estado em memória + spawn + eventos Tauri
│   │       └── spawn_manager.rs    # Spawn do claude CLI, captura stdout/stderr
│   ├── Cargo.toml
│   ├── tauri.conf.json         # Configuração Tauri (janela 1200×750, segurança)
│   ├── rustfmt.toml            # max_width=100, tab_spaces=4
│   └── .clippy.toml            # cognitive-complexity=30, too-many-lines=100
│
├── ui/                         # Frontend SvelteKit
│   ├── App.svelte              # Raiz: listeners de eventos Tauri, banners globais
│   ├── app.css                 # Estilos globais (variáveis CSS, temas)
│   ├── routes/                 # Rotas SvelteKit (+page.svelte, +layout.svelte)
│   ├── components/             # 23 componentes Svelte
│   └── lib/
│       ├── tauri.ts            # Wrapper IPC com fallback mock
│       ├── types.ts            # Tipos TypeScript (espelho dos models.rs)
│       ├── status.ts           # Helpers de status/cor
│       ├── cost.ts             # Cálculo de custo por tokens
│       └── stores/             # Svelte stores (sessions, journal, preferences, agents)
│
├── .github/workflows/          # CI: lint + build (Windows)
├── CLAUDE.md                   # Este arquivo
├── CHANGELOG.md                # Histórico de alterações (auto-atualizado no pre-commit)
├── package.json
├── vite.config.js
├── svelte.config.js
├── tsconfig.json
├── eslint.config.js
└── .prettierrc
```

---

## Tipos centrais (models.rs)

```
AgentStatus        Working | Input | Idle | New
SessionStatus      Initializing | Running | Waiting | Completed | Stopped | Error
JournalEntryType   User | Thinking | Assistant | ToolCall | ToolResult | System | Progress
Session            id, status, model, pid, cwd, tokens, contextPercent, pendingApproval, miniLog
JournalEntry       sessionId, timestamp, entryType, text, thinking, tool, toolInput, output
TokenUsage         input, output, cacheRead, cacheWrite
MiniLogEntry       tool, target, result, success
TaskItem           id, subject, description, status (pending|in_progress|completed)
```

---

## Banco de dados (SQLite)

Arquivo: `{AppData}/Local/com.josefernando.orbit/agent-dashboard.db`

```sql
projects        (id, name, path UNIQUE, created_at)
sessions        (id, project_id, name, status, permission_mode, model, pid, cwd,
                 claude_session_id, created_at, updated_at)
session_outputs (id, session_id, data TEXT, created_at)
                 -- Armazena JSONL bruto de cada sessão
```

Migrations são aplicadas automaticamente ao iniciar o app (`database.rs`).

---

## Fluxo de sessão (end-to-end)

```
Usuário cria sessão
  ↓
ipc/session.rs::create_session()
  → Fase 1 (síncrona): cria registro no DB, retorna Session imediatamente

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xinnaider/orbit](https://github.com/xinnaider/orbit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
