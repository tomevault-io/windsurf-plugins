---
trigger: always_on
description: Extensão Chrome (Manifest V3, JavaScript puro, **sem build step**) que adiciona um painel
---

# PJe IA — Extensão Chrome

Extensão Chrome (Manifest V3, JavaScript puro, **sem build step**) que adiciona um painel
de chat com Claude à tela de autos digitais do PJe. O usuário seleciona peças do
processo e conversa sobre elas; os PDFs são enviados diretamente à API da Anthropic.

## Arquitetura

**Multi-PJe (default-on)**: `content_scripts`, `host_permissions` e
`web_accessible_resources` cobrem `https://*.jus.br/*` — qualquer tribunal
funciona sem nenhuma ação do usuário (decisão de produto: zero fricção; o
aviso de permissão do Chrome fica mais amplo, aceito). Como o script roda em
TODA página jus.br (login SSO, portais…), o boot do painel em `content.js`
vive em `iniciar()`, chamada só quando `#divTimeLine` existe (ou surge — SPA
do PJe novo) — sem timeline, nada é injetado no DOM. O grau e o base path
variam por tribunal (`pje.tjce.jus.br/pje1grau`, `pje1g.trf5.jus.br/pje`…):
`pje.js` deriva o base path da URL (`getBase`). `DOMINIOS_JURIDICOS` ganha o
domínio-raiz do tribunal atual em runtime (busca de jurisprudência).

Content scripts injetados nesta ordem
(cada um é um IIFE que expõe um global — não há imports entre content scripts):

| Arquivo | Global | Papel |
|---|---|---|
| `src/pje.js` | `PJE` | Acesso ao PJe: lista peças da timeline (`#divTimeLine`), baixa cada uma pelo endpoint REST autenticado por cookie de sessão. |
| `src/panel.js` | `PjePanel` | Toda a UI (chat, seletor de peças, chips, popup `@`, card de progresso), isolada em **Shadow DOM**. CSS carregado de `src/panel.css` via `web_accessible_resources`. |
| `src/content.js` | — | Orquestração: downloads com concorrência 3, cache por peça, montagem dos blocos da API, conversa multi-turno, streaming via `Port`. |

O worker (`src/background.js` + `src/claude.js`, ES modules) guarda a chave da API e faz o
streaming SSE — **a chave nunca chega ao contexto da página**. Dois canais content↔worker:

- **Port** `chrome.runtime.connect({name:"claude"})` para os turnos (streaming). Tipos
  content→worker: `chat` e `gerarDoc`; worker→content: `delta`, `thinking`, `citation`,
  `tool`, `file`, `trunc`, `iter` (início de request físico — checkpoint da UI),
  `retry` (re-tentativa transitória — a UI reverte ao checkpoint para não duplicar
  texto/citações), `done {content, stopReason}`, `error`. **AUTO-RESUME**: se a porta
  cair SEM `done`/`error` (worker MV3 morto no meio do turno — acontece mesmo com
  keepalive), `stream()` em content.js reconecta e REENVIA o payload sozinho (até 2
  vezes; o turno é stateless e o prefixo está no cache de prompt). O handler
  `onReinicio` zera TODO o estado de UI do turno (o novo stream re-emite do zero).
  Não transformar esse reenvio em erro imediato — era a causa nº 1 de ".docx falha
  às vezes" no Haiku.
- **`chrome.runtime.sendMessage`** (request/response) para `caps` (capacidades do modelo),
  `upload` (Files API) e `countTokens` (pré-voo gratuito).

## Fluxo de um turno (protocolo v2)

`claude.js` acumula os **blocos completos** da resposta a partir do SSE (padrão dos SDKs:
`content_block_start/delta/stop`, incluindo `signature_delta` do thinking, `citations_delta`
e `input_json_delta`) e emite `{kind:"final", content, stopReason, containerId}`.
`background.js` resolve sozinho as continuações de **`pause_turn`** (reenvia
`messages + [{role:"assistant", content: parcial}]`, reutilizando `container.id` quando há
skills; máx. 8 iterações no chat e 16 na geração de documento — `payload.maxIter`) — o
content script enxerga um único turno lógico. **Erros transitórios re-tentam sozinhos**:
cada request físico ganha até 2 re-tentativas com backoff (429 espera 10 s) quando o
erro é 429/529/5xx ou queda de rede no meio do SSE (flag `retryable` posta pelo
`claude.js`; janela típica: os longos silêncios do code execution no docx). Se a
geração de documento terminar sem arquivo com `stop_reason` `pause_turn` (teto de
iterações) ou `max_tokens`, o worker LANÇA erro claro em vez de retornar em silêncio —
o Haiku precisa de mais rodadas de code execution que o Sonnet e era onde o docx
"falhava às vezes" sem explicação. `maxTokens` do docx é 32000 (16000 truncava o
código do Haiku no meio).

`MODEL_CAPS` em `background.js` governa por modelo: `contextTokens`, `maxPages` (600 nos
modelos de 1M; 100 no Haiku), versões de `web_search`/`web_fetch` (variantes `_20260209`
no Sonnet 5/Opus 4.8; básicas no Fable/Haiku), `thinking` (adaptive+summarized; omitido no
Haiku) e `effort` (não suportado no Haiku).

## Invariantes importantes

- **Assistant no histórico é SEMPRE array de blocos** (`response.content` completo), nunca
  string: a API exige thinking assinado intacto e os blocos de ferramenta/citações nos
  turnos seguintes. Em fallback (sem blocos), texto puro com os placeholders de citação
  removidos. **Citações NUNCA voltam à API**: a resposta traz campos que o request
  rejeita (`file_id` em `page_location` → 400 "Extra inputs are not permitted") e,
  pior, a API revalida os `document_index` contra o layout do request atual — com o
  anexo incremental essa revalidação falha (400 "Invalid citation indices: Document
  not found for placeholder citation", sempre na 2ª mensagem). Por isso o campo
  `citations` é REMOVIDO dos blocos de texto do assistant antes de qualquer reenvio:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marcosmarf27/pje-ia](https://github.com/marcosmarf27/pje-ia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-20 -->
