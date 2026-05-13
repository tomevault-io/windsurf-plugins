---
trigger: always_on
description: > Arquivo gerado por `/init` com análise automática. Edite manualmente para ajustar convenções.
---

# AGENTS.md

> Arquivo gerado por `/init` com análise automática. Edite manualmente para ajustar convenções.

## Projeto

- **Nome:** SSH Config Sync (`ssh-orchestrator`)
- **Descrição:** Cliente SSH/SFTP cross-platform construído com Tauri v2, oferecendo sincronização seletiva de workspaces via repositórios GitHub privados com resolução de conflitos baseada em CRDTs (LWW-Register com HLC).

## Stack

- **Linguagem(s):** TypeScript 5.8 (frontend) · Rust 2021 / 1.77+ (backend)
- **Frameworks:** React 19, TailwindCSS 3, Vite 7, Tauri v2, Tokio, SQLx 0.7, russh 0.57, git2 0.20, portable-pty 0.8

## Gerenciamento de Dependências

> **`pnpm` only** — nunca usar `npm` ou `yarn`.

- **Instalar tudo:** `pnpm install`
- **Adicionar pacote (frontend):** `pnpm add <pacote>`
- **Remover pacote (frontend):** `pnpm remove <pacote>`
- **Adicionar crate (backend):** editar `src-tauri/Cargo.toml` e rodar `cd src-tauri && cargo build`

## Comandos Essenciais

- **Dev server (full-stack):** `pnpm tauri dev`
- **Dev server (frontend only):** `pnpm dev`
- **Build:** `pnpm build`
- **Preview:** `pnpm preview`
- **Testes (Rust):** `cd src-tauri && cargo test`
- **Lint (Rust):** `cd src-tauri && cargo clippy`
- **Formato (Rust):** `cd src-tauri && cargo fmt`
- **Type-check (TS):** `tsc` (embutido em `pnpm build`)

## Estrutura de Diretórios

- **Código principal (frontend):** `src/`
- **Código principal (backend):** `src-tauri/src/`
- **Testes:** módulos `#[cfg(test)]` inline em `src-tauri/src/sync/crdt.rs` e `src-tauri/src/services/crypto.rs` — não há diretório `tests/` separado

## Módulos

### Frontend — `src/`

- **`src/components/Terminal/`** — Emulador de terminal SSH remoto e PTY local com tabs, split-pane e temas (xterm.js): `Terminal.tsx`, `LocalTerminal.tsx`, `TerminalWorkspace.tsx`, `TerminalTabBar.tsx`, `ServerPickerModal.tsx`
- **`src/components/Sftp/`** — Gerenciador de arquivos dual-pane (remoto ↔ local) via SFTP com drag & drop: `SftpDualPane.tsx`, `SftpPanel.tsx`, `TransferQueue.tsx`
- **`src/components/Servers/`** — Modal de criação/edição de servidores SSH (`AddServerModal.tsx`)
- **`src/components/Workspaces/`** — Tela de detalhes de workspace com listagem e gerenciamento de servidores (`WorkspaceDetail.tsx`)
- **`src/components/sync/`** — Indicador de progresso da sincronização de workspace (`SyncStatus.tsx`)
- **`src/components/Sidebar.tsx`** — Barra lateral com lista de workspaces e navegação global
- **`src/components/TitleBar.tsx`** — Barra de título customizada com controles de janela e ações globais
- **`src/components/VaultGuard.tsx`** — Guarda de autenticação do vault; exige master password antes de renderizar a app
- **`src/components/Modal.tsx`** — Componente base de modal reutilizável com animações
- **`src/components/Toast.tsx`** — Container de notificações toast
- **`src/hooks/useTerminalManager.ts`** — Gerencia o estado de todas as tabs (SSH, SFTP, local) e split-pane
- **`src/hooks/useTerminalTheme.ts`** — Seleção e persistência do tema de terminal ativo
- **`src/hooks/useToast.tsx`** — Provider e hook de notificações toast globais
- **`src/hooks/useAuth.ts`** — Gerencia estado de autenticação GitHub OAuth
- **`src/hooks/useSftpQueue.ts`** — Fila de transferências SFTP com progresso e cancelamento
- **`src/lib/api/servers.ts`** — Wrappers de `invoke()` para CRUD de servidores
- **`src/lib/api/workspaces.ts`** — Wrappers de `invoke()` para CRUD de workspaces e push/pull de sync
- **`src/lib/api/ssh.ts`** — Wrappers de `invoke()` para conectar, escrever, redimensionar e desconectar sessões SSH
- **`src/lib/api/sftp.ts`** — Wrappers de `invoke()` para todas as operações SFTP (upload, download, list, delete, rename, mkdir)
- **`src/lib/api/pty.ts`** — Wrappers de `invoke()` para spawn, write, resize e kill de PTY local
- **`src/lib/api/vault.ts`** — Wrappers de `invoke()` para setup, unlock e verificação do vault
- **`src/lib/api/auth.ts`** — Wrappers de `invoke()` para login/logout GitHub e obtenção do usuário atual
- **`src/lib/keybindings.ts`** — Mapa de atalhos de teclado (`KEYBINDINGS`) e helper `matchesBinding`
- **`src/lib/themes.ts`** — Definição de temas de terminal disponíveis

### Backend — `src-tauri/src/`

- **`handlers/`** — Funções IPC `#[tauri::command]`: entry points para workspace, server, ssh, sftp, vault, auth e pty
- **`services/crypto.rs`** — Criptografia AES-256-GCM + PBKDF2 via `ring`; gerencia VaultState (Locked/Unlocked/Unconfigured); testes inline `#[cfg(test)]`
- **`services/db.rs`** — Pool SQLite via sqlx; criação de schema e migrations inline
- **`services/ssh.rs`** — Engine SSH usando russh; gestão de sessões via `DashMap`
- **`services/sftp.rs`** — Operações SFTP sobre sessões SSH existentes
- **`services/pty.rs`** — Terminal PTY local via portable-pty; gestão de processos por sessão
- **`models/mod.rs`** — Structs compartilhadas: `Workspace`, `Server`, `ServerRow`; todas com `Serialize`/`Deserialize`/`sqlx::FromRow`
- **`models/vault.rs`** — Structs do vault (`VaultConfig`, etc.)
- **`sync/crdt.rs`** — Implementação do HLC (Hybrid Logical Clock) e LWW merge; `get_or_create_node_id`; testes inline `#[cfg(test)]`
- **`sync/git_ops.rs`** — Operações git de baixo nível (clone, commit, push, pull) via git2

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcos2872/SSH_Orchestrator](https://github.com/marcos2872/SSH_Orchestrator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-12 -->
