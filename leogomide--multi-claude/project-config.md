---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**multi-claude** — CLI tool para gerenciar multiplos provedores de API e iniciar o Claude Code com as environment variables corretas. O usuario roda `mclaude` para selecionar um provedor (o gerenciamento de providers e feito dentro da TUI).

## Commands

- **Install dependencies:** `bun install`
- **Run:** `mclaude` (apos `bun link`)
- **Type check:** `bunx tsc --noEmit`

## Project Structure

```
cli.ts                  # Entry point da CLI (bin: mclaude)
src/
├── schema.ts           # Schemas Zod e tipos TypeScript
├── providers.ts        # Templates dos provedores suportados
├── config.ts           # Leitura/escrita de ~/.multi-claude/config.json
├── runner.ts           # Spawn do claude com env vars
├── tui-process.ts      # Processo da TUI (setup e resultado)
├── app.tsx             # Render do UnifiedApp com Ink
├── debug.ts            # Utilitarios de debug
├── headless.ts         # Modo headless (non-interactive CLI)
├── credential-store.ts # Gerenciamento de credenciais encriptadas
├── crypto.ts           # Operacoes criptograficas (AES-256-GCM)
├── keystore.ts         # Gerenciamento de chaves de encriptacao
├── statusline.ts       # Renderizacao da status line do Claude Code
├── logs-viewer.ts      # Visualizador de logs de debug
├── services/
│   ├── api-models.ts   # Fetch de modelos de APIs externas
│   ├── openrouter.ts   # Integracao OpenRouter
│   ├── requesty.ts     # Integracao Requesty
│   ├── ollama.ts       # Integracao Ollama
│   ├── lmstudio.ts     # Integracao LM Studio
│   ├── llamacpp.ts     # Integracao llama.cpp
│   ├── litellm.ts      # Integracao LiteLLM Proxy
│   ├── nanogpt.ts      # Integracao NanoGPT
│   └── version-check.ts # Verificacao de atualizacoes
├── i18n/
│   ├── index.ts        # Setup do i18n (rosetta)
│   ├── types.ts        # Tipos das traducoes
│   ├── context.tsx     # Context provider do i18n (React)
│   └── locales/
│       ├── en.ts       # Ingles
│       ├── pt-BR.ts    # Portugues (BR)
│       └── es.ts       # Espanhol
├── hooks/
│   ├── useTerminalSize.ts  # Hook de tamanho do terminal
│   ├── useBreadcrumb.tsx   # Hook de breadcrumbs para navegacao
│   └── useUpdateCheck.ts   # Hook de verificacao de atualizacoes
├── utils/
│   └── win32-console-size.ts # Deteccao de tamanho do console no Windows
└── components/
    ├── types.ts             # Tipos compartilhados dos componentes
    ├── common/
    │   ├── StatusMessage.tsx    # Mensagens de status com icone e cor
    │   ├── Note.tsx             # Box decorado com titulo e conteudo
    │   ├── ConfirmPrompt.tsx    # Prompt de confirmacao Yes/No
    │   ├── TextPrompt.tsx       # Input de texto com validacao e mask
    │   ├── GroupedSelect.tsx    # Select com grupos e sidebar
    │   ├── SearchableSelect.tsx # Select com busca
    │   ├── ChecklistSelect.tsx  # Select com checkboxes (multi-selecao)
    │   ├── CyanSelectInput.tsx  # Select estilizado com tema cyan
    │   └── LanguageSelector.tsx # Seletor de idioma
    ├── layout/
    │   ├── AppShell.tsx    # Shell principal (header + content + footer)
    │   ├── Header.tsx      # Header com titulo e versao
    │   ├── Footer.tsx      # Footer com breadcrumbs e atalhos
    │   └── Sidebar.tsx     # Sidebar com informacoes contextuais
    ├── app/
    │   ├── UnifiedApp.tsx          # Router principal da aplicacao
    │   ├── MainMenu.tsx            # Menu principal
    │   ├── StartClaudeFlow.tsx     # Fluxo: provider -> modelo -> instalacao -> launch
    │   ├── ManageProvidersPage.tsx  # Pagina de gerenciamento de providers
    │   ├── ManageInstallationsPage.tsx # Pagina de gerenciamento de instalacoes
    │   ├── SettingsPage.tsx        # Pagina de configuracoes
    │   └── StatusLinePage.tsx      # Pagina de configuracao da status line
    └── config-wizard/
        ├── AddProviderFlow.tsx     # Fluxo: template -> nome -> api key
        ├── EditProviderFlow.tsx    # Fluxo: selecionar -> editar
        ├── AddInstallationFlow.tsx # Fluxo: nome -> criar diretorio
        ├── EditInstallationFlow.tsx # Fluxo: renomear / remover
        └── ManageModelsFlow.tsx    # Fluxo: gerenciar modelos
```

## Key Dependencies

- **zod** — validacao de schemas de configuracao
- **ink** — React para terminal (UI declarativa)
- **react** — renderizacao de componentes
- **ink-text-input** — input de texto para terminal
- **ink-select-input** — menu de selecao para terminal
- **@inkjs/ui** — componentes UI adicionais para Ink

## Config Storage

Configuracoes sao salvas em `~/.multi-claude/config.json`.

## Tech Stack

- **Runtime:** Bun (not Node.js)
- **Language:** TypeScript with strict mode enabled
- **Module system:** ESNext with bundler module resolution (`noEmit: true`, no build step — Bun runs `.ts` directly)

## Protected Files

**NEVER modify the following files:**

- `TODO.md` - Developer's personal notes and task tracking file

## Release Process

Checklist completo para lançar uma nova versão:

### 1. Validação

```bash
bunx tsc --noEmit
bun test
```

### 2. Bump de versão


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [leogomide/multi-claude](https://github.com/leogomide/multi-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
