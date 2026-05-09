---
trigger: always_on
description: > **Sistema Gerador de Sistemas** - Transforma descricoes de dominios em sistemas Claude Code Native completos e funcionais.
---

# GENESIS - Meta-Sistema Claude Code Native

> **Sistema Gerador de Sistemas** - Transforma descricoes de dominios em sistemas Claude Code Native completos e funcionais.

---

## PRIMEIRA ACAO EM QUALQUER SESSAO

```
1. LER: STATE.yaml
2. VERIFICAR: current.phase e current.next_action
3. USAR: Comandos slash /GENESIS:tasks:*
4. GERAR: Novos sistemas com /GENESIS:tasks:generate-system
```

**Versao Atual:** 2.0.0 (State-of-Art)
**Estado:** Ver STATE.yaml
**Standalone:** Este e um projeto GENESIS standalone pronto para uso

---

## Overview

GENESIS e um **Meta-Sistema** que gera sistemas Claude Code Native completos a partir de descricoes de dominio (domain briefs).

**Core Transformation:**
```
DE: Descricao de dominio (texto livre ou estruturado)
PARA: Sistema Claude Code Native funcional com:
      - Agents especializados
      - Tasks executaveis
      - Workflows orquestrados
      - Knowledge bases
      - Slash commands
      - Validacao automatica
```

**Domain Type:** HYBRID (COGNITIVE + TECHNICAL)

---

## Status v2.0.0 (State-of-Art)

### Componentes

| Componente | Total | Status |
|------------|-------|--------|
| Agents | 5 | COMPLETE |
| Tasks (pipeline) | 22 | COMPLETE |
| Templates | 6 | COMPLETE |
| Knowledge | 12 | COMPLETE |
| Slash Commands | **11** | COMPLETE |
| Skills | 1 | COMPLETE |
| Hooks | 3 | READY |
| SOPs | 4 | COMPLETE |
| Test Suite | 20 tests | COMPLETE |

---

## Comandos Disponiveis (11 total)

### Comandos Principais (v2.0.0 State-of-Art)

```bash
# Gerar sistema completo (v2.0.0)
# - TodoWrite para tracking visual
# - AskUserQuestion em todos os gates
# - Task tool para geracao paralela
# - Extended thinking para classificacao/arquitetura
/GENESIS:tasks:generate-system

# Validar sistema com scoring REAL (v2.0.0)
# - Glob para contagem de arquivos
# - Grep para verificacao de referencias
# - Calculo automatico de scores
/GENESIS:tasks:validate-system
```

### Comandos de Gerenciamento

```bash
# Listar sistemas gerados
/GENESIS:tasks:list-systems

# Ver detalhes de um sistema
/GENESIS:tasks:show-system

# Exportar sistema para deploy
/GENESIS:tasks:export-system

# Deletar sistema (com confirmacao)
/GENESIS:tasks:delete-system
```

### Comandos de Controle

```bash
# Ver status atual
/GENESIS:tasks:check-status

# Retomar geracao interrompida
/GENESIS:tasks:resume-generation

# Abortar geracao
/GENESIS:tasks:abort-generation

# Executar test suite (20 testes)
/GENESIS:tasks:run-tests
```

### Ativacao de Agente

```bash
# Ativar meta-orchestrator
@meta-orchestrator
```

---

## Features State-of-Art (v2.0.0)

### 1. TodoWrite Integration
```yaml
# Ao executar /generate-system:
- Cria 11 items de tracking (fases + gates)
- Atualiza status em tempo real
- Usuario ve progresso visual
```

### 2. AskUserQuestion em Gates
```yaml
# Em cada gate critico:
- G0: Validar entendimento do brief
- G1: Confirmar classificacao de dominio
- G2: Aprovar arquitetura proposta
- G8: Aprovacao final do sistema
```

### 3. Task Tool para Paralelismo
```yaml
# Fases P3-P6 rodam em paralelo:
- P3: Gerar agents simultaneamente
- P4: Gerar workflows
- P5: Gerar tasks
- P6: Gerar arquivos de suporte
```

### 4. Extended Thinking
```yaml
# Em decisoes criticas:
- P1 (Classify): Analisa multiplos fatores
- P2 (Structure): Considera tradeoffs de arquitetura
```

### 5. Scoring Automatico Real
```yaml
# /validate-system usa:
- Glob: Conta arquivos reais (nao estimativas)
- Grep: Verifica referencias existem
- Formula: Calcula scores numericos
```

### 6. Test Suite Completo
```yaml
# /run-tests executa:
- 5 smoke tests
- 5 generation tests
- 5 validation tests
- 5 utility tests
```

---

## Estrutura de Arquivos (Standalone)

```
D:/deploy-ready/genesis/
├── STATE.yaml              # CONSCIENCIA - ler SEMPRE primeiro
├── CLAUDE.md               # Este arquivo
├── README.md               # Documentacao principal
├── config.yaml             # Configuracao do sistema
│
├── .claude/
│   ├── commands/GENESIS/
│   │   ├── tasks/              # 11 slash commands
│   │   │   ├── generate-system.md      (v2.0.0)
│   │   │   ├── validate-system.md      (v2.0.0)
│   │   │   ├── check-status.md
│   │   │   ├── resume-generation.md
│   │   │   ├── abort-generation.md
│   │   │   ├── list-systems.md
│   │   │   ├── show-system.md
│   │   │   ├── export-system.md
│   │   │   ├── delete-system.md
│   │   │   └── run-tests.md
│   │   └── agents/
│   │       └── meta-orchestrator.md
│   ├── skills/
│   │   └── genesis-core.md
│   └── settings.local.json
│
├── agents/                 # 5 agents especializados
│   ├── domain-analyst.md
│   ├── system-architect.md
│   ├── component-generator.md
│   ├── system-validator.md
│   └── meta-orchestrator.md
│
├── tasks/                  # 22 tasks do pipeline
│   ├── 00-interpret/       # P0: Interpret
│   ├── 01-classify/        # P1: Classify
│   ├── 02-structure/       # P2: Structure
│   ├── 03-agents/          # P3: Agents
│   ├── 04-workflows/       # P4: Workflows
│   ├── 05-tasks/           # P5: Tasks
│   ├── 06-support/         # P6: Support
│   ├── 07-generate/        # P7: Generate
│   └── 08-validate/        # P8: Validate
│
├── workflows/              # Orquestracao

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [joaolozano-lendario/genesis-meta-system](https://github.com/joaolozano-lendario/genesis-meta-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
