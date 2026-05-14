---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project status & community

NTK is an **open initiative**. The current maintainer can only cover so
much ground — we actively need collaborators for: stack-trace classifier
coverage (more languages/frameworks), real-hardware benchmarks on GPUs
we don't own (AMD / Apple Silicon / Intel AMX), L3 prompt A/B testing,
hook compatibility with editors other than Claude Code & OpenCode, and
translations of the docs site. Issues and PRs are welcome at
<https://github.com/VALRAW-ALL/ntk>. See **`CONTRIBUTING.md`** for the
concrete starter tasks.

## Implementation Guide

> **Implementando do zero?** Siga `IMPLEMENTATION-SEQUENCE.md` — 26 etapas ordenadas por dependência, com critério de "pronto" em cada uma. Não pule etapas.

## Project-specific rules (`.claude/rules/`)

These rules are loaded automatically when you work on matching files:

| Rule | Triggers on |
|---|---|
| `implementation-gate.md` | **every code change** — security, memory, quality, issue fidelity checklist |
| `clippy-gate.md` | any Rust change — the CI clippy flags that must pass locally |
| `cuda-ci.md` | edits to `.github/workflows/release.yml` with CUDA steps |
| `gpu-vendor.md` | `src/gpu.rs`, `src/config.rs`, `src/main.rs` setup wizard, `layer3_llamacpp.rs` |
| `l1-l2-invariants.md` | any L1/L2 algorithm change — the 5 invariants CI enforces |
| `l1-template-dedup.md` | `group_by_template` changes — blank-line handling |
| `stack-trace-classifier.md` | `is_framework_frame` extensions, new language support |
| `l4-context-injection.md` | `layer4_context.rs`, server handler, hook scripts |

## Project-specific skills (`.claude/skills/`)

| Skill | When to invoke |
|---|---|
| `bench-runner` | Running microbench / full bench / prompt A/B / Claude Code A/B |
| `add-stack-trace-language` | Extending L1's classifier to a new language/framework |

## Skills e Agentes Disponíveis neste Projeto

Skills e agentes do Claude Code configurados para auxiliar na implementação, testes e validação do NTK.

### Skills Auto-Acionadas por Contexto

| Skill | Quando acionar |
|---|---|
| **`write-tests`** | Ao implementar qualquer módulo novo (`layer1_filter.rs`, `layer3_inference.rs`, etc.) — gera suite completa unit + integration |
| **`clean-code`** | Ao finalizar cada etapa do `IMPLEMENTATION-SEQUENCE.md` — verifica naming, funções, SOLID |
| **`architecture-review`** | Antes de iniciar Fases 3, 5 e 8 — valida decisões de design antes de implementar |
| **`dead-code`** | Após Etapa 17 (Layer 3 integrada) e após Etapa 26 (CI) — remove código não usado |
| **`brainstorm`** | Ao encontrar decisão de design não coberta no roadmap — ex: como estruturar detecção RTK output |

### Regras Automáticas por Fase

```
FASE 1-2 (Estrutura + Config):
  → clean-code ao finalizar config.rs (naming em inglês, sem magic numbers)

FASE 2 (Camadas de compressão):
  → write-tests ao finalizar cada layer (layer1, layer2, detector)
  → clean-code antes de commitar cada módulo

FASE 3 (Daemon HTTP):
  → architecture-review antes de definir schema JSON do /compress endpoint
  → write-tests ao finalizar server.rs

FASE 5 (Layer 3 / Inferência):
  → brainstorm se qualidade de compressão nos fixtures for < 70%
  → write-tests para cliente Ollama (mock via wiremock)

FASE 8 (GPU):
  → architecture-review antes de implementar gpu.rs (abstração de backend)

FASE 9 (Testes avançados):
  → write-tests para proptest + snapshots insta
  → dead-code ao finalizar todos os testes
```

### Comandos de Validação por Etapa

Após cada etapa do `IMPLEMENTATION-SEQUENCE.md`, rodar **nesta ordem**:

```bash
# 1. Compila?
cargo check

# 2. Lints de segurança (unwrap, panic, overflow)
cargo clippy -- \
  -W clippy::unwrap_used \
  -W clippy::expect_used \
  -W clippy::panic \
  -W clippy::arithmetic_side_effects \
  -D warnings

# 3. Testes
cargo test

# 4. Formatação
cargo fmt --check

# 5. Auditoria de dependências (se Cargo.toml mudou)
cargo audit
```

Se qualquer um falhar, **não avançar para a próxima etapa**.

### Regra de Segurança (automática)

A regra **`rust-security-audit`** está ativa globalmente e é acionada automaticamente após qualquer implementação de código Rust. Cobre:

- `unwrap()`/`expect()` em produção → panic/DoS
- Validação de tamanho de input antes de alocar
- Path traversal em `ntk test-compress`
- SSRF via `ollama_url` configurável
- Prompt injection na Layer 3
- Escrita atômica em `settings.json`
- Permissões de arquivo (`600`/`700`)
- Integer overflow em operações com dados externos
- `unsafe` blocks obrigatoriamente comentados
- Telemetria: NTK_TELEMETRY_DISABLED verificado antes de coletar

Ver regra completa em: `~/.claude/rules/rust-security-audit.md`

### Agente de Revisão de Arquitetura

Antes de iniciar as fases críticas, usar o agente `Plan` para validar:

```
Fase 3 (Daemon):  Validar schema do endpoint /compress e estrutura do pipeline
Fase 5 (Layer 3): Validar integração Ollama + fallback + threshold logic
Fase 8 (GPU):     Validar abstração GpuBackend e feature flags CUDA/Metal
```

## Project Overview


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [VALRAW-ALL/ntk](https://github.com/VALRAW-ALL/ntk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
