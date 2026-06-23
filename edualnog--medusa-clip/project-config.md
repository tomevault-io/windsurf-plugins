---
trigger: always_on
description: Contexto para o Claude Code. Leia antes de codar.
---

# CLAUDE.md — Medusa Clip

Contexto para o Claude Code. Leia antes de codar.

> ⚠️ DIRECAO ATUAL (2026-06-21): app **desktop local-first, SEM CADASTRO** ("Medusa
> Clip", medusaclip.com). O processamento roda **no PC do usuario** (app Electron com
> motor + ffmpeg embutidos) — NAO em VPS, NAO em worker na nuvem. A VPS/worker foi
> abandonada. **Login/conta removidos** (era Supabase auth) — o app nao tem mais
> cadastro, nem backend. O motor de cortes Python foi reaproveitado integral, agora
> empacotado como binario.

## O que e

**App desktop para criadores de games**: o usuario instala o Medusa Clip, **abre (sem
cadastro, sem login)**, conecta a **propria** chave de IA (OpenRouter, OpenAI ou
Anthropic), escolhe um video local (ou cola um link publico) e recebe **cortes
verticais 9:16** nivel Opus Clip (ganchos, legenda karaoke, enquadramento) — tudo
processado **na propria maquina**. App **gratuito** (sem assinatura), **sem creditos** —
o custo de IA (LLM) e do usuario, pela chave dele. Estrategia: base enorme de usuarios
primeiro (atrito zero, **no sign-up**), **monetizar de outra forma no futuro** (modelo
ainda nao definido). Privacidade por padrao: **o gameplay nunca sobe pra nuvem** e
**nao ha conta nossa**.

## Arquitetura: app Electron (local), sem backend

Processamento de video (yt-dlp, ffmpeg, whisper, render) e **pesado** e roda **100%
no computador do usuario**, dentro do app Electron. **Nao ha servidor nosso**: sem
conta, sem auth, sem Supabase. Tudo (config, aceite legal, clipes) fica no disco do
usuario.

```
DESKTOP (Electron @ PC do usuario)
  renderer/  UI 8-bit (4 views: Inicio / Biblioteca / Chaves API / Ajustes)
  main.js    spawn do motor (binario)
  engine/    medusacut-engine + ffmpeg + ffprobe (embutidos)
  config.json (userData): provider + chave de IA local (por provedor) +
              onboarding (aceite legal gravado SO local: versao/data/itens)
  ~/Downloads/Medusa Clip/: biblioteca de clipes

WEB (Next.js @ Vercel)
  landing 8-bit + downloads  (so isso — SEM login, SEM api, SEM Supabase)
```

- **Desktop**: faz todo o trabalho pesado; o motor e um **binario** (`medusacut-engine`)
  chamado por `main.js` via `spawn`, que repassa progresso em JSON linha a linha.
  `ffmpeg`/`ffprobe` vivem em `desktop/engine/` e entram no PATH do subprocesso —
  self-contained, sem dep do sistema nem Python instalado. **Sem auth**: ao abrir, o
  app vai direto pro onboarding (1o acesso) ou pro app.
- **Web (Vercel)**: so a landing + downloads. Sem login, sem worker, sem fila, **sem
  rotas `api/*`**. Site 100% estatico.
- **Custo de compute e do USUARIO** (CPU/banda/disco dele). O dono so paga o site.

## Regras inegociaveis (seguranca + modelo)

- **BYO key**: cada usuario usa a PROPRIA chave de IA — **OpenRouter**, **OpenAI** ou
  **Anthropic** (escolhe o provedor no app). **NUNCA** publicar a chave do dono nem rodar
  IA "as custas da casa". A chave fica salva **no dispositivo do usuario** (`config.json`
  em userData, cifrada, uma por provedor) e vai direto pro provedor — nunca passa por
  servidor nosso. No motor: `LLM_PROVIDER` + `LLM_API_KEY`; OpenRouter/OpenAI via SDK
  da OpenAI (base_url), Anthropic via SDK nativo `anthropic` (API != OpenAI).
- **Local-first**: video bruto e clipes **nunca** saem do PC do usuario. Nada de
  upload de gameplay pra nuvem.
- **Sem cadastro (no sign-up)**: o app **nao tem login nem conta**. Nao reintroduzir
  auth/Supabase/backend sem decisao explicita. O aceite legal (Termos, Privacidade,
  18+, responsabilidade de conteudo) e gravado **so localmente** (`config.json`,
  `onboarding`: versao/data/itens) — nenhuma prova de aceite sobe pra servidor.
- **App gratuito**: sem assinatura, sem creditos. Custo de IA e do usuario (BYO key).
  Monetizacao futura ainda nao definida — nao assumir/implementar cobranca.
- Se algum dia voltar a haver segredo de servidor (p/ billing): **so no servidor**
  (env Vercel), nunca `NEXT_PUBLIC`, nunca no git.

## Onde mora a qualidade ("nivel Opus Clip")

No motor (`agent/src/medusacut/`). **Prioridade: rapido E bom** — em 2026-06-21 o
pipeline foi muito simplificado/acelerado (de ~25min p/ ~3-5min num video de 12min).

1. **Transcricao** (`transcribe/whisper.py`): default `base` + greedy (`beam_size=1`,
   `condition_on_previous_text=False`). **DOIS backends, auto:** **MLX** (GPU/Neural
   Engine no Mac Apple Silicon, ~3x — via `mlx-whisper`) e **faster-whisper** (CPU, ou
   **GPU NVIDIA/CUDA** no Win/Linux quando ha libs). Fallback seguro: se MLX/CUDA
   falhar, cai pra CPU sem quebrar. Override: `MEDUSA_WHISPER_BACKEND`, `MEDUSA_WHISPER_DEVICE`.
2. **Analise viral multimodal + multi-modelo** (`hooks/`, `frames.py`, `llm.py`):
   triagem barata (texto) -> juiz forte que VE keyframes -> re-rank. Chamadas de IA
   rodam em **paralelo** (`MEDUSA_LLM_WORKERS`, default 4).
3. **Legenda karaoke + hook** (`caption/karaoke.py`): queimadas, estilo gamer (ffmpeg
   local sem libass/drawtext -> Pillow desenha PNGs). Compostas como **faixa alpha
   (qtrle) + overlay unico, FUNDIDO no mesmo encode do render** (1 passada — NAO 2).
   - Legenda: palavra a palavra (karaoke). Hook: manchete nos **primeiros ~5s**, na
     divisa abaixo da facecam (`build_hook_track`).

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Edualnog/medusa-clip](https://github.com/Edualnog/medusa-clip) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-23 -->
