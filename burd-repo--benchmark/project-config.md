---
trigger: always_on
description: Esta landing page usa uma identidade visual escura, tecnica e premium. O site trabalha com:
---

# Design System Burd

## 1. Visao geral do design

Esta landing page usa uma identidade visual escura, tecnica e premium. O site trabalha com:

- fundo preto/carvao quase constante;
- linhas finas cinza-escuro estruturando a pagina como um grid editorial;
- blocos com imagens SVG grandes e composicoes densas;
- textos principais em sans serif estilo SF Pro;
- textos de apoio, labels e UI tecnica em mono (`JetBrains Mono`);
- contraste alto, mas sem branco puro dominante o tempo inteiro;
- azul e verde usados como acentos funcionais, nao como cor estrutural primaria.

A sensacao geral e de produto de infraestrutura/IA: frio, controlado, preciso, modular e orientado a sistema.

## 2. Principios visuais obrigatorios

- Manter dark mode como base de toda a interface.
- Usar bordas finas em `#262626` ou `#2A2A2A` para estrutura e divisorias.
- Manter blocos retangulares, grids e seccoes com composicao bem alinhada.
- Evitar cores fora da paleta extraida do projeto.
- Usar arredondamento minimo e seletivo. O sistema nao e baseado em cards fofos ou suaves.
- Usar mono em labels, microcopy, navegação tecnica, preços, FAQ auxiliar e indicadores.
- Evitar sombras fortes fora dos casos ja existentes de mock/cards sobre imagem.
- Evitar gradientes novos sem referencia direta nas artes atuais.
- Preservar a sensacao de layout arquitetado por linhas horizontais e verticais.
- Antes de criar uma nova interface, comparar o bloco novo com a landing atual em desktop e mobile.

## 3. Paleta de cores real do projeto

Valores extraidos diretamente de `app/page.tsx`, `app/globals.css` e classes utilitarias atuais.

| Token | Valor | Uso atual | Quando usar |
|---|---|---|---|
| `background.page` | `#0A0A0A` | fundo principal de `body`, `main`, cards escuros | fundo padrao de paginas e seccoes principais |
| `background.section.alt` | `#080808` | separadores vazios, bloco de texto central | secoes de pausa/transicao |
| `background.panel` | `#111111` | cards e blocos de preview | areas internas sobre o fundo principal |
| `background.panel.alt` | `#141414` | card central da secao azul | variacao sutil de card tecnico |
| `background.panel.deep` | `#090909` | `MockWindow` | UI embutida / painéis internos |
| `background.badge` | `#1A1A1A` | `SectionEyebrow`, estado ativo do nav glass | labels, pequenas barras, superficies compactas |
| `background.muted` | `#202020` | hover de botao | hover discreto |
| `border.default` | `#262626` | bordas principais da landing | divisorias gerais |
| `border.panel` | `#2A2A2A` | bordas do `MockWindow` | UI embutida, painéis detalhados |
| `border.muted` | `#2F2F2F` | barra vertical do eyebrow | acento estrutural secundario |
| `border.soft` | `#3A3A3A` | circulos/controles secundarios | elementos menos estruturais |
| `text.primary` | `#F5F5F5` | titulos, labels principais, nav | texto principal |
| `text.secondary` | `#9CA3AF` | paragrafos, labels tecnicas | texto auxiliar |
| `text.tertiary` | `#626262` | assinatura em depoimento | texto de hierarquia baixa |
| `text.inverse` | `#111111` | seta sobre faixa clara | texto sobre superficie clara |
| `accent.green` | `#3F8047` | status, ganhos positivos | status positivo, feedback funcional |
| `accent.blue` | `#1f7ea6` | elemento radial no card de escalabilidade | acentos graficos pontuais |
| `accent.brand.blue` | `#0091E2` | label do ecossistema em componente antigo | somente quando a composicao pedir um highlight institucional |
| `surface.light` | `#D9D9D9` | faixa da URL no mock | superficies claras de contraste alto |

Observacao: ha uso de opacidades recorrentes, como `opacity-70`, `opacity-90`, `opacity-40` e `opacity-[0.13]`, especialmente em imagens e overlays.

## 4. Tipografia

### Fontes

- Sans principal: `SF Pro Display`, `SF Pro Text`, `-apple-system`, `BlinkMacSystemFont`, `Inter`, `sans-serif`
- Mono: `JetBrains Mono`, `monospace`

### Regras

- Titulos principais usam sans.
- Textos auxiliares e informacoes tecnicas usam mono.
- Nao introduzir uma terceira familia tipografica.

### Escala real encontrada

| Uso | Classes / tamanho | Observacao |
|---|---|---|
| Hero H1 | `text-[32px] sm:text-[36px] xl:text-[40px]` | principal chamada institucional |
| H2 de secao | `text-[32px] xl:text-[40px]` | secoes principais |
| H3 de card | `text-[28px] xl:text-[32px]` | cards do ecossistema |
| Titulo FAQ item | `text-[18px] xl:text-[20px]` | perguntas |
| Label eyebrow | `text-[16px] uppercase` | mono, compacto |
| Nav desktop | `text-[16px]` | mono |
| Paragrafo padrao | `text-[14px] xl:text-[16px] leading-[1.6]` | copy de secao |
| Paragrafo tecnico pequeno | `text-[12px] xl:text-[11.8px] leading-[1.65]` | feature cards |
| Microcopy tecnica | `text-[10px]`, `text-[11px]`, `text-[13px]` | mocks, faq auxiliar, labels |
| Quote destaque | `text-[24px] xl:text-[32px] leading-[1.35]` | bloco editorial |

### Pesos e tracking

- O projeto usa principalmente peso normal/regular e bold visual via tamanho/contraste.
- Tracking custom aparece pontualmente, por exemplo `tracking-[-0.02em]` em bloco central e `tracking-[0.05em]` no mock.
- Nao exagerar em letter-spacing; usar apenas quando a referencia ja pedir isso.

## 5. Layout e grid

### Estrutura geral

- A pagina principal usa `GridShell`.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Burd-repo/benchmark](https://github.com/Burd-repo/benchmark) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
