---
trigger: always_on
description: Este arquivo é lido automaticamente por sessões do Claude Code neste
---

# CLAUDE.md — Contexto do Projeto para Claude Code

Este arquivo é lido automaticamente por sessões do Claude Code neste
repositório. Contém o conhecimento acumulado de meses de desenvolvimento
deste projeto — leia antes de mexer no código, especialmente as seções
de **armadilhas recorrentes**.

## O que é este projeto

**ControleOrcamento** (nome de exibição: "BudgetDev | Gestão Orçamentária")
é um web app de controle orçamentário/financeiro rodando inteiramente em
**Google Apps Script (GAS)**, com **Google Sheets** como banco de dados.
Não há backend externo, não há build step, não há framework de frontend —
é HTML/CSS/JS puro servido via `HtmlService`.

Domínio: controle de notas fiscais, fornecedores, prestadores de serviço
(consultorias/squads alocados em iniciativas de projeto), forecast
orçamentário por iniciativa, e dashboards de acompanhamento.

## Stack e não-stack

- **Backend**: arquivos `.gs` (Google Apps Script = JavaScript no runtime V8 do Google).
- **Frontend**: arquivos `.html` servidos via `HtmlService.createTemplateFromFile`,
  com `<script>`/`<style>` inline. Cada `Page_*.html` e `Card_*.html` é um
  módulo autocontido (IIFE) com seu próprio CSS, HTML e JS no mesmo arquivo.
- **Sem** bibliotecas externas de gráficos — todos os charts são SVG puro
  desenhado à mão em JS (ver `Page_Home.html`, funções tipo `_hxScale`).
- **Sem** build step, sem bundler, sem npm no runtime. `npm`/`clasp` existem
  só como ferramenta de deploy (ver `docs/DEPLOYMENT.md`).
- **Sem** testes automatizados. "Testar" = `clasp push` para um deployment
  de teste e clicar na UI manualmente.
- **Sem** framework — tudo é JS vanilla com padrão de módulo `var X = (function(){ ... })();`.

## Documentação relacionada

- `docs/ARCHITECTURE.md` — mapa de arquivos, camadas (Router/DAO), schema das
  abas do Sheets, convenções de frontend.
- `docs/DEPLOYMENT.md` — como configurar deploy automático via `clasp` +
  GitHub Actions (push para `main` → publica no Apps Script).

## Fluxo de trabalho de Git usado neste projeto

- **Merge direto na `main`** — não há branch `develop`/`staging`. Toda PR
  tem `main` como base.
- Uma feature/ajuste = uma branch curta = uma PR pequena e objetiva
  (o histórico deste repo tem dezenas de PRs pequenos, um por ajuste pedido).
- Depois que a PR é mergeada, se o deploy automático (`docs/DEPLOYMENT.md`)
  estiver configurado, o `push` para `main` dispara `clasp push` +
  `clasp deploy` automaticamente — não é necessário fazer deploy manual.
- Sem esse workflow configurado, deploy é manual: `clasp push` (ou colar o
  código no editor do Apps Script).

## Armadilhas recorrentes (leia antes de codar)

Estas são causas reais de bugs que já se repetiram várias vezes nesta base
de código. Preste atenção especial nelas.

### 1. Regex de acentos — NUNCA usar caractere combinante literal

Qualquer função de normalização de texto (remover acentos, comparar nomes)
deve usar a forma **ASCII-escapada**:

```js
// CERTO — o range Unicode de combining marks (U+0300 a U+036F) é
// referenciado só por código de escape, nunca digitando o caractere em si.
// Esta é a forma usada em DAO.gs (normalizeKey) — copie literalmente
// dali sempre que precisar de uma função parecida.
const semAcento = str.normalize('NFD').replace(new RegExp('[\\u0300-\\u036f]', 'g'), '');
```

**Errado**: um regex literal (`/[...]/`) contendo o caractere combinante
Unicode de verdade dentro dos colchetes, em vez do código de escape acima.
Costuma ser introduzido sem querer ao colar código de um editor ou ao
gerar código com ajuda de IA sem revisar o resultado. O arquivo parece
salvar normalmente e o bug só aparece depois, quando o pipeline de
edição corrompe o byte — manifestando como "o filtro de Iniciativa veio
vazio", bug que já se repetiu pelo menos 3 vezes nesta base. Regra prática:
**todo `replace(/[...]/g, ...)` que mexe com acentuação deve usar
`new RegExp('[\\u0300-\\u036f]', 'g')` com o range escrito em `\uXXXX`,
nunca um literal `/[...]VOGAL_ACENTUADA_COLADA.../`.**

Isso vale tanto em `.gs` (`normalizeKey` em `DAO.gs`) quanto em qualquer
`.html` que normalize nomes/colunas.

### 2. `_route()` acha o resultado na RAIZ da resposta, não em `res.data`

Em `Router.gs`, todo endpoint é envolvido por `_route(handler)`:

```js
function _route(handler) {
  const result = handler();
  return Object.assign({ success: true, metrics: {...} }, result);
}
```

Ou seja, se o handler retorna `{ rows, months }`, a resposta final do
`google.script.run` é `{ success, metrics, rows, months }` — **os campos
ficam na raiz**, não em `res.data.rows`. Isso já causou bugs de
"undefined" no frontend múltiplas vezes. Sempre confira o que o handler
específico retorna antes de acessar `res.data.X` — na maioria dos casos é
`res.X` diretamente.

### 3. V8 do Apps Script não aceita function declaration dentro de bloco

```js
// ERRADO — SyntaxError em runtime V8 do GAS dentro de try/if
if (cond) {
  function minhaFuncao() { ... }
}

// CERTO
if (cond) {
  const minhaFuncao = function() { ... };
}
```

### 4. `ALLOWED_SHEETS` é uma whitelist obrigatória

Qualquer endpoint genérico de CRUD (`routerGetSheet`, `routerInsertRow`,
`routerSaveComponent`, etc. em `Router.gs`/`ComponentHelper.gs`) valida a

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brunotrolo-bank/ControleOrcamento](https://github.com/brunotrolo-bank/ControleOrcamento) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-29 -->
