---
trigger: always_on
description: > Este arquivo é a fonte de verdade do projeto. O Claude Code lê este arquivo automaticamente em toda sessão. **Toda decisão técnica importante mora aqui.** Quando algo do projeto evoluir, atualize este arquivo no mesmo PR.
---

# CLAUDE.md — Corporis Finance

> Este arquivo é a fonte de verdade do projeto. O Claude Code lê este arquivo automaticamente em toda sessão. **Toda decisão técnica importante mora aqui.** Quando algo do projeto evoluir, atualize este arquivo no mesmo PR.

---

## 1. O que é este projeto

**Corporis Finance** é uma plataforma web de gestão financeira empresarial focada em **DFC (Demonstrativo de Fluxo de Caixa)**, construída inicialmente para a clínica **Corporis Fisioterapia & Pilates** (Xanxerê/SC).

**MVP single-tenant** (uma única clínica, um único usuário gestor). A arquitetura, no entanto, é desenhada para evoluir para multi-tenant SaaS no futuro — toda tabela tem `organization_id` desde o dia 1.

### Princípios de produto

1. **Caixa é rei.** Todo o sistema opera em regime de caixa. Competência fica para uma futura versão com DRE.
2. **O DFC é o coração.** Toda feature precisa responder: "isso melhora a clareza ou velocidade de produzir o DFC?"
3. **Dados realistas, não Lorem ipsum.** Em dev, seed com dados verossímeis de uma clínica de fisio+pilates.
4. **Tom acolhedor.** "Aluna", "atendimento", "movimentação". Nunca "paciente" ou "transação financeira".
5. **Não reinventar a contabilidade.** O plano de contas Corporis é a referência. Não criar grupos novos sem necessidade.

### Princípios técnicos

1. **Tipos primeiro.** TypeScript estrito. Schema do banco define tipos via Supabase codegen.
2. **Server Components por padrão.** Client Components apenas quando precisa de estado, eventos ou hooks.
3. **RLS em todas as tabelas, sempre.** Mesmo em single-tenant. Nunca commitar tabela sem policy.
4. **Erros falam português.** Mensagens de UI são em pt-BR. Logs e código em inglês.
5. **Sem feature flag gambiarra.** Se vai construir, constrói direito. Se ainda não vai, não constrói.

---

## 1.1 Como usar a pasta `/design`

> **Regra absoluta:** toda tela implementada deve ser fiel ao mockup aprovado em `/design/mockups/`. O Claude Code não toma decisões visuais autônomas — qualquer dúvida de layout ou componente, a resposta está no HTML de referência.

### Estrutura da pasta

```
/design
  /mockups
    01-dashboard.html          ← mockup interativo completo (fonte de verdade visual)
    01-dashboard.png           ← screenshot para referência rápida
    02-lancamentos-listagem.html
    02-lancamentos-listagem.png
    03-lancamentos-modal.html
    03-lancamentos-modal.png
    04-importacao-upload.html
    04-importacao-upload.png
    05-importacao-conciliacao.html
    05-importacao-conciliacao.png
    06-contas-listagem.html
    06-contas-listagem.png
    07-contas-cadastro.html
    07-contas-cadastro.png
    08-dfc.html
    08-dfc.png
    09-orcado-realizado.html
    09-orcado-realizado.png
    10-orcamento-editor.html
    10-orcamento-editor.png
    11-projecao.html
    11-projecao.png
    12-consultor-ia.html
    12-consultor-ia.png
    13-plano-de-contas.html
    13-plano-de-contas.png
    14-importacoes-historico.html
    14-importacoes-historico.png
    15-configuracoes.html
    15-configuracoes.png
    16-login.html
    16-login.png
    17-onboarding.html
    17-onboarding.png
  DLS.md                       ← Design Language System completo (tokens, componentes, regras)
  SCREENS.md                   ← índice de todas as telas com status de aprovação
```

### Como o Claude Code deve ler os mockups

Ao começar a implementar qualquer tela, **leia o HTML de referência antes de escrever uma linha de código**. Extraia:

1. **Estrutura de layout** — grid, flex, proporções de colunas, larguras fixas
2. **Hierarquia de componentes** — quais componentes existem e como se encaixam
3. **Tokens aplicados** — cores, espaçamentos, tipografia, raios de borda usados
4. **Dados exibidos** — quais campos existem, formatos de data/moeda, labels
5. **Estados visíveis** — o mockup mostra um estado específico; inferir os outros (vazio, erro, loading) a partir do DLS.md
6. **Interações indicadas** — comentários no HTML descrevem hover, modais, transições

### Hierarquia de decisão visual

```
1. Mockup HTML aprovado em /design/mockups/   → fonte de verdade
2. DLS.md em /design/                         → tokens e regras
3. Componentes shadcn/ui customizados          → base de implementação
4. Intuição do Claude Code                     → NUNCA — sempre consultar 1 ou 2
```

### Quando o mockup e o DLS conflitarem

O mockup prevalece — ele foi produzido a partir do DLS e representa a decisão final. Se o conflito for genuíno (ex: mockup usa uma cor fora do DLS), **parar e perguntar** antes de resolver por conta própria.

### SCREENS.md — índice de aprovação

Antes de implementar qualquer tela, verifique o status em `design/SCREENS.md`. Só implemente telas com status `✅ aprovado`. Se estiver como `🔄 revisão`, aguarde aprovação antes de começar.

---

## 2. Stack

| Camada | Tecnologia | Versão | Por quê |
|---|---|---|---|
| Framework | **Next.js** (App Router) | 15.x | RSC, Server Actions, ótima DX |
| Linguagem | **TypeScript** | 5.x estrito | Segurança de tipos no domínio financeiro é não-negociável |
| UI | **React** | 19.x | — |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [corporisfisioterapiapilates-ops/corporis-finance](https://github.com/corporisfisioterapiapilates-ops/corporis-finance) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-14 -->
