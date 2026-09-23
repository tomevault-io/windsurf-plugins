---
trigger: always_on
description: Antes de qualquer análise, proposta, edição, execução de teste, geração de documento ou implementação neste repositório, o agente deve ler e respeitar este `AGENTS.md`.
---

# AGENTS.md

## Regra Zero — Leitura Obrigatória

Antes de qualquer análise, proposta, edição, execução de teste, geração de documento ou implementação neste repositório, o agente deve ler e respeitar este `AGENTS.md`.

Esta leitura não é opcional e não pode ser substituída por memória de conversas anteriores.

### Verificação de Bootstrap (obrigatória, antes de tudo)

Ler `.ai/context/project-status.md`.

* Se `bootstrap: pending` ou `bootstrap: in_progress` → **a única ação permitida é executar `$bootstrap-project`**. Nenhuma Intent, PRD, análise ou implementação pode começar. Este repositório ainda é um harness vazio: ele não sabe qual é o projeto, a arquitetura ou a stack.
* Se `bootstrap: done` → prosseguir normalmente com este documento.

Depois de confirmar que o bootstrap está concluído, consultar também, conforme a natureza da tarefa:

* `.ai/context/business.md`
* `.ai/context/architecture.md`
* `.ai/context/coding-standards.md`
* `.ai/context/testing-standards.md`
* A skill aplicável em `.agents/skills/<skill>/SKILL.md`

Se houver conflito entre código existente e os documentos oficiais, a ordem de verdade é:

```text
AGENTS.md
  ↓
Intent
  ↓
PRD
  ↓
ADR
  ↓
Código
```

O código existente nunca justifica violar uma regra arquitetural documentada.

---

## Missão

Este repositório é um **harness genérico** de desenvolvimento assistido por IA: um processo seguro e rastreável (Intent → PRD → Impact Analysis → Reviews → Implementação → Testes → Validação) que se aplica a qualquer projeto de software, mais um conjunto de skills que o agente aciona em cada etapa.

Na primeira execução ele não conhece nada sobre a aplicação — isso é resolvido pela skill `$bootstrap-project`, que entrevista o usuário e preenche `.ai/context/*.md` com o contexto real do projeto.

<!-- BOOTSTRAP:PROJECT-STRUCTURE -->
### Estrutura do Projeto

*(preenchido pelo `$bootstrap-project` com a lista real de módulos/repositórios e o papel de cada um — ver `.ai/context/architecture.md` para o detalhamento completo)*
<!-- /BOOTSTRAP:PROJECT-STRUCTURE -->

---

## Estrutura de Governança

```text
.agents/
└── skills/
    ├── bootstrap-project/
    ├── orchestrator/
    ├── grill-me/
    ├── grilling/
    ├── domain-modeling/
    ├── to-tickets/
    ├── resolving-merge-conflicts/
    ├── create-intent/
    ├── create-prd/
    ├── impact-analysis/
    ├── architecture-review/
    ├── business-review/
    ├── security-review/
    ├── observability-review/
    ├── generate-tests/
    ├── implementation-review/
    ├── validate-delivery/
    └── reintegrar-main/

.ai/
├── templates/
│   ├── intent_template.md
│   ├── prd_template.md
│   ├── impact_analysis_template.md
│   ├── test_plan_template.md
│   ├── validation_report_template.md
│   └── adr_template.md
│
└── context/
    ├── project-status.md
    ├── business.md
    ├── architecture.md
    ├── coding-standards.md
    └── testing-standards.md

intent/
└── <slug>.md, <slug>_prd.md, <slug>_impact.md, <slug>_test_plan.md, <slug>_*-review.md, <slug>_validation_report.md

docs/
└── adr/
    ├── README.md
    └── ADR-XXXX-<slug>.md
```

---

## Fluxo Obrigatório

Toda solicitação de desenvolvimento segue obrigatoriamente (coordenado por `$orchestrator`):

```text
IDEIA
  ↓
ENTENDIMENTO ($grilling / $grill-me)
  ↓
INTENT ($create-intent)
  ↓
PRD ($create-prd)
  ↓
IMPACT ANALYSIS ($impact-analysis)
  ↓
ARCHITECTURE REVIEW ($architecture-review)
  ↓
BUSINESS REVIEW ($business-review)
  ↓
TICKETS, quando a entrega exigir mais de uma integração segura na main ($to-tickets)
  ↓
IMPLEMENTAÇÃO
  ↓
TESTES ($generate-tests)
  ↓
IMPLEMENTATION REVIEW ($implementation-review)
  ↓
SECURITY REVIEW ($security-review)
  ↓
OBSERVABILITY REVIEW ($observability-review)
  ↓
REINTEGRAÇÃO NA MAIN ($reintegrar-main)
  ↓
VALIDAÇÃO PRD × ENTREGA ($validate-delivery)
  ↓
ENTREGA
```

Para bugs pequenos e correções operacionais de baixo risco, `$orchestrator` define uma Matriz de Decisão com um caminho simplificado — o rigor é proporcional ao risco, não burocracia por burocracia.

---

## Estratégia Obrigatória de Execução

### Isolamento de Contexto

Toda solicitação que gere Intent deve ser tratada como uma frente isolada.

* Não misturar duas Intents na mesma entrega.
* Não reaproveitar PRD de outra demanda para justificar implementação atual.
* Não alterar arquivos fora do escopo da Intent/PRD, salvo correção indispensável documentada no Impact Analysis.
* Se a ferramenta permitir criar/usar thread ou sessão separada, cada Intent deve rodar na sua própria.
* Se não permitir, manter o slug da Intent como fronteira rígida de contexto e registrar qualquer desvio no Validation Report.

### Entendimento Obrigatório

Antes de criar uma Intent, toda solicitação passa por uma entrevista de entendimento (`$grilling`, ou `$grill-me` para pedidos não técnicos). Fatos verificáveis no repositório/ferramentas devem ser investigados pelo agente; o usuário só decide o que depende de preferência, prioridade, negócio ou design. Não criar Intent, PRD, Impact Analysis, ADR ou implementação enquanto houver decisão aberta ou sem confirmação de entendimento compartilhado.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [fabiogodoy/godharness](https://github.com/fabiogodoy/godharness) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
