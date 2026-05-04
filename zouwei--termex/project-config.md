---
trigger: always_on
description: > Terminal + Experience | 以 SSH 协议为底座，打造 AI 时代永不断线的云端智能工作平台
---

# Termex - Claude Code Rules

> Terminal + Experience | 以 SSH 协议为底座，打造 AI 时代永不断线的云端智能工作平台

## Project Overview

Termex is an open-source, AI-native SSH client built with Tauri v2 (Rust backend) + Vue 3 (TypeScript frontend). Built on SSH as the foundation, it creates an always-on cloud AI workspace for the AI era. It targets developers and ops engineers who need a beautiful, fast, intelligent, and free SSH client.

## Tech Stack

- **Runtime**: Tauri v2
- **Backend**: Rust (russh, ring, SQLCipher, tokio)
- **Frontend**: Vue 3 + Vite + TypeScript
- **UI**: Element Plus + Tailwind CSS
- **Terminal**: xterm.js (WebGL renderer)
- **State**: Pinia
- **Database**: SQLite + SQLCipher (encrypted)

## Architecture

```
src-tauri/src/           # Rust backend
├── commands/            # Tauri IPC command handlers
├── ssh/                 # SSH protocol (russh)
├── sftp/                # SFTP operations
├── crypto/              # AES-256-GCM encryption, Argon2id KDF
├── storage/             # SQLCipher database
├── ai/                  # AI provider abstraction (Claude/OpenAI/Ollama)
└── state.rs             # Global AppState

src/                     # Vue 3 frontend
├── components/          # Vue components (sidebar/, terminal/, ai/, sftp/, settings/)
├── composables/         # Composition API hooks (useSSH, useTerminal, useAi, etc.)
├── stores/              # Pinia stores (serverStore, sessionStore, settingsStore, aiStore)
├── types/               # TypeScript type definitions
└── utils/               # Utility functions
```

## Code Conventions

### Rust (src-tauri/)

- All Tauri commands are in `src/commands/`, one file per module
- Business logic lives in dedicated modules (`ssh/`, `crypto/`, `ai/`), NOT in command handlers
- Command handlers are thin wrappers: validate input → call module → return result
- Use `thiserror` for error types, map to `String` at the command boundary
- All async operations use `tokio`
- Never log passwords, API keys, or any sensitive data
- Use `uuid::Uuid::new_v4()` for all IDs
- Database timestamps use ISO 8601 format (`chrono::Utc::now().to_rfc3339()`)
- Encrypted fields (passwords, API keys) use the `_enc` suffix and store as `BLOB`
- All public functions must have doc comments

### TypeScript / Vue (src/)

- Use `<script setup lang="ts">` for all Vue components
- Use Composition API exclusively, no Options API
- Component naming: PascalCase files, multi-word names (e.g., `ServerItem.vue`, not `Item.vue`)
- All Tauri IPC calls go through `src/utils/tauri.ts` wrapper functions
- Type definitions in `src/types/`, one file per domain
- Pinia stores in `src/stores/`, one file per store
- Composables in `src/composables/`, prefixed with `use`
- CSS: use Tailwind utility classes first, Element Plus components for complex widgets, custom CSS only when necessary
- No inline styles except for dynamic values (e.g., `style="width: ${w}px"`)
- Event naming: kebab-case for template events, camelCase for emits

### General

- Commit messages: `<type>(<scope>): <description>` (e.g., `feat(ssh): add RSA key authentication`)
- Types: feat, fix, refactor, style, docs, test, chore
- Scopes: ssh, sftp, ui, ai, crypto, storage, config
- No `any` type in TypeScript — define proper interfaces
- No `unwrap()` in Rust production code — use `?` operator or proper error handling
- No hardcoded strings for user-facing text — prepare for i18n
- **单文件行数上限 800 行** — 超过时必须按职责拆分为多个文件。Rust 用 `mod` 子模块拆分，Vue 用子组件 + composable 拆分，TypeScript 按领域拆分到独立文件。宁可多文件也不要单文件臃肿
- **实现与测试必须同步** — 每个功能模块在实现的同时必须编写对应的单元测试/集成测试。不允许先实现后补测试，也不允许提交无测试覆盖的功能代码
- **测试代码独立存放** — 所有测试用例必须放在独立的 `tests/` 目录下，按模块分类组织，禁止在业务代码文件内编写 `#[cfg(test)] mod tests`。Rust 测试放在 `src-tauri/tests/`，前端测试使用 Vitest 放在 `src/__tests__/`

## Security Rules (CRITICAL)

- **NEVER** log sensitive data (passwords, keys, tokens) at any log level
- **NEVER** send credentials to AI providers — only send command text and server metadata (OS, hostname)
- **NEVER** include secrets in error messages returned to frontend

### Credential Storage Strategy (v0.10.0+)

- **Primary: OS Keychain** — SSH passwords, passphrases, AI API keys stored via `keyring` crate
  - macOS: Keychain Services (Secure Enclave)
  - Windows: Credential Manager (DPAPI)
  - Linux: Secret Service (GNOME Keyring / KDE Wallet)
- **termex.db stores only keychain reference IDs** (e.g., `termex:ssh:password:{uuid}`), never actual credentials
- **Fallback: AES-256-GCM + Master Password** — when OS keychain is unavailable (headless Linux)
  - Encryption uses `ring` crate with AES-256-GCM
  - Key derivation uses Argon2id (m=64MB, t=3, p=4)
  - Master key exists only in memory, zeroed on app lock
- Export files use independent password + salt, decoupled from keychain/master password

### Keychain Single-Prompt Rule (CRITICAL)

- **OS keychain 每次启动最多弹出 1 次密码提示** — 这是硬性要求，违反会严重影响用户体验
- 所有凭据存储在**单一 keychain entry** (`__termex_store__`) 中，以 JSON 对象形式存储
- **只有 `init()` 可以调用 `get_password()`**（读取 OS keychain）
- **只有 `flush()` 可以调用 `set_password()`**（写入 OS keychain）
- **其他任何函数禁止直接访问 `keyring::Entry`** — 一律通过内存缓存 `get()`/`store()`
- 禁止创建额外的 keychain entry（无 per-server、per-provider、verification token 条目）
- `verify_accessible()` 仅检查 `is_available()` 返回值，不执行额外的 keychain 读取

## IPC Conventions


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zouwei/termex](https://github.com/zouwei/termex) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
