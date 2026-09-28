---
trigger: always_on
description: Para qualquer feature, refactor ou alteração na base de código (exceto tarefas triviais/chores), seguir obrigatoriamente o processo definido em:
---

# Agents Code Instructions

## Base Prompt

Para qualquer feature, refactor ou alteração na base de código (exceto tarefas triviais/chores), seguir obrigatoriamente o processo definido em:

**[docs/agents/prompts/base.prompt.md](docs/agents/prompts/base.prompt.md)**

### Resumo do Processo (TDD)

1. Analisar se a feature/alteração faz sentido com a base atual
2. Escrever teste unitário primeiro (TDD) - deixar falhar
3. Implementar seguindo clean code, atomic design, estrutura existente
4. Rodar testes até passar
5. Atualizar README/documentação se necessário
6. Registrar no CHANGELOG (exceto triviais)
7. Revisão geral antes de submeter

---

## Arquitetura do Projeto

### Tech Stack

- **VS Code Extension:** TypeScript, VS Code Extension API
- **Webview:** React 19 + Vite + Tailwind CSS
- **Testes:** Vitest + Testing Library
- **Design:** Atomic Design para componentes
- **Build:** Vite (extension + webview separados)

### Estrutura de Diretórios

```text
markdown-kanban/
├── src/
│   ├── extension.ts              # Entry point da extensão VS Code
│   ├── kanbanWebviewPanel.ts     # Gerenciador do painel webview + comunicação
│   ├── kanbanTreeProvider.ts     # Sidebar tree view provider
│   ├── markdownParser.ts         # Parser bidirecional markdown ↔ board
│   ├── templates/
│   │   └── kanbanTemplate.ts     # Template padrão de novo board
│   └── webview/
│       ├── App.tsx               # Componente React root
│       ├── main.tsx              # Entry point do webview
│       ├── hooks/
│       │   ├── useVSCodeApi.ts   # Wrapper da API de mensagens VS Code
│       │   └── useKanbanBoard.ts # Hook principal de state management
│       ├── components/
│       │   ├── KanbanBoard/      # Board principal + drag-drop
│       │   │   ├── KanbanBoard.tsx
│       │   │   ├── Column.tsx
│       │   │   ├── SortableTask.tsx
│       │   │   └── TaskCard.tsx
│       │   ├── TaskModal.tsx     # Modal de detalhes da task
│       │   └── atoms/            # Componentes UI reutilizáveis
│       ├── types/
│       │   └── kanban.ts         # Definições TypeScript
│       └── __tests__/            # Testes unitários
├── vite.config.ts                # Build config da extension
├── vite.webview.config.ts        # Build config do webview
└── dist/
    ├── extension.js              # Extension compilada
    └── webview/                  # Webview compilado (React bundle)
```

---

## Integração Extension ↔ Webview ↔ Markdown

### Fluxo de Comunicação

```text
┌─────────────────────────────────────────────────────────────────┐
│                         VS Code                                  │
│  ┌─────────────────┐    ┌─────────────────┐    ┌─────────────┐ │
│  │  extension.ts   │◄──►│kanbanWebviewPanel│◄──►│MarkdownFile│ │
│  │  (activation)   │    │  (message hub)   │    │ (.kanban.md)│ │
│  └─────────────────┘    └────────┬─────────┘    └─────────────┘ │
│                                  │                               │
│                          postMessage()                           │
│                                  │                               │
│  ┌───────────────────────────────▼───────────────────────────┐  │
│  │                      WEBVIEW (iframe)                      │  │
│  │  ┌─────────────────┐    ┌─────────────────────────────┐   │  │
│  │  │  useVSCodeApi   │◄──►│      useKanbanBoard         │   │  │
│  │  │ (message layer) │    │ (state + optimistic updates)│   │  │
│  │  └─────────────────┘    └──────────────┬──────────────┘   │  │
│  │                                        │                   │  │
│  │  ┌─────────────────────────────────────▼───────────────┐  │  │
│  │  │              KanbanBoard (React + dnd-kit)          │  │  │
│  │  │  ┌─────────┐  ┌─────────┐  ┌─────────┐              │  │  │
│  │  │  │ Column  │  │ Column  │  │ Column  │              │  │  │
│  │  │  │┌───────┐│  │┌───────┐│  │┌───────┐│              │  │  │
│  │  │  ││Task   ││  ││Task   ││  ││Task   ││              │  │  │
│  │  │  │└───────┘│  │└───────┘│  │└───────┘│              │  │  │
│  │  │  └─────────┘  └─────────┘  └─────────┘              │  │  │
│  │  └─────────────────────────────────────────────────────┘  │  │
│  └───────────────────────────────────────────────────────────┘  │
└─────────────────────────────────────────────────────────────────┘
```

### Protocolo de Mensagens

**Extension → Webview:**

```typescript
{ type: 'updateBoard', board: KanbanBoard }
{ type: 'toggleTaskExpansion', taskId: string }
```

**Webview → Extension:**

```typescript
{
  type: "webviewReady";
}
{
  type: "moveTask", taskId, fromColumnId, toColumnId, newIndex;
}
{
  type: "updateTask", taskId, updates;
}
{
  type: "addTask", columnId, taskData;
}
{
  type: "deleteTask", taskId, columnId;
}
```

---

## Sistema Anti-Flickering (Queue + Fingerprint)

### O Problema

Durante drag & drop, múltiplas operações podem causar:

1. Race conditions entre saves
2. Backend enviando estado "antigo" enquanto webview já atualizou
3. Re-renders desnecessários causando flicker visual

### Solução: 3 Padrões Combinados

#### 1. Fingerprint Pattern

Cria uma "impressão digital" do estado do board para comparação rápida:

```typescript

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wguilherme/kanban.md](https://github.com/wguilherme/kanban.md) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-24 -->
