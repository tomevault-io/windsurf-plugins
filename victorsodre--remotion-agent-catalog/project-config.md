---
trigger: always_on
description: Este arquivo é o guia para **copiar para o seu projeto Remotion** (agentes leem
---

# AGENTS.md

Este arquivo é o guia para **copiar para o seu projeto Remotion** (agentes leem
antes de escrever composição). Se você está *neste* repositório
(`remotion-agent-catalog`), o mapa do produto está no `README.md` e em
`docs/ARQUITETURA.md`. O `npm run catalog` citado abaixo é o **gerador da origem**
— não existe aqui.

O índice do que existe está em `catalog.json` (gerado — nunca edite à mão).

---

## As seis armadilhas

Todas foram descobertas quebrando alguma coisa aqui. Nenhuma dá erro claro — a maioria falha em silêncio.

### 1. `useVideoConfig()` reporta a COMPOSIÇÃO, não o container

Componentes que calculam tamanho de fonte, safe area ou posição percentual leem `useVideoConfig()`. Isso devolve as dimensões da `<Composition>`, **não** do elemento onde você os colocou.

Consequência: reduzir por `transform: scale()` encolhe o quadro mas **não o texto**, e tudo vaza.

- Para exibir uma cena 1920×1080 dentro de uma célula, use a flag `cena: true` do `Item` — ela renderiza no tamanho real e reduz por transform.
- Para formato vertical, **crie uma `<Composition>` 1080×1920 de verdade**. Não existe atalho.
- `cena: true` também é obrigatório para qualquer componente posicionado por porcentagem do canvas (ex.: `SimulatedCursor`, cujo `x`/`y` são 0–100 do vídeo inteiro).

### 2. `Sequence` renderiza um `AbsoluteFill`

Ele sai do fluxo. Dentro de um flex com `space-between`, todos os blocos colapsam no canto superior esquerdo.

Em peça com várias `Sequence` empilhadas, use **posições absolutas explícitas** (`top`, `left`), como em `src/playground/pecas.tsx`.

### 3. `defaultProps` passa por JSON

Elementos React não sobrevivem: chegam do outro lado como `{key, ref, props}` e o React lança o erro #31.

Nunca passe JSX por `defaultProps`. Crie um componente estável por página no módulo:

```tsx
const COMPONENTES = Object.fromEntries(
  PAGINAS.map((p) => [p.id, () => <Pagina {...p} />]),
);
```

### 4. Em `effects`, a ordem importa — e o tipo do efeito também

- **Gerador** (`checkerboard`, `rings`, `lightLeak`, `zigzag`) desenha imagem nova e ignora o que havia embaixo.
- **Filtro** (`thermalVision`, `halftone`, `pixelate`, `duotone`, `zoomBlur`) lê os pixels existentes e os transforma.

Um filtro sem nada embaixo não produz nada. Um gerador no fim do array apaga tudo que veio antes.

Muitos efeitos com parâmetros default entregam pouco (`duotone({})` sai branco chapado). Sempre configure.

### 5. `TransitionSeries` dura menos que a soma das partes

A duração é **soma das sequências − soma das sobreposições**. Duas de 40 frames com transição de 20 dão 60, não 80.

Se o `<Loop>` for maior que isso, sobra tela preta. Use o helper `cicloTransicao()` em `pages-extra.tsx`.

Prefira blocos curtos e transição longa: senão o espectador quase sempre pega a cena parada em vez do movimento.

### 6. Cenas de mídia nem sempre aceitam vídeo

`MediaFrame` e `SplitScreen` chamam `isVideoSource(src)` e trocam `<Img>` por `<Video>` sozinhas — basta passar um `.mp4`.

`ZoomPanFrame`, `DeviceMockupZoom` e `CalloutSpotlight` renderizam **só `<Img>`**: passar vídeo resulta em quadro vazio, **sem erro**. Extraia um frame:

```bash
ffmpeg -ss 3.2 -i public/video.mp4 -frames:v 1 -q:v 2 public/video.jpg
```

Mídia quadrada em cena 16:9 deixa tarja preta — use `fit="cover"`.

---

## Regras do ambiente

- **`typescript` fica fixo em `5.x`.** O TS 7 removeu `ts.sys` da API JS e o bundler do Remotion depende dela. Sintoma: `Cannot read properties of undefined (reading 'readFile')`.
- **O alias `@/` precisa estar em dois lugares** — `tsconfig.json` (`paths`) e `remotion.config.ts` (`overrideWebpackConfig`). Só o tsconfig faz o typecheck passar e o render quebrar.
- **Mexeu em `remotion.config.ts` ou `tsconfig.json`? Reinicie o Studio.** Ele lê esses arquivos apenas no boot; hot reload não pega, e o sintoma é tela em branco.
- **Mapas não renderizam headless.** `maplibre-gl` exige WebGL2; o `delayRender` nunca resolve. A página existe em `pages-mapas-ia.tsx` como `PAGINA_MAPAS`, desregistrada de propósito.

## Ao usar o CLI do RemotionUI

- **`remotion-ui add` de um *composition* edita o seu `Root.tsx`**, injetando `<Composition>` e imports fora das suas pastas, sem avisar. Sempre confira o Root depois.
- **`remotion-ui init` cria um projeto novo aninhado** se não achar config. Escreva o `remotion-ui.json` à mão.
- **Atualizar a lib sobrescreve arquivos já copiados.** A 0.7.0 reescreveu `transition-timing.ts` (derrubando quatro wipes) e trocou a API do `TerminalSimulator`. Reinstale os componentes afetados.

---

## Como escrever uma peça aqui

1. **Consulte `catalog.json`** para achar o componente pela intenção (campo `quando`) e o caminho (`importa`).
2. **Cores e fonte vêm do tema.** Importe de `src/shared/theme.ts` (`PALETTE`, `MONO`, `RADIUS`). Nunca escreva hex no meio do componente — quebra a troca de marca em `brand.ts`.
3. **Passe `fontSize` explícito** sempre que o componente aceitar. Sem isso ele calcula a partir de `useVideoConfig()` e não escala.
4. **Uma coisa se move por vez.** Duas animações simultâneas viram ruído; veja as anotações em `src/recipes/recipes.tsx`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [victorsodre/remotion-agent-catalog](https://github.com/victorsodre/remotion-agent-catalog) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
