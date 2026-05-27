---
trigger: always_on
description: > Instruction file lido automaticamente pelo **GitHub Copilot Chat** e **Copilot Workspace / Agent Mode**. Espelha [AGENTS.md](../AGENTS.md) com foco em **Agent Mode workflow**.
---

# Copilot Instructions

> Instruction file lido automaticamente pelo **GitHub Copilot Chat** e **Copilot Workspace / Agent Mode**. Espelha [AGENTS.md](../AGENTS.md) com foco em **Agent Mode workflow**.
>
> Ao trabalhar em Agent Mode, o Copilot pode delegar pra custom agents em [`.agents/`](../.agents/) (canônico, padrão AGENTS.md ecosystem) e/ou em `.github/copilot/agents/` (mirror lido pelo Copilot Coding Agent). Lista atual: `tdd.agent.md`, `reviewer.agent.md`, `architect.agent.md`.

---

## Stack

`<STACK>` (placeholder — substitui pela stack real do projeto, ex: `Node.js 20 + TypeScript + Next.js 14 + Playwright + Vitest`).

- Linguagem principal: `<STACK>`
- Framework web/API: `<STACK>`
- Banco de dados: `<STACK>`
- Test runner unit: `<STACK>` (Vitest, Jest, pytest, xUnit)
- Test runner E2E: **Playwright** (config em `playwright.config.ts`)
- Linter/formatter: `<STACK>` (ESLint + Prettier, Ruff, dotnet format)
- CI/CD: GitHub Actions (`.github/workflows/`)
- Deploy: `<STACK>` (ver `.specs/workflow/RELEASE.md`)

> Antes de adicionar dependência nova: pergunta ao humano. Sem exceção.

---

## Comandos importantes

```bash
# desenvolvimento
npm run dev
npm run build

# qualidade
npm run lint
npm run lint:fix
npm test
npm test -- --coverage

# E2E
npx playwright install
npx playwright test
npx playwright show-report

# git/PR
git checkout -b feat/<task-id>-<slug>
gh pr create --fill
gh run watch
```

Adapta pra `pnpm`, `yarn`, `bun`, `dotnet`, `python`, `go` conforme stack real.

---

## Padrão de sincronização deste projeto

Quando a mudança for **release-relevant**, o padrão deste repositório é fechar o trabalho com tudo sincronizado no mesmo ciclo:

- npm publicado na mesma versão de `package.json`
- tag GitHub `vX.Y.Z`
- GitHub Release correspondente
- `main` limpa e sincronizada com `origin/main`

Validação obrigatória antes de publicar/sincronizar:

```bash
npm run lint
npm test
npm run docs:build
npm run test:e2e -- --reporter=list,html
```

Se qualquer comando falhar, não publique e não crie a release/tag.

---

## Workflow loop OBRIGATÓRIO (Agent Mode)

Em Copilot Workspace/Agent Mode, todo plano de execução segue esse loop. Não pula etapa.

1. **Ler task** — abre `.specs/sprints/sprint-XX/<task-id>.task.md`. Lê contexto + acceptance criteria + test plan + DoD.
2. **Plano explícito** — Copilot Workspace gera spec/plan. Revisa antes de implementar.
3. **Carregar contexto** — `.specs/architecture/PATTERNS.md` + ADRs relevantes em `.specs/architecture/ADR-*.md`. Skills aplicáveis em `.skills/`.
4. **Implementar (Agent Mode)** — edits cirúrgicos. Só toca o que a task pede. Sem refactor extra.
5. **Lint** — `npm run lint`. Vermelho = corrige.
6. **Unit** — `npm test`. Vermelho = corrige. Coverage do diff >= 80%.
7. **E2E (OBRIGATÓRIO em TODA task)** — `npx playwright test --reporter=list,html`. Captura **trace + screenshot + video** (todos). Sem evidência em `playwright-report/` + `test-results/` = task não fechada.
8. **Fix loop** — falhou? Volta ao 4. Repete até verde.
9. **Commit** — Conventional Commits (`feat:`, `fix:`, `chore:`, `docs:`). Mensagem em **inglês**.
10. **PR** — `gh pr create --fill`. Preenche template inteiro.

---

## Definition of Done

PR só faz merge quando todos os itens abaixo estão marcados:

- [ ] Unit tests passam
- [ ] Lint passa
- [ ] E2E Playwright passa **com evidência anexada em TODA task** — `playwright-report/index.html` + `test-results/<spec>/trace.zip` + screenshots por cenário + video. Hard rule: sem evidência, sem merge.
- [ ] Coverage do diff >= 80%
- [ ] Acceptance Criteria todos marcados
- [ ] PR template preenchido (link task + descrição + evidências)
- [ ] Conventional commit no merge
- [ ] ADR criado se mudou decisão arquitetural
- [ ] Changelog atualizado se release-relevant
- [ ] Sem warning novo, sem `console.log`/`print` deixado pra trás
- [ ] Sem TODO sem dono e sem prazo

CI bloqueia merge se DoD falhar (`.github/workflows/dod.yml`).

---

## Padrões de código

`.specs/architecture/PATTERNS.md` é a **fonte única**. Naming, estrutura, criação de endpoint/componente/teste, tratamento de erro, logging, validação — tudo lá.

Decisões irreversíveis viram **ADR** em `.specs/architecture/ADR-XXX-*.md` (template em `.specs/architecture/ADR-template.md`).

---

## Onde encontrar contexto

| Pergunta | Onde olha |
|---|---|
| Por que esse produto existe? | `.specs/product/VISION.md` |
| Quem é o usuário? | `.specs/product/PERSONAS.md` |
| Quais entidades de negócio? | `.specs/product/DOMAIN.md` |
| Como o sistema é desenhado? | `.specs/architecture/DESIGN.md` |
| Como escrever código aqui? | `.specs/architecture/PATTERNS.md` |
| Por que decidimos X? | `.specs/architecture/ADR-*.md` |
| Como faço PR/branch/release? | `.specs/workflow/WORKFLOW.md`, `RELEASE.md`, `CONTRIBUTING.md` |
| Backlog? | `.specs/sprints/BACKLOG.md` |
| Sprint atual? | `.specs/sprints/sprint-XX/SPRINT.md` |
| Skills? | `.skills/README.md` + `.skills/*/SKILL.md` |

---

## Proibido

- **Pular testes** — sem unit/E2E = sem merge.
- **Mockar pra fazer passar** — mock só pra dep externa real (HTTP, DB), nunca pra esconder falha.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wesleysimplicio/simplicio-cli](https://github.com/wesleysimplicio/simplicio-cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-26 -->
