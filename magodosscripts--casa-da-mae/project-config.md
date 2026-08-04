---
trigger: always_on
description: > Este arquivo é a fonte de verdade para quem pega o projeto agora. A pasta
---

# Casa da Mãe — guia completo do projeto

> Este arquivo é a fonte de verdade para quem pega o projeto agora. A pasta
> `docs/` (A–O) tem o design original detalhado, mas está **parcialmente
> desatualizada** — quando este arquivo e a `docs/` divergirem, vale este.

## O que é

PWA mobile-first de rotina doméstica para uma família de 8 pessoas, com
etiquetas NFC coladas pela casa e um sistema de **mesada gamificada** para as
crianças. Nome do sistema: **"Casa da Mãe"** (não usar logo; sempre o nome
escrito). Idioma de tudo — UI, código, commits: **português**.

- **Produção**: https://casaos-eight.vercel.app (projeto Vercel `casaos`)
- **Repositório**: https://github.com/Magodosscripts/casa-da-mae (privado)
- **Deploy**: push na `main` = deploy automático em produção (Vercel Git).
  Também dá manual: `npx vercel --prod --yes`.

## Stack

- **Next.js 16** (App Router, Server Components, Server Actions) + TS strict + Tailwind v4
- **Supabase**: Postgres (região `aws-sa-east-1`), Auth (login por **senha**),
  Storage (bucket privado `mesada-fotos`)
- **`pg` Pool direto** no Postgres (pooler porta 6543) para as queries do app —
  o client REST do Supabase só é usado para Auth e Storage
- **Vercel** com `regions: ["gru1"]` (São Paulo — NÃO remover, é o que segura a
  latência; sem isso as funções vão para os EUA e cada tela custa ~2s)
- **OpenAI** (`gpt-4o-mini` com visão) para a prévia de IA das fotos da mesada
- Camadas: `src/domain` (regras puras) → `src/application` (use-cases) →
  `src/infrastructure` (banco, auth, storage) → `src/app` (telas/rotas)

## Os 8 moradores

| Nome | Papel | Observações |
|---|---|---|
| Michella (40) | `house_admin` — "a mãe" | Dona da casa; aprova fotos e ajusta dinheiro |
| Gabriel | `tech_admin` | Autor original (quem te passou o projeto) |
| Henrique | `resident` | |
| Naty (26) | `resident` | Nora |
| **Rafael (17)** | `resident` + mesada | **Sem NFC no celular** → permissão `allowance.app` deixa ele tirar a foto pelo app |
| **Luiz (12)** | criança + mesada | |
| **Carlos (10)** | criança + mesada | |
| **Luiza (8)** | criança + mesada | Tem quarto próprio (separado dos meninos) |

Logins: `nome@casa.com.br` + senha simples (palavra+2 dígitos, definidas por
`scripts/set-passwords.mjs` — as senhas atuais só a família tem; o script gera
novas e imprime UMA vez). Crianças veem uma barra de navegação reduzida
(Mesada + Perfil). `/perfil` só troca senha — o resto foi removido de propósito.

## A mesada (o coração atual do sistema)

Regras pedidas pela família — **não relaxar sem pedido explícito**:

1. **Toda tarefa exige foto NOVA da câmera** (getUserMedia ao vivo em
   `src/app/mesada/camera.tsx`; nunca `<input type=file>`, senão dá para pegar
   da galeria).
2. **Toda tarefa exige a etiqueta NFC** (`requires_tag`): no app aparece só
   "Vá até a etiqueta e encoste o celular", sem botão. Encostar no NFC
   (`/t/[code]`) abre a tela da foto. **Exceção**: Rafael (`allowance.app`)
   tem o botão no app.
3. **Tarefa é única no dia**: quem faz primeiro leva; os outros veem "Fulano
   já fez". Índice único `(household_id, task_key, ref_date, slot)`.
4. **Louça tem 2 turnos reais** (`slots_per_day=2`): manhã/tarde (até 17:59) e
   noite. E é **opcional** (`mandatory=false`) — fazer ganha R$10, não fazer
   não perde nada.
5. As demais são **obrigatórias e diárias**: se NINGUÉM fizer, todos os
   elegíveis perdem o valor (penalidade = mesmo valor da tarefa).
6. **Dois quartos**: `room` = meninos (Luiz, Carlos, Rafael); `room_luiza` =
   só a Luiza. Tabela `allowance_task_participants` — tarefa sem linhas vale
   para todos; com linhas, só para os listados (telas, ações e penalidades
   respeitam isso).
7. **Dinheiro só entra quando a mãe aprova a foto** (`/mesada`, seção
   Aprovar). Depois de aprovar/recusar, **a foto é apagada do storage**
   (pedido: não ocupar espaço). Recusa devolve a vaga da tarefa.
8. **IA**: ao subir a foto, `after()` chama `analise-foto.ts` (OpenAI) e grava
   `ai_ok`/`ai_resumo` no lançamento — a mãe vê "✨ IA: ..." junto da foto. A
   decisão é sempre dela. Sem `OPENAI_API_KEY`, o app funciona sem a prévia.
9. **Meia-noite (cron)**: `/api/cron/fechar-dia` (Vercel Cron 03:00 UTC =
   00:00 SP, autenticado por `CRON_SECRET`): aprova sozinho tudo que a mãe não
   revisou no dia e aplica as penalidades. Não existe botão manual.
10. **Anti-fraude**: segunda foto da mesma tarefa pendente no dia é recusada
    ("já mandou"); corrida pelo mesmo turno é resolvida pelo índice único.
11. Dinheiro **sempre em centavos (int)**, ledger append-only
    (`allowance_entries`), ranking soma só `status='confirmed'`.

Arquivos-chave: `src/domain/mesada/money.ts` (regras puras: turnos, situação,
penalidade) · `src/infrastructure/mesada/mesada-repository.ts` (todas as
leituras/escritas) · `src/app/mesada/` (tela, actions, câmera) ·
`src/app/t/[code]/` (tela da etiqueta; `tag-mesada-foto.tsx` é a da foto).

## As 6 etiquetas NFC físicas (códigos em produção — NÃO regenerar)

| Etiqueta | Mesada | URL |
|---|---|---|
| Lavar louça | R$10 (opcional, 2 turnos) | `/t/7RBDapMef9UosnSCVPrl` |
| Passear com o João (cachorro) | R$5 | `/t/ErdRNP3hbMDikO2fGn9b` |
| Arrumar quarto dos meninos | R$5 | `/t/8XVM7SaeApBFrdLykHn8` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Magodosscripts/casa-da-mae](https://github.com/Magodosscripts/casa-da-mae) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-01 -->
