---
trigger: always_on
description: Convenções do agente para a esteira SDD. Sempre ativo.
---


# CLAUDE.md — Convenções para agentes de IA

Este projeto segue **Spec-Driven Development (SDD)**. Leia antes de implementar qualquer coisa.

## Início de sessão — carregue o contexto base
> Um hook **`SessionStart`** (`.claude/settings.json` → `.claude/hooks/load-context.mjs`) injeta
> este contexto base **automaticamente** (garantia determinística). Se o hook estiver desativado,
> esta diretiva é o fallback — e como o `CLAUDE.md` é sempre carregado, ela cobre o caso.

**Garanta o contexto base antes da primeira tarefa — os docs `alwaysApply: true`:**
`docs/STATE.md` · `docs/product/vision.md` · `docs/product/roadmap.md` · e a `spec.md` da
feature ativa em `specs/`.

Todos os outros docs são `alwaysApply: false` — **não os leia agora**. Puxe cada um **sob demanda**,
quando a tarefa exigir, guiado pelo `description` no frontmatter dele. Para **achar onde algo mora**,
o índice navegável é `docs/README.md` (mapa do projeto).

## A spec é a fonte da verdade
- Implemente **a partir de** `specs/NNNN-*/spec.md`. Os critérios de aceite
  (Given/When/Then) são o contrato e o oráculo de teste.
- Se a spec for ambígua ou estiver errada, **pare e pergunte** — não adivinhe.
  Atualizar a spec é uma decisão consciente, não um efeito colateral do código.
- Nunca implemente além do escopo da spec. "Out of scope" é vinculante.

## Verificação de conhecimento (nunca invente)
Antes de afirmar como algo funciona, siga esta ordem — pare assim que tiver a resposta:
1. **Padrões do próprio codebase** (como já é feito aqui).
2. **Docs do projeto** (`specs/`, `docs/`, ADRs, glossário).
3. **MCP** de referência (ex.: Context7 para libs) quando conectado.
4. **Web/doc oficial** da tecnologia.
5. **Não encontrou? Diga "não sei" e sinalize.** Nunca invente API, padrão ou comportamento —
   inventar causa falha em cascata. Incerteza explícita é melhor que um chute confiante.

## Ferramentas conectadas (MCP)
> **Mantido pela skill `/integracoes`.** Lista os MCP servers validados e quem os consome, para o
> roteamento de skills/rules. Vazio até a primeira conexão — rode `/integracoes` para preencher.

| MCP (`mcp__<servidor>__*`) | Conta/workspace validada | Skills que consomem |
|---|---|---|
| _nenhum ainda_ | — | — |

Regra: conexão ativa **não** autoriza uso. Confirme a conta/workspace antes de ler e **reconfirme
antes de qualquer escrita** (ver `/integracoes`). Só use um MCP presente na sessão (`mcp__<servidor>__*`).

## Antes de codar — descubra o tier
Pergunta: *isso introduz decisão difícil de reverter ou nova fronteira de domínio?*
- **Trivial** (≤3 arquivos, sem decisão): só o PR (ou `quick/` se quiser deixar rastro).
- **Pequeno** (feature isolada, <10 tasks): exige `spec.md` + `tasks.md`.
- **Arquitetural** (novo bounded context, integração externa, decisão irreversível):
  exige `design.md` aprovado **antes** de implementar. Se não existir, pare e sinalize.

> **Escalonamento dinâmico:** mesmo quando `tasks.md` é dispensado, **sempre liste os passos
> atômicos inline antes de codar**. Se a lista passar de ~5 passos ou tiver dependências
> complexas, **PARE e crie um `tasks.md` formal** — o tier real era maior do que parecia.

## Linguagem ubíqua
- Use **exatamente** os termos de `docs/glossary.md` e do `domain.md` da feature.
  Mesmo nome no código, na spec e na conversa com o time. Não invente sinônimos.
- Termo novo → adicione ao glossário no mesmo PR.

## Arquitetura em camadas (regra de dependência)
`src/` segue DDD tático. A dependência aponta **só para dentro**:

```
interfaces → application → domain ← infrastructure
```

- `domain/` não importa NADA de framework, I/O, ou de outras camadas.
- `application/` orquestra casos de uso; depende só de `domain/`.
- `infrastructure/` implementa as portas definidas no domínio (repos, adapters).
- `interfaces/` é a borda (API/CLI/UI).

## Disciplina de contexto e delegação
Cada doc declara no frontmatter sua política de carregamento (padrão estilo Cursor rules):
- `alwaysApply: true` — **contexto base**, leia em toda sessão.
- `alwaysApply: false` — **sob demanda**; o campo `description` diz **quando** puxá-lo.

**Base (`alwaysApply: true`):** este `CLAUDE.md` · `docs/STATE.md` · `docs/product/vision.md` ·
`docs/product/roadmap.md` · a `spec.md` da feature ativa. Todo o resto é sob demanda — puxe
pelo `description` quando a tarefa exigir (TESTING, glossary, context-map, ADRs, integrations…).

> **Dois dialetos de frontmatter:** os **docs da esteira** usam `name`+`description`+`alwaysApply`;
> as **skills** (`.claude/skills/*/SKILL.md`) e os **templates de skill/subagente** usam o dialeto
> do alvo (`name`+`description`, sem `alwaysApply`) — porque são copiados para virar skills/agentes.

- **Carregue sob demanda:** não leia o repo inteiro — puxe o on-demand conforme a task atual.
- **Delegue para manter o contexto enxuto:** pesquisa e tasks paralelas (`[P]`) vão para
  subagentes (ver `docs/engineering/_templates/subagent.template.md`), que recebem só a task +
  spec + TESTING e devolvem um report-back estruturado. Reserve a janela para o trabalho.

### Orçamento de contexto (alvo)
- **Base (`alwaysApply: true`): ~15k tokens.** Mantenha enxuto — se crescer, é sinal de mover
  detalhe para docs on-demand.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [igoruehara/spec-driven](https://github.com/igoruehara/spec-driven) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
