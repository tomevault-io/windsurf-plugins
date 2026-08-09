---
trigger: always_on
description: Este é o projeto-base usado para gerar páginas premium de apresentação
---

# Projeto: Páginas Premium de Portfólio (Base)

Este é o projeto-base usado para gerar páginas premium de apresentação
pessoal para profissionais autônomos (nutricionistas, personal trainers,
modelos comerciais, tatuadores, influencers, etc). Este arquivo define as
convenções que devem ser seguidas em TODA página gerada a partir deste
template.

Este repositório específico também é o laboratório onde o template foi
construído e testado (cliente fictícia "Isabella Marques", 5 rotas de
demonstração — ver "Direções de partida" abaixo). Pra um cliente novo,
ele é **duplicado** — ver "Fluxo de trabalho por cliente".

## O que este projeto NÃO é

Isto NÃO é uma landing page de vendas. Não existe produto, checkout,
oferta ou "comprar agora". A pessoa é o produto — a página existe para
apresentá-la da forma mais impactante possível e gerar em quem visita o
desejo de contratá-la, agendar com ela ou fechar uma parceria.

Pense nela como um "link na bio" premium: substitui o Linktree/WhatsApp
genérico por uma experiência editorial que reforça autoridade e desejo
antes mesmo da pessoa conversar com o cliente/parceiro.

## Stack técnica

- React + Vite + TypeScript
- Tailwind CSS para estilização
- GSAP + ScrollTrigger para animação (ver "O motor comprovado" e
  "Experiência de scroll" — como exatamente cada mecanismo é usado)
- Framer Motion apenas para transições simples de UI que NÃO dependem
  da posição do scroll (hover de botão, abrir/fechar menu, troca de
  rota)
- Antes de escrever um componente do zero, **verificar se já existe um
  equivalente em `src/components/`** (ver "O motor comprovado" abaixo) —
  esse é o ativo mais valioso deste projeto, já testado em 5 paletas
  diferentes.
- Deploy via Vercel

## Estrutura de pastas

```
src/
  components/     # componentes reutilizáveis e já comprovados (Hero, Sobre,
                   # Portfólio, Campanhas, Processo, Contato, etc.) — ver
                   # "O motor comprovado" abaixo
  pages/          # uma página por rota; durante a fase de propostas tem
                   # várias (ver "Múltiplas propostas por cliente"), mas o
                   # projeto final de um cliente é normalmente uma só
  content/        # dados do cliente atual (bio, stats, portfólio, links)
                   # num único arquivo tipado — renomear por cliente
  assets/         # imagens, vídeos, ícones já processados — organizados em
                   # portrait/ e landscape/ dentro de images/ e videos/ (ver
                   # "Regras de design")
  lib/            # helpers (setup do gsap, conversão de cor, etc.)
Context/          # (não versionado) prints, bio, fotos/vídeos originais e
                   # brief do cliente atual — ver .gitignore
```

## O motor comprovado: reaproveitar, não recriar

Este projeto passou por bastante iteração até chegar num resultado que
funciona bem (várias abordagens de transição entre seções foram tentadas
e descartadas antes desta, ver o "por quê" logo abaixo). O resultado está
validado em produção nas rotas `/vinho`,
`/noir`, `/riviera`, `/studio` e `/cover`, cada uma com paleta e
tipografia próprias sobre a mesma estrutura. **Não reinvente essas peças
pra um cliente novo** — componha a partir delas, e só escreva um
componente novo quando a seção realmente não tiver equivalente.

### Componentes prontos (`src/components/`)

- `VideoHero.tsx` — abertura em vídeo full-bleed, textos entrando com
  GSAP, `replayOnScroll` pra reanimar a entrada toda vez que a seção volta
  a ficar visível.
- `AboutSection.tsx` — foto de fundo com parallax sutil, bio + stats
  embaixo, "eyebrow" fixo (`position: sticky`) no topo da seção.
- `PinnedPortfolio.tsx` — a galeria principal: cada item é uma seção
  cheia, troca por fade de opacidade com duração fixa (não escrubado) ao
  ficar ativa. É a seção mais importante da página (ver "Estrutura de
  conteúdo").
- `CampaignsSection.tsx` — chips de categoria/especialidade com ícone
  opcional (`CategoryIcon.tsx`) + carrossel de marcas parceiras opcional
  (`BrandsMarquee.tsx`).
- `ProcessSection.tsx` — linha do tempo "como funciona" (numerada, com
  foto de fundo escurecida). Genérica o suficiente pra qualquer nicho com
  um processo de trabalho (briefing → produção → entrega; avaliação →
  plano → acompanhamento; orçamento → sessão → entrega de fotos) — só
  trocar os textos dos passos.
- `ContactSection.tsx` — fechamento com foto de fundo, CTA principal,
  bloco opcional de material de apoio (mídia kit, tabela de valores,
  portfólio em PDF), barra fixa com Instagram/e-mail/localização.
- Suporte: `FullBleedMedia.tsx` (crop 9:16/16:9 sem esticar/cortar mal;
  aceita `desktopSrc` pra trocar a mídia por uma versão landscape só no
  desktop — ver "Regras de design" — via JS `matchMedia`, nunca `<source
  media>` dentro de `<video>`, que não tem suporte confiável entre
  browsers), `MagneticGlowButton.tsx` e `CTAButton.tsx` (CTAs),
  `ScrollFade.tsx` (reveal simples pra conteúdo fora do container de
  scroll-snap), `CategoryIcon.tsx`, `BrandsMarquee.tsx`.

Todo componente recebe cor/fonte via props — nunca tem cor hardcoded
internamente. Adaptar o visual pra um cliente novo é passar props
diferentes (ver "Direções de partida"), não editar o componente.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [doupsdigital/projeto-personal](https://github.com/doupsdigital/projeto-personal) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
