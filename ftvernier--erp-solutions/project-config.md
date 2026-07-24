---
trigger: always_on
description: Instruções permanentes para o Claude Code neste projeto.
---

# CLAUDE.md — PO-UI Protheus Angular

Instruções permanentes para o Claude Code neste projeto.
Leia este arquivo antes de qualquer tarefa de desenvolvimento Angular/PO-UI.

---

## Stack do projeto

| Item | Versão |
|---|---|
| Node.js | 20.11.x ou superior |
| Angular | ~21.2.4 (NgModules — **NUNCA** `standalone: true`) |
| @angular/cli | ~21.2.4 |
| @po-ui/ng-components | 21.x |
| @po-ui/ng-templates | 21.x |
| @totvs/po-theme | 21.x |
| @totvs/protheus-lib-core | 21.x |
| RxJS | ~7.8.1 |
| TypeScript | ~5.9.3 |
| zone.js | ~0.15.0 |
| SubSink | 1.x |

> **Adapte as versões acima para as do seu `package.json` antes de usar.**
> A partir da v14, o número de versão major do **PO UI acompanha o do Angular** (Angular 21 → PO UI 21.x). Consulte a [tabela de compatibilidade](https://github.com/po-ui/po-angular/wiki) se o projeto estiver em versão mais antiga.

### Observação sobre o build system (Angular 19+/21+)

Projetos recentes podem usar o novo builder `@angular/build:*` em vez do tradicional `@angular-devkit/build-angular`. Antes de investigar um erro de build/serve, **verifique no `angular.json` qual builder está configurado**:

```json
"architect": {
  "build": { "builder": "@angular/build:application" },
  "serve": { "builder": "@angular/build:dev-server" }
}
```

Se o builder for `@angular/build:*`, o pacote `@angular/build` precisa estar instalado (`npm install -D @angular/build`). Erro comum: `Could not find the @angular/build:dev-server builder's package`.

---

## Passo 0 — Setup e scaffolding (use os schematics oficiais antes de criar manualmente)

Antes de escrever uma feature do zero, prefira gerar a base via CLI/schematics e depois aplicar as convenções deste documento por cima.

```bash
# Adicionar o PO UI a um projeto existente (configura tema, módulo PO e HttpClientModule)
ng add @po-ui/ng-components

# Adicionar templates dinâmicos
ng add @po-ui/ng-templates
```

### Schematics de componentes (`@po-ui/ng-components`)

| Schematic | Descrição |
|---|---|
| `po-page-list` | Container principal para telas de listagem (lista ou tabela) |
| `po-page-default` | Container principal para telas sem template definido |
| `po-page-edit` | Container para tela de edição/inclusão de registro |
| `po-page-detail` | Container para tela de detalhamento de registro |

### Schematics de templates dinâmicos (`@po-ui/ng-templates`)

| Schematic | Descrição |
|---|---|
| `po-page-dynamic-table` | Lista de registros em tabela baseada em metadados/fields |
| `po-page-dynamic-edit` | Edição/criação de registros baseada em metadados |
| `po-page-dynamic-detail` | Exibição de detalhes baseada em metadados |
| `po-page-dynamic-search` | Tela de pesquisa com ações já definidas |
| `po-page-job-scheduler` | Tela de agendamento de processos (Job Scheduler) |
| `po-page-login` | Template de tela de login |
| `po-page-change-password` | Template de cadastro/alteração de senha |
| `po-page-blocked-user` | Template de tela de bloqueio de usuário |

Uso: `ng generate <package>:<schematic-name>`. Exemplo:

```bash
ng generate @po-ui/ng-templates:po-page-dynamic-table
```

### Atualizando o PO UI (`ng update`)

```bash
# 1. Atualize o Angular primeiro
npm install @angular-devkit/schematics --save-dev   # necessário em Angular 20+
ng update @angular/cli@21 @angular/core@21 --force

# 2. Depois atualize o PO UI
ng update @po-ui/ng-components --allow-dirty --force
```

Se o `ng update` falhar: apague `node_modules` e `package-lock.json`, rode `npm i --legacy-peer-deps`, e tente novamente. O processo pode perguntar se deseja migrar para a nova biblioteca de ícones — decida conscientemente, não aceite por padrão sem avaliar o impacto visual.

Antes de subir uma versão major, consulte:
- [Guia de Depreciações](https://po-ui.io/guides/deprecations)
- [CHANGELOG no GitHub](https://github.com/po-ui/po-angular/blob/master/CHANGELOG.md)

---

## Passo 1 — Buscar documentação PO-UI antes de escrever código

**Antes de implementar qualquer componente, busque a referência oficial.** Use o MCP do PO-UI como fonte primária; recorra ao `web_fetch` apenas se o MCP não estiver disponível no ambiente atual.

### Fonte primária — MCP `@po-ui/mcp`

O PO-UI oferece um servidor MCP oficial. Se as ferramentas abaixo estiverem disponíveis na sessão, **use-as em vez de `web_fetch`** — são mais confiáveis (sem necessidade de montar/adivinhar slug de URL) e têm busca textual nativa.

| Ferramenta | Uso |
|---|---|
| `list_components` | Lista componentes, serviços, interfaces e enums (com filtro por seção/texto). Útil para descobrir o que existe antes de implementar algo novo ou verificar se já há um componente pronto para o caso. |
| `get_component_docs` | Documentação completa em Markdown de um componente/API pelo slug. Use sempre antes de implementar ou alterar uso de um componente específico. |
| `search_docs` | Busca texto livre em toda a documentação. Ideal quando não se sabe o nome exato do componente (ex.: "componente com upload de arquivo"). |
| `get_guide` | Conteúdo de guias (ex.: `getting-started`, `schematics`, `migration-poui`). |

**Configuração** (não exige instalação — executa via `npx`):

```json
{
  "mcpServers": {
    "po-ui": {
      "command": "npx",
      "args": ["-y", "@po-ui/mcp"]
    }
  }
}
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ftvernier/erp-solutions](https://github.com/ftvernier/erp-solutions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
