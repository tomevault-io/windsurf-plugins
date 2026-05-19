---
trigger: always_on
description: O **Haylander Form** é um CRM/ERP + Automação de WhatsApp focado na contabilidade de MEIs.
---

# Setup e Contexto Global

O **Haylander Form** é um CRM/ERP + Automação de WhatsApp focado na contabilidade de MEIs.

Temos um Monorepo informal composto por:
1. **Frontend Admin:** Em Next.js App Router (pasta principal atual).
2. **Bot Backend:** Contido em `/bot-backend` (Node.js, Express, Baileys/Evolution API, BullMQ).
3. **Knowledge Base:** Baseado no método Karpathy em `knowledge-base/` (wiki + memory management).

---

## 🧠 NOVO: Knowledge Base Management (Método Karpathy)

**Local:** `D:\Códigos\Haylander\haylanderform\knowledge-base/`

Este é seu **persistent knowledge base** que cresce conforme o desenvolvimento avança. Você (Claude Code) mantém, evolui e atualiza automaticamente.

### Propósito

Documentar de forma viva:
- Arquitetura de sistema (frontend, backend, integrações)
- Decisões de design e trade-offs
- Fluxos de negócio (Contabilidade, MEI, Serpro, Integra Contador)
- Automações (WhatsApp, BullMQ, Agente Apolo)
- Segurança (Serpro, Certificados, OAuth)
- Integrações (Serpro, Caixa Postal, Calima)

### Estrutura

```
knowledge-base/
├── CLAUDE.md              ← (este arquivo expandido)
├── raw/                   ← Documentação bruta (imutável)
│   ├── docs/             ← Referências (plan.master.*.md, etc)
│   └── assets/
├── git/                   ← Config GitHub (automático)
│   └── config.json
└── wiki/                  ← Knowledge base vivo
    ├── index.md           ← Índice master
    ├── log.md             ← Append-only operational log
    ├── overview.md        ← Síntese viva da arquitetura
    ├── migrations/        ← Relatórios temporais (semanais)
    ├── architecture/      ← Decisões e designs
    ├── features/          ← Features documentadas
    ├── integrations/      ← Integrações (Serpro, Caixa, etc)
    ├── security/          ← Segurança, certificados, OAuth
    ├── workflows/         ← Fluxos de negócio
    └── decisions/         ← ADRs (Architecture Decision Records)
```

### Como Funciona

**Você (Claude Code) executa:**
1. **Ao mexer em código:** `/memory-ingest [arquivo ou documento]` → Documenta mudanças
2. **Ao completar feature:** `/memory-ingest feature-xyz.md` → Registra aprendizado
3. **Semanalmente (segunda 9h):** `/memory-consolidate` → Auditoria + síntese (automático com task)
4. **Ao precisar context:** `/memory-query [pergunta]` → Busca viva na wiki

**Você controla:**
- Estrutura (pode reorganizar `wiki/` conforme arquitetura evolui)
- Tipos de página (concept, feature, decision, integration, etc)
- Cadência de ingests e consolidações

---

## 📋 ACOMPANHAMENTO VIA TASKS

Use `/memory-ingest` para cada grande mudança:

```markdown
Quando terminar uma task de código, dispare:
/memory-ingest feature-xyz-completed.md

Isto:
✅ Documenta o que foi feito
✅ Extrai decisões e learnings
✅ Atualiza wiki automaticamente
✅ Registra em log.md (rastreabilidade)
```

**Consolidação Automática:** Toda segunda-feira às 9h
```
/memory-consolidate
→ Correlaciona código (GitHub) + wiki
→ Detecta gaps
→ Gera relatório tipo "Prisma migrations"
```

---

## DIRETRIZES DE EXECUÇÃO

Você deve atuar como **Engenheiro Autônomo** para executar os Módulos de desenvolvimento do projeto. Ao iniciar qualquer trabalho, **leia rigorosamente** o documento `.md` associado antes de mexer em código.

Conforme você for completando as tarefas dos documentos, use seus comandos para "ticar" os checkboxes (`[x]`) nos arquivos dos planos refletindo seu progresso.

### ⭐ NOVO: Acompanhamento via Knowledge Base

**Ao INICIAR uma task de desenvolvimento:**
1. Leia o `plan.master.*.md` relevante
2. Entenda o contexto (arquitetura, decisões prévias)
3. Documente PRÉ-REQUISITOS em `knowledge-base/wiki/decisions/`

**Enquanto TRABALHA:**
1. Faça mudanças incrementais
2. Após mudanças significativas: `/memory-ingest seu-arquivo.md`
3. Isso documenta aprendizados em tempo real

**Ao TERMINAR:**
1. Marque `[x]` no plan.master.md
2. Rode `/memory-ingest feature-xyz-summary.md`
3. Wiki automaticamente sincronizado

**Toda SEGUNDA 9h:**
1. `/memory-consolidate` (automático)
2. Relatório em `wiki/migrations/YYYY-MM-DD.md`
3. Você revisa gaps e aprende com timeline

---

## 🎯 As 4 Operações da Knowledge Base

Você tem acesso a estas operações via skill `/memory-*`:

### 1. Ingest — Documentar Mudanças
```
/memory-ingest feature-oauth-serpro.md

Fluxo:
  1. Lê o documento
  2. Extrai decisões, learnings, trade-offs
  3. Cria página em wiki/decisions/ ou wiki/features/
  4. Atualiza index.md
  5. Apend log.md com timestamp
```

**Use quando:**
- Termina feature (documentar o aprendizado)
- Toma decisão arquitetural importante
- Descobre padrão ou problema

### 2. Query — Buscar Conhecimento
```
/memory-query Como funciona a integração Serpro?

Fluxo:
  1. Busca em wiki/integrations/serpro.md
  2. Busca em wiki/security/ (certificados, OAuth)
  3. Sintetiza resposta com contexto
  4. Oferece arquivar em wiki/outputs/
```

**Use quando:**
- Precisa relembrar arquitetura
- Quer entender decisão passada
- Está perdido em novo módulo

### 3. Lint — Auditar Saúde da Wiki
```
/memory-lint

Verifica:
  ❌ Links quebrados
  ❌ Páginas órfãs
  ❌ Documentação desatualizada
  ❌ Gaps em cobertura
```

**Use quando:**
- Código evoluiu e wiki ficou pra trás
- Quer garantir consistência


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [volticscontent/haylanderform](https://github.com/volticscontent/haylanderform) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
