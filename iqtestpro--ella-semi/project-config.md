---
trigger: always_on
description: > Este arquivo é lei. Toda sessão do Claude Code que abrir neste repo carrega estas regras automaticamente. Não são sugestões.
---

# CLAUDE.md — Regras Permanentes Deste Projeto (ELLA Semijoias)

> Este arquivo é lei. Toda sessão do Claude Code que abrir neste repo carrega estas regras automaticamente. Não são sugestões.

---

## Princípio raiz

**Toda solicitação do usuário passa por uma skill antes de virar código.** Skills são obrigatórias, não opcionais.

Você nunca escreve código de aplicação direto. Você nunca "vai logo fazendo". Você nunca improvisa um fluxo só porque o pedido parece simples. O caminho é sempre: *entender → alinhar via skill → planejar via skill → executar via skill → revisar via skill*.

A regra existe porque o trabalho deste projeto é **lapidação contínua**, não velocidade bruta.

Adicionalmente, este é um **site-catálogo de joias e semijoias premium**. Sem checkout no site (toda venda termina no WhatsApp). Isso significa que **a apresentação visual é o ponto único de conversão** — qualidade prevalece sobre economia em todo trade-off.

---

## A marca

- **Marca**: ELLA Semijoias (caixa-alta no logo: "ELLA")
- **Fundadora**: Ellen Lopes (pessoa real — ELLA é marca, Ellen é pessoa, não confundir)
- **WhatsApp atendimento**: link curto `wa.link/adq88g` para atendimento geral; número direto E.164 para mensagem dinâmica do carrinho (a confirmar no grilling)
- **Identidade visual atemporal**: rosa salmão da logo + ELLA serif preta + sparkles dourados + perna alongada do "A". Não muda nunca.

---

## Skills disponíveis

### Engenharia — Matt Pocock (espinha do projeto)
- **`/grill-with-docs`** — Grilling antes de qualquer mudança. Alinha escopo, sharpening de jargão, atualiza `CONTEXT.md`, cria ADR.
- **`/grill-me`** — Grilling sem código. Discussões de produto, arquitetura, processo.
- **`/to-prd`** — Conversa atual → PRD → issue no tracker.
- **`/to-issues`** — Plano/PRD → issues vertical-slice independentes.
- **`/triage`** — Triagem de issues pelo state-machine de roles.
- **`/tdd`** — Red-green-refactor. **Único caminho permitido para escrever código de feature ou fix.**
- **`/diagnose`** — Loop disciplinado: reproduzir → minimizar → hipotetizar → instrumentar → consertar → teste de regressão.
- **`/zoom-out`** — Antes de mexer em código que você não escreveu nesta sessão.
- **`/improve-codebase-architecture`** — Achar oportunidades de deepening. Rodar periodicamente.
- **`/caveman`** — Modo ultra-comprimido. Use durante execução longa.
- **`/write-a-skill`** — Criar skills custom quando padrão se repete 3+ vezes.

### Frontend — Emil Kowalski + Leon Lin (Taste-Skill)
- **`emil-design-eng`** — Audita UI sob princípios Emil Kowalski: animações ≤300ms, easing custom, perceived performance. **Ativo em todo trabalho de UI.**
- **`taste-skill` com variante `minimalist-ui`** (ADR-0002) — Anti-slop frontend, calibrado para esta marca: clean editorial, warm monochrome, pastéis. Parâmetros: `DESIGN_VARIANCE=7`, `MOTION_INTENSITY=8`, `VISUAL_DENSITY=3`. Vocabulário do projeto: "warm editorial soft glam". **Ativo em todo trabalho de UI.**
- **`redesign-skill`** — Quando a tarefa é melhorar UI já existente.
- **`output-skill`** — Quando o agent estiver truncando código longo.

> **Mapeamento de aliases ↔ skills instaladas**: `taste-skill` = `design-taste-frontend`; `redesign-skill` = `redesign-existing-projects`; `output-skill` = `full-output-enforcement`. Quando invocar, use o nome real da skill instalada; quando documentar, use o alias deste arquivo.

### Geração de mídia — Higgsfield CLI (`higgsfield`)

**Política**: qualidade prevalece sobre economia (ADR-0001). Sem teto de créditos. Modelo escolhido por adequação à tarefa, não por preço.

**Pipeline único** (ADR-0006): **toda mídia visual do site é gerada via Higgsfield CLI**. Sem exceção produtiva. Stock photos, bancos de imagens (Unsplash/Pexels/etc.), Midjourney, DALL-E, Stable Diffusion local — **proibidos** como fonte de asset publicado. Foto real fornecida pela Ellen pode entrar como **input de background swap Higgsfield** para padronização visual, nunca como asset cru final.

**Modelos por contexto**:
- **Foto isolada de produto**: **Nano Banana Pro** (reasoning forte em detalhe fino de joia, textura de metal, engaste, sparkle).
- **Lifestyle / hero / retrato com modelo**: **Soul** com Soul Character "Modelo Ella" (consistência de identidade entre frames).
- **Fundo / ambiente neutro / fotorrealismo de cenário**: **Flux**.
- **Vídeos aspiracionais (hero da home, hero da Campanha Atual, hero da `/campanha`, qualquer cena ≥6s ou de marca)**: **Cinema Studio** — real optical physics.
- **Microvídeos decorativos (loops <4s, transições)**: **Seedance 2.0 1080p**.
- **Cenas com diálogo / lipsync / storyboard multi-shot**: **Kling 3.0**.
- **Background swap de foto real**: **Nano Banana Pro** com input da foto original.

**Regras de uso**:
- **Nunca** placeholder em produção (`https://placehold.co/...`, "lorem picsum", `<img src=""/>`). Higgsfield ou foto real da Ellen via bg-swap.
- Toda mídia em `assets/generated/<categoria>/<id>.{webp|mp4}` + metadata em `assets/generated/manifest.json` (prompt, modelo, seed, data, camada). Reprodutibilidade obrigatória.
- Prompts versionados em `assets/prompts/*.md`. Trate prompt como código.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [IQtestPRO/Ella-Semi](https://github.com/IQtestPRO/Ella-Semi) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
