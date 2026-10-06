---
trigger: always_on
description: Este repositorio distribuye los plugins `fs-dev`, `fs-user` y `fs-mcp` de FacturaScripts. Aunque el repositorio conserva el nombre `fs-claude-plugin`, el código debe funcionar tanto en Claude Code como en Codex.
---

# Instrucciones del proyecto para Claude Code y Codex

Este repositorio distribuye los plugins `fs-dev`, `fs-user` y `fs-mcp` de FacturaScripts. Aunque el repositorio conserva el nombre `fs-claude-plugin`, el código debe funcionar tanto en Claude Code como en Codex.

## Reglas generales

- Trabaja únicamente dentro de este repositorio, salvo que el usuario autorice expresamente otra carpeta.
- Nunca crees commits, ramas, tags, pushes ni publicaciones sin una petición expresa del usuario.
- Conserva los cambios ajenos presentes en el árbol de trabajo y no reviertas archivos que no formen parte de la tarea.
- No expongas tokens, credenciales ni el contenido sensible de `~/.fs-claude.json`.
- No crees archivos temporales, informes de auditoría o resúmenes dentro del repositorio salvo que el usuario pida expresamente conservarlos. Comunica los resultados en el chat.

## Fuentes de verdad

- Para verificar clases y patrones de FacturaScripts, consulta el código local en `/Users/daniel89fg/Trabajos/FacturaScripts/facturascripts`.
- Para comportamiento específico de Claude Code, consulta la documentación vigente en <https://code.claude.com/docs/>.
- Para comportamiento específico de Codex, consulta la documentación oficial vigente en <https://developers.openai.com/codex/> y <https://learn.chatgpt.com/docs/>.
- No supongas que una característica de un host existe en el otro. Verifica especialmente hooks, agentes, variables de entorno, manifiestos, instalación y recarga.

## Arquitectura compartida

- Mantén un único marketplace y un único código fuente para ambos hosts. No crees un plugin paralelo para Codex salvo que una limitación técnica demostrada haga imposible compartir la implementación y el usuario acepte la separación.
- Conserva los identificadores `fs-dev`, `fs-user`, `fs-mcp` y el marketplace `fs-claude-plugin`.
- Conserva `~/.fs-claude.json` como configuración compartida por retrocompatibilidad.
- La matriz actual de compatibilidad está en [`COMPATIBILITY.md`](COMPATIBILITY.md). Actualízala únicamente cuando cambien capacidades, limitaciones o el flujo de recarga.

## Skills y perfiles especializados

- Las skills de `*/skills/<nombre>/SKILL.md` son la interfaz portable común a Claude Code y Codex.
- Mantén el frontmatter YAML válido y limita `description` a una explicación breve, discriminante y útil para la selección automática. Evita catálogos extensos de ejemplos en la descripción.
- Resuelve los recursos relativos respecto al directorio de la skill y enlázalos desde `SKILL.md`.
- Los agentes Markdown de `*/agents/*.md` son nativos de Claude Code. Codex debe poder reutilizarlos como perfiles leídos desde una skill sin depender de un tipo de agente propietario.
- Cuando añadas una nueva especialidad en `fs-dev`, crea o actualiza conjuntamente:
  1. El perfil `fs-dev/agents/<nombre>.md` para Claude Code.
  2. La skill `fs-dev/skills/<nombre>/SKILL.md` que lea ese perfil y pueda aplicarlo directamente en Codex o delegarlo a un subagente genérico.
- No declares el campo `agents` en `plugin.json` cuando los agentes estén en el directorio estándar `agents/`; permite que Claude Code los descubra automáticamente.
- Si un agente necesita documentación técnica o de usuario, precarga `docs-expert` mediante su campo `skills`.
- Guarda documentación y catálogos únicamente en `references/`, nunca dentro de `agents/`, porque Claude Code valida recursivamente los Markdown situados bajo el directorio de agentes.
- Las ubicaciones canónicas son:
  - `fs-dev/references/docs/`
  - `fs-user/references/docs/`
  - `fs-user/references/projects/`

## Hooks y scripts

- Los hooks deben aceptar los formatos de ambos hosts:
  - Claude Code: `Write` y `Edit`, con `tool_input.file_path`.
  - Codex: `apply_patch`, con el parche en `tool_input.command`.
- Mantén una única entrada secuencial para transformaciones que puedan escribir sobre el mismo archivo. No configures handlers concurrentes para copyright y ordenación.
- El hook canónico de edición es `fs-dev/scripts/post-edit.py`: primero actualiza copyright y después ordena la clase PHP.
- Conserva `SessionStart` en ambos plugins. `CwdChanged` es exclusivo de Claude Code; en Codex el usuario debe abrir un hilo nuevo tras cambiar el directorio de trabajo.
- En hooks compartidos usa `${CLAUDE_PLUGIN_ROOT}`, que ambos hosts proporcionan por compatibilidad. En servidores MCP conserva el lanzador portable: Claude Code exporta `CLAUDE_PLUGIN_ROOT` al subproceso y Codex resuelve `cwd: "."` respecto a la raíz instalada del plugin. Cita siempre las rutas usadas en comandos shell para soportar directorios con espacios.
- Los hooks de Codex requieren revisión del usuario mediante `/hooks`, y una modificación de `hooks.json` invalida la confianza anterior.
- Cualquier cambio funcional en un script debe incluir o actualizar pruebas automatizadas en `fs-dev/tests/`.

## fs-mcp y módulos privados

- El servidor MCP debe seguir funcionando en ambos hosts desde `fs-mcp/server/dist/index.js`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FacturaScripts/fs-claude-plugin](https://github.com/FacturaScripts/fs-claude-plugin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
