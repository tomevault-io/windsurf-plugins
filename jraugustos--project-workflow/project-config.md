---
trigger: always_on
description: >
---


# Project Workflow — Framework de Desenvolvimento Estruturado

Framework em **5 etapas** para desenvolver projetos com IA sem perder qualidade.
O principio central: **cada etapa roda em uma conversa separada**, com limpeza de contexto entre elas.
O output de cada etapa e um documento que alimenta a proxima.

## Os 5 Problemas que Este Framework Resolve

1. **Over-engineering** — a IA complica o que poderia ser simples
2. **Reinventar a roda** — criar do zero em vez de usar padroes existentes
3. **Limites de conhecimento do modelo** — docs mais recentes que o training cutoff
4. **Codigo duplicado** — componentes repetidos por falta de visao global
5. **Nao saber implementar o que foi pedido** — bloqueio por falta de pesquisa

## Principio: Context Window Discipline

```
CONVERSA 1 — Pesquisa
  Pesquisar padroes, refs, docs → gerar PRD.md
  /clear (ou nova conversa)

CONVERSA 2 — Planejamento
  Ler PRD.md → gerar SPEC.md (arquitetura + decisoes)
  /clear (ou nova conversa)

CONVERSA 3 — Quebra da Spec
  Ler SPEC.md → gerar TASKS.md (tarefas atomicas ordenadas)
  /clear (ou nova conversa)

CONVERSA 4..N — Build (uma por task)
  Ler TASKS.md → implementar Task Txx → testar → commit
  /clear (ou nova conversa)
  Ler TASKS.md → implementar Task Txx+1 → testar → commit
  ...
```

A qualidade do input determina a qualidade do output.
Se a spec nao e clara, a IA vai implementar do jeito dela — e provavelmente vai sair ruim.

---

## Etapa 0: Setup do Projeto

Antes de comecar, estruture o workspace:

```
projeto/
├── docs/
│   ├── PRD.md              ← Output da Etapa 1
│   ├── SPEC.md             ← Output da Etapa 2
│   ├── TASKS.md            ← Output da Etapa 3
│   ├── RESEARCH-LOG.md     ← Notas brutas da pesquisa (opcional)
│   └── DECISIONS.md        ← ADRs simplificados (opcional)
├── CLAUDE.md               ← Instrucoes persistentes para o Claude Code
└── src/                    ← Codigo (Etapa 4+)
```

### CLAUDE.md inicial

Crie um `CLAUDE.md` na raiz com contexto minimo do projeto:

```markdown
# [Nome do Projeto]

## Contexto
[1-2 frases sobre o que e o projeto]

## Stack
[Tecnologias escolhidas ou a definir]

## Convencoes
- [Patterns de codigo, naming, etc.]

## Docs de referencia
- docs/PRD.md — Requisitos do produto
- docs/SPEC.md — Especificacao tecnica
- docs/TASKS.md — Tarefas de implementacao
```

---

## Etapa 1: Pesquisa (Research) — CONVERSA 1

**Objetivo:** Entender o problema, encontrar padroes existentes, estudar implementacoes de referencia.
**Output:** `docs/PRD.md`
**Duracao tipica:** 1 conversa

### O que fazer nesta etapa

1. **Definir o problema claramente**
   - O que precisa ser construido?
   - Quem vai usar?
   - Quais sao os requisitos funcionais e nao-funcionais?

2. **Pesquisar padroes e implementacoes existentes**
   - Buscar no Stack Overflow, GitHub, docs oficiais
   - Importar repos de referencia para o Claude analisar (clonar em pasta temporaria)
   - Preferir padroes documentados e provados — nunca reinventar a roda
   - Se a feature depende de docs mais recentes que o modelo conhece, usar WebSearch/WebFetch

3. **Analisar trade-offs**
   - Listar abordagens possiveis
   - Comparar pros/contras de cada uma
   - Decidir a abordagem antes de planejar

### Tecnicas de pesquisa

| Tecnica | Quando usar |
|---|---|
| **Clonar repo de referencia** | Projeto open-source faz algo similar — clone em `/tmp` e peca ao Claude para estudar o padrao |
| **WebSearch para docs recentes** | Lib/framework mais nova que o cutoff do modelo |
| **Ler source code de libs** | Precisa entender como algo funciona internamente |
| **Analisar issues do GitHub** | Quer saber problemas comuns de uma abordagem |
| **Consultar CHANGELOG** | Precisa de features especificas de uma versao |

### Template do PRD.md

Consulte `references/templates.md` secao "PRD" para o template completo.

O PRD deve ser **conciso** — um resumo do que foi pesquisado, nao um despejo de tudo que foi lido.
Se o PRD passar de 3-4 paginas, esta longo demais. Resuma mais.

### Ao terminar

1. Gere o `docs/PRD.md` com todo o conhecimento acumulado
2. Revise com o usuario: "O PRD esta completo? Falta algo?"
3. **Limpe o contexto** — `/clear` no Claude Code, nova conversa no Cowork
4. Na proxima conversa, comece referenciando o PRD

---

## Etapa 2: Planejamento (Spec) — CONVERSA 2

**Objetivo:** Transformar o PRD em uma especificacao tecnica com arquitetura e decisoes.
**Input:** `docs/PRD.md`
**Output:** `docs/SPEC.md`
**Duracao tipica:** 1 conversa

### O que fazer nesta etapa

1. **Ler o PRD.md** como ponto de partida
2. **Definir arquitetura** — componentes, fluxo de dados, integracoes
3. **Tomar decisoes tecnicas** — stack, libs, patterns, estrutura de pastas
4. **Definir contratos** — APIs, modelo de dados, interfaces entre componentes
5. **Identificar riscos** — o que pode dar errado e como mitigar

A SPEC e o "raciocinio tecnico" — ela explica COMO o projeto vai ser construido.
Ela nao precisa listar tarefas ainda (isso e a Etapa 3).

### Template do SPEC.md

Consulte `references/templates.md` secao "SPEC" para o template completo.

A spec deve ser **tatica e precisa**. Se voce nao deixar claro o que quer, a IA vai implementar
do jeito dela — e voce vai reclamar que saiu errado. Quanto mais especifica a spec, melhor o codigo.

### Ao terminar


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jraugustos/project-workflow](https://github.com/jraugustos/project-workflow) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
