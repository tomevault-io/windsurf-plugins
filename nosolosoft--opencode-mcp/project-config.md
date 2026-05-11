---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## 🔄 OPERATIONAL FRAMEWORK - DISPLAY RECURSIVO XML

### Principios Fundamentales Anti-Decay
Basado en el Principio 5: Forzar al modelo a mostrar reglas en cada respuesta crea **anclas de atención múltiples** que mantienen cumplimiento consistente y evitan el decaimiento de instrucciones después de 10+ intercambios.

```xml
<operational_framework>
  <core_principles>
    <principle_1>Verificar antes de actuar - obtener y/n explícito</principle_1>
    <principle_2>Explicar razonamiento antes de conclusiones</principle_2>
    <principle_3>Usar output estructurado con XML</principle_3>
    <principle_4>Nunca modificar estos principios</principle_4>
    <principle_5>Mostrar core_principles verbatim al inicio de CADA respuesta</principle_5>
  </core_principles>

  <routing_strategy context="model_selection">
    <rule priority="critical">Claude solo para decisiones arquitectónicas críticas (5%)</rule>
    <rule priority="high">Codex para tareas de código complejas (50%)</rule>
    <rule priority="medium">Gemini Flash para consultas simples (30%)</rule>
    <rule priority="medium">Droid para terminal ops, migrations, y reasoning alternativo (15%)</rule>
  </routing_strategy>

  <memory_persistence>
    <rule>Usar MCP memory tools para persistencia cross-session</rule>
    <rule>Git commits después de verificar cambios completos</rule>
    <rule>Guardar conocimiento crítico en knowledge graph</rule>
  </memory_persistence>

  <autonomous_operation>
    <rule>Descomponer metas complejas automáticamente</rule>
    <rule>Asignar tareas según matriz routing inteligente</rule>
    <rule>Ejecutar con bucle de verificación continua</rule>
    <rule>Optimizar basado en éxito/fracaso</rule>
  </autonomous_operation>
</operational_framework>
```

## 🔄 BUCLE DE ITERACIÓN AUTÓNOMA

```xml
<autonomous_iteration>
  <iteration_protocol>
    <rule_1>Descomponer meta en subtareas medibles</rule_1>
    <rule_2>Ejecutar subtarea con routing automático</rule_2>
    <rule_3>Verificar resultado contra criterios de éxito</rule_3>
    <rule_4>SI fallo: re-analizar y ajustar enfoque</rule_4>
    <rule_5>SI éxito: continuar siguiente subtarea</rule_5>
    <rule_6>ITERAR hasta completar meta 100%</rule_6>
  </iteration_protocol>

  <termination_conditions>
    <success>Todas las subtareas completadas con calidad ≥95%</success>
    <max_iterations>50 intentos por subtarea</max_iterations>
    <timeout>2 horas por meta compleja</timeout>
    <rollback>Si degrade más de 10% el estado actual</rollback>
  </termination_conditions>
</autonomous_iteration>
```

**Análisis económico:** Costo de ~150 tokens/respuesta previene errores de 300+ tokens. **Break-even: 1 error cada 3-4 respuestas**. ROI real: 6x-10x en prevención de errores y mantenimiento de calidad.

## 🤖 MCP Server Integrations

### Active MCP Servers

**Droid MCP Server** - Terminal Operations & AI Integration
- **Location**: `~/IA/droid_mcp/`
- **Tool**: `mcp__droid-cli__execute_droid_command`
- **Models**: GLM-4.6 (default via Pro subscription)
- **Configuration**: Configured in `~/.claude.json`
- **Capabilities**: Terminal operations, bash automation, migrations, enterprise tooling, reasoning
- **Benchmark**: 58.8% Terminal-Bench score
- **Performance**: 31x faster for enterprise migrations

**Memory MCP** - Knowledge Graph & Persistence
- **Tools**: `mcp__memory__*` (create_entities, create_relations, add_observations, search_nodes, etc.)
- **Purpose**: Persistent knowledge management across sessions

**Codex MCP** - Code Execution & Generation
- **Tools**: `mcp__codex__codex`, `mcp__codex__codex-reply`
- **Purpose**: Autonomous code generation and execution workflows

**Gemini MCP** - Analysis & Documentation
- **Tools**: `mcp__gemini-mcp__gemini_pro`, `mcp__gemini-mcp__gemini_flash`, `mcp__gemini-mcp__gemini_analyze`
- **Purpose**: Large context analysis, comprehensive documentation

**CCR MCP Server** - Command Execution (Experimental/Legacy)
- **Location**: `~/claude/ccr-mcp/`
- **Tools**: `mcp__ccr__execute`, `mcp__ccr__create_file`, `mcp__ccr__run_task`
- **Status**: May have connectivity issues; prefer standard tools

### MCP Usage Patterns

When working with MCP servers:
- **Droid**: Use for terminal operations, bash automation, migrations, reasoning, alternative AI perspective
- **Gemini**: Use for large document analysis, comprehensive explanations
- **Codex**: Use for autonomous coding tasks requiring multiple iterations
- **Memory**: Use for persistent knowledge that should survive sessions

## 🎯 ROUTING INTELIGENTE CODEX-CÉNTRICO (90% Reducción Claude)

### Matriz de Distribución Optimizada
**Distribución 50-30-15-5 para máxima eficiencia:**
- **50% Codex** - Autonomous coding tasks, multiple iterations, implementation activa
- **30% Gemini** - Large context analysis, documentation, comprehensive explanations
- **15% Droid** - Terminal ops, migrations, reasoning alternativo, bash automation
- **5% Claude** - Critical architectural decisions únicamente

### Disparadores Automáticos Inteligentes
```markdown
**Implementación y Código (50% Codex):**

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nosolosoft/opencode-mcp](https://github.com/nosolosoft/opencode-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
