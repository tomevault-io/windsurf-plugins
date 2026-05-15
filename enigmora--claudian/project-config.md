---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

---

## CRITICAL RULES - READ FIRST

> **These rules are NON-NEGOTIABLE and must be followed at all times.**

### 1. NEVER commit directly to `main`
```
STOP! Before ANY commit, ask yourself:
- Am I on main? → WRONG. Create a feature branch first.
- Did I create a branch? → Good. Proceed with commit.
```

**Workflow for ANY change:**
```bash
# 1. FIRST: Create and switch to a feature branch
git checkout -b feat/your-feature-name

# 2. Make your changes and commits on the branch
git add <files>
git commit -m "feat: description"

# 3. Push the branch (NOT main)
git push -u origin feat/your-feature-name

# 4. Create PR via GitHub
```

### 2. All strings must be internationalized
Use `t('key')` from `./i18n` for ALL user-visible text. Never hardcode strings.

### 3. Never use console.log directly
Use `logger.debug/info/warn/error` from `./logger.ts`.

---

## Project Overview

**Claudian** - The ultimate Claude AI integration for Obsidian. Powered by Claude.

Plugin de Obsidian para chat con Claude y generación de notas estructuradas con wikilinks, tags y YAML frontmatter.

## Build Commands

```bash
npm install              # Instalar dependencias
npm run dev              # Build desarrollo (con sourcemaps)
npm run build            # Build producción (minificado)
```

**Deploy scripts (compile + copy to vault):**

| Platform | Command |
|----------|---------|
| Linux/macOS | `./deploy.sh . <destino>` |
| Windows (PowerShell) | `.\deploy.ps1 -Destination <destino>` |

Both scripts support development mode for debug logging:
```bash
# Linux/macOS
./deploy.sh . <destino> -d

# Windows PowerShell
.\deploy.ps1 -Destination <destino> -Dev
```

## Testing Local

**Option 1: Use deploy scripts**
```bash
# Linux/macOS
./deploy.sh . /ruta/a/boveda/.obsidian/plugins/claudian/

# Windows PowerShell
.\deploy.ps1 -Destination "C:\Users\tu-usuario\boveda\.obsidian\plugins\claudian"
```

**Option 2: Manual copy**
1. Run `npm run build`
2. Copy files from `dist/` to `.obsidian/plugins/claudian/`

**After deploying:**
1. Recargar Obsidian (Ctrl/Cmd + R)
2. Activar plugin en Settings > Community Plugins

## Architecture

```
src/
├── main.ts                  # Entry point, registra comandos y vistas
├── settings.ts              # PluginSettingTab con config (API key, modo, etc.)
├── claude-client.ts         # Wrapper Anthropic SDK con streaming
├── chat-view.ts             # ItemView para panel lateral de chat
├── model-orchestrator.ts    # Enrutador inteligente de modelos
├── logger.ts                # Sistema de logging centralizado
├── note-creator.ts          # Modal para crear notas desde chat
├── note-processor.ts        # Procesamiento de notas existentes
├── vault-indexer.ts         # Indexación de bóveda
├── suggestions-modal.ts     # Modal de sugerencias interactivo
├── extraction-templates.ts  # Templates de extracción predefinidos
├── batch-processor.ts       # Procesamiento batch de notas
├── batch-modal.ts           # Modal de selección para batch
├── concept-map-generator.ts # Generador de mapas de conceptos
├── vault-actions.ts         # Ejecutor de acciones sobre bóveda
├── agent-mode.ts            # Gestión del modo agente
├── confirmation-modal.ts    # Modal de confirmación de acciones
├── context-manager.ts       # Gestión de contexto de conversación
├── context-storage.ts       # Almacenamiento temporal de contexto
├── i18n/                    # Internationalization system
│   ├── index.ts             # Public API (t, setLocale, etc.)
│   ├── types.ts             # TypeScript types and translation keys
│   ├── core.ts              # Runtime logic
│   └── locales/
│       ├── en.ts            # English translations (default)
│       ├── es.ts            # Spanish translations
│       ├── zh.ts            # Chinese translations (Simplified)
│       ├── de.ts            # German translations
│       ├── fr.ts            # French translations
│       └── ja.ts            # Japanese translations
└── templates/
    └── default.ts           # Template de notas con frontmatter
```

**Flujo principal:**
- `main.ts` inicializa plugin y registra `ChatView` como vista lateral
- `ChatView` usa `ClaudeClient` para streaming de mensajes
- Modo agente permite ejecutar acciones sobre la bóveda via lenguaje natural
- Respuestas se pueden convertir a notas via `NoteCreator` modal
- Settings persisten en `.obsidian/plugins/claudian/data.json`

## Key Patterns

**Obsidian API:**
- Extender `Plugin` para entry point
- Extender `ItemView` para paneles personalizados
- Extender `PluginSettingTab` para UI de configuración
- Usar `MarkdownRenderer.render()` para renderizar respuestas
- Usar `app.fileManager.processFrontMatter()` para modificar YAML

**Anthropic SDK:**
- Requiere `dangerouslyAllowBrowser: true` para funcionar en Obsidian
- Usar `client.messages.stream()` con eventos `on('text')` para streaming
- API key almacenada localmente, nunca en código

**CSS:**
- Usar variables de Obsidian (`--background-primary`, `--text-normal`, etc.)
- Soporte automático tema claro/oscuro
- Clases con prefijo `claudian-` para evitar conflictos

## Internationalization (i18n)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Enigmora/claudian](https://github.com/Enigmora/claudian) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-10 -->
