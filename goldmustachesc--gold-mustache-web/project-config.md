---
trigger: always_on
description: Diretrizes globais para agentes e IA neste repositório. Este arquivo é a fonte canônica e único contrato compartilhado entre **Claude Code**, **Codex** e **OpenCode** — todos os 3 leem `AGENTS.md` automaticamente. Workflows longos ficam em `.claude/skills/` (espelhados via symlink em `.opencode/command/` quando aplicável).
---

# Gold Mustache - AI Development Guidelines

## Overview

Diretrizes globais para agentes e IA neste repositório. Este arquivo é a fonte canônica e único contrato compartilhado entre **Claude Code**, **Codex** e **OpenCode** — todos os 3 leem `AGENTS.md` automaticamente. Workflows longos ficam em `.claude/skills/` (espelhados via symlink em `.opencode/command/` quando aplicável).

## Ferramentas suportadas

| Provider | Lê | Tooling |
|---|---|---|
| Claude Code | `AGENTS.md`, `CLAUDE.md` (→ `@AGENTS.md`), `.claude/{skills,agents,commands,hooks}/`, `.claude/settings.json`, `.mcp.json` | hooks runtime, slash commands, MCP project |
| Codex | `AGENTS.md`, `~/.codex/config.toml` (user-level) | MCP user-level (`scripts/setup-codex-mcp.sh` adiciona vercel + prisma) |
| OpenCode | `AGENTS.md`, `opencode.json`, `.opencode/{command,plugins}/` | MCP project, plugins (hooks), slash commands |

Nenhum provider lê configuração específica do outro: contrato precisa estar em `AGENTS.md` ou ser duplicado em arquivos paralelos. Quando precisar adicionar diretriz nova, **edite `AGENTS.md` primeiro**.

## Repository structure

- `src/app` — Next.js App Router, layouts, route handlers
- `src/components/ui` — primitivos; `src/components/custom` — widgets específicos
- `src/hooks`, `src/utils`, `src/lib`, `src/services` — lógica compartilhada e integrações
- `src/config`, `src/constants`, `src/types` — config e contratos; `public/` — assets
- Imports com alias `@/` (ver `tsconfig.json`)

## Commands

| Comando | Uso |
|---------|-----|
| `pnpm install` | Dependências; manter lockfile commitado |
| `pnpm dev` | Dev local (Turbopack), `http://localhost:3001` |
| `pnpm build` | Build de produção |
| `pnpm start` | Servir build compilado |
| `pnpm lint` / `pnpm format` | Biome |
| `pnpm test` / `pnpm test:watch` | Vitest |
| `pnpm test:gate` | Lint + test + coverage — antes de PR |
| `pnpm snyk:test` / `pnpm snyk:code` | Snyk SCA e SAST locais |

## Core

- TypeScript em todo código novo (`.ts`/`.tsx`). Imports com alias `@/`.
- **Proibido `any`**. Preferir tipos explícitos, `unknown` com narrowing, generics e utility types.
- **TDD obrigatório** (RED → GREEN → REFACTOR) para: lógica de negócio (`src/services/**`), hooks (`src/hooks/**`), route handlers (`src/app/api/**`), utils (`src/utils/**`), integrações (`src/lib/**`). Rodar `pnpm test` após cada etapa.
- **TDD opcional** para: componentes UI sem lógica em `src/components/ui/**` (Tailwind puro, primitivos). Snapshot ou visual regression cobre melhor.
- Cobertura medida por **caminho crítico** (auth, booking, billing, fidelidade), não % global.
- Clean Code, SOLID, KISS, YAGNI; evitar overengineering.
- Biome: seguir `biome.json` (indentação, import sorting). Não duplicar o que o linter já garante.
- Decisões arquiteturais relevantes: documentar em PR ou na resposta ao revisor, não em comentários no código.
- Preferir código autoexplicativo; evitar comentários desnecessários.

### Anti-padrões

| Anti-padrão | Sintoma | Mitigação |
|---|---|---|
| Skip do RED | Teste passa de primeira sem falhar antes | Confirmar que teste falha antes de implementar |
| `any` no TS | "tipagem depois" | Bloqueado por Biome/tsconfig |
| Vibe coding em superfície sensível | Auth/billing sem spec nem teste | Risco ALTO força tier Full (`.kiro/TIERS.md`) |
| Multi-agente sem orchestrator | Trabalho duplicado, conflitos | Definir lead session |
| Spec inflada | brainstorm.md de 5k palavras pra feature pequena | Cap: brainstorm ≤ 800, requirements ≤ 1200 palavras |
| Gate skipado | "depois eu rodo" | CI bloqueia merge sem `test:gate` verde |

## SDD — Subagent-Driven Development

- Aplicado com cerimônia proporcional ao tier (ver `.kiro/TIERS.md`).
- Classificar antes de executar: **Trivial** → execução direta, **Light** → só requirements, **Full** → brainstorm + spec completa.
- Mentalidade SDD sempre ativa (decomposição, delegação, paralelização, checkpoints); cerimônia apenas quando agrega valor.
- `SDD` complementa `TDD`, testes, validação final. Nunca usar `SDD` como justificativa pra pular testes, revisão de impacto, confirmação de causa raiz ou validações necessárias.

### Quando NÃO delegar pra subagent

- Task sequencial e curta (< 10 min de implementação)
- Resposta cabe em 1 ferramenta (1 grep, 1 read, 1 edit)
- Decisão crítica que precisa de contexto cumulativo da sessão
- Edição de 1 linha em 1 arquivo conhecido

## App Router e APIs

Aplica-se a `src/app/**/*.{ts,tsx}`.

- Respeitar limites server/client: `"use client"` só quando necessário; dados sensíveis e secrets só no servidor.
- Route handlers: validar entrada com Zod; respostas HTTP adequadas; não expor stack traces nem detalhes internos ao cliente.
- Autenticação/autorização: verificar sessão Supabase e regras de negócio antes de operações sensíveis.
- Preferir helpers de `@/lib/api/response` (`apiSuccess`, `apiError`, `apiCollection`, etc.) nas route handlers.
- Server actions: mesma disciplina de validação e permissões que em rotas HTTP.

## Prisma e Supabase


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [goldmustachesc/gold-mustache-web](https://github.com/goldmustachesc/gold-mustache-web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
