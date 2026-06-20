---
trigger: always_on
description: |
---


# Project Maker

Workflow estruturado de Spec-Driven Development para construir projetos com IA sem vibe coding.

## Escala do problema (auto-sizing)

**Princípio:** a complexidade determina a profundidade, não o contrário. Antes de começar qualquer trabalho, avalie o escopo e aplique apenas o necessário.

| Escopo | O que é | Discover | Init | Spec | Break | Plan | Execute |
|---|---|:-:|:-:|:-:|:-:|:-:|:-:|
| **Small** (`--quick`) | ≤3 arquivos, uma frase | — | — | — | — | ⚠️ inline | ✅ issue isolada |
| **Medium** (`--feature`) | Feature clara, <10 issues, 1 sprint | — | ⚠️ | ✅ breve | ⚠️ inline | ✅ | ✅ sprint único |
| **Large** (`--feature-large`) | Multi-componente, 2-5 sprints | — | ✅ | ✅ completo | ✅ | ✅ | ✅ dual-agent |
| **Complex** (`--epic`) | Ambiguidade, domínio novo | ✅ | ✅ | ✅ + discuss | ✅ + research | ✅ | ✅ + UAT |

**Etapas de fechamento** (após o execute, quando há entrega real):

| Escopo | Verify | Secure | Ship |
|---|:-:|:-:|:-:|
| **Small** (`--quick`) | ⚠️ se user-facing | ⚠️ se toca auth/dados/input | ⚠️ se há remote |
| **Medium** (`--feature`) | ✅ se user-facing | ✅ | ✅ |
| **Large** (`--feature-large`) | ✅ | ✅ | ✅ |
| **Complex** (`--epic`) | ✅ | ✅ | ✅ |

**Legenda:** ✅ obrigatório · ⚠️ condicional/inline · — pulado

**Regras de skip:**
- **Specify e Execute são sempre obrigatórios** — você sempre precisa saber O QUÊ e fazer
- **Discover** só em projeto novo (Complex)
- **Init** pulado se o projeto já tem steering/ e Constitution.md configurados
- **Design inline** (dentro do Spec) quando a mudança é direta — sem decisões arquiteturais, sem padrões novos
- **Break pulado** quando ≤3 passos óbvios — viram steps inline no Execute
- **Discuss** é disparado dentro do Spec **apenas** quando o agente detecta gray areas críticas
- **Interactive UAT** é disparado dentro do Execute **apenas** para features user-facing com comportamento complexo

**Safety valve:** mesmo quando `/break` é pulado, `/execute` SEMPRE começa listando os passos atômicos inline. Se essa lista revelar >5 passos ou dependências complexas, PARE e force criar sprint/issues formais — o Break foi pulado por engano.

**Auto-detecção:** se o usuário não especificou escopo, infira pelo contexto. "mudar um texto" → Small. "nova feature em projeto existente" → Medium. "adicionar módulo de cobrança com Stripe" → Large. "construir o produto do zero" → Complex.

## Fluxo completo

```
/discover            → brief.md                                (Complex)
/init                → steering/ + Constitution.md +
                        STATE.md + DECISIONS.md + KNOWLEDGE.md  (Medium+)
/spec [new|feature]  → Spec.md com EARS (+ context.md)         (Medium+)
/break               → research.md + data-model.md + contracts/
                        + sprints/ + issues/ + PRD.md           (Large+)
/plan [issue|sprint] → issue enriquecida                       (todos)
/execute [sprint]    → orquestra: waves → implementer → validator
                        → reassess → milestone gate → PRD        (todos)
/verify [sprint]     → UAT resumível + cold-start smoke +
                        loop diagnose→fix→re-verify → uat.md     (user-facing)
/secure [sprint]     → gate de segurança (delega security-review)
                        → SECURITY.md                            (auth/dados/input)
/ship [sprint]       → preflight → push → PR com body rico       (quando há remote)
/build [issue]       → atalho: plan + execute (issue isolada)   (Small/Medium)
/pause               → snapshot em STATE.md para próxima sessão (qualquer hora)
/resume              → retoma a partir de STATE.md              (qualquer hora)
```

**Loop de fechamento (após o sprint passar no milestone gate):** `/verify` (se user-facing) → `/secure` (se toca segurança) → `/ship`. O `/ship` só cria o PR se o milestone gate passou, o `/verify` não tem gaps abertos e o `/secure` não tem ameaças abertas.

**Hierarquia de trabalho (regra de ferro):**

```
Projeto → Sprint → Issue
```

- **Projeto** é o produto inteiro (definido em `PRD.md`, `Spec.md`, `steering/`).
- **Sprint** agrupa issues relacionadas com um Goal único e Success Criteria verificáveis.
- **Issue** é a unidade atômica de implementação. **Uma issue tem que caber em uma janela de contexto. Se não cabe, são duas issues.**

`/execute` roda um sprint por vez: para cada issue do sprint, dispara implementer → validator em loop até passar, e ao fim do sprint roda reassess + milestone gate.

**Artefatos persistentes (memory bank do projeto):**

Living docs — lidos no início, atualizados ao longo do trabalho:
- `STATE.md` — estado volátil (sessão atual, blockers, todos, handoff) — alto churn
- `DECISIONS.md` — decisões arquiteturais append-only — nunca deletar, baixo churn
- `KNOWLEDGE.md` — patterns, anti-patterns, gotchas reusáveis — cross-sprint
- `PRD.md` — estado vivo do produto com rastreabilidade de issues

Steering (contexto do projeto, estável):
- `steering/product.md` — visão, usuários, propósito do produto
- `steering/structure.md` — estrutura de diretórios, organização
- `steering/tech.md` — stack, versões
- `steering/architecture.md` — overview arquitetural, camadas, fluxos (Large/Complex)
- `steering/conventions.md` — naming, padrões de código, anti-padrões (Large/Complex)
- `steering/testing.md` — estratégia, comandos, coverage (Large/Complex)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [renatodpaula/project-maker](https://github.com/renatodpaula/project-maker) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
