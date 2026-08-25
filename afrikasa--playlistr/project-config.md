---
trigger: always_on
description: This file provides guidance to Claude Code ([claude.ai/code](http://claude.ai/code)) when working with code in this repository.
---

# [CLAUDE.md](http://CLAUDE.md)

This file provides guidance to Claude Code ([claude.ai/code](http://claude.ai/code)) when working with code in this repository.

---

## DIRECTRIZES DE AGENTES — OBRIGATÓRIO LER ANTES DE QUALQUER ACÇÃO

### REGRA ABSOLUTA

**O Claude Code principal NÃO escreve código, NÃO corrige bugs, NÃO faz deploy.**
Toda a implementação, correcção, revisão e deploy é feita por subagentes especializados.
Violar esta regra — mesmo para "uma pequena alteração" — é proibido.

### Sequência obrigatória para qualquer pedido de feature ou alteração

```
PASSO 1 — software-architect
  Planeia a implementação. SEMPRE PRIMEIRO. Nunca saltar.

PASSO 2 — Implementação em paralelo (conforme camadas afectadas)
  Backend Python/FastAPI  → python-pro ou api-designer
  Frontend React          → react-specialist ou feature-developer
  Base de dados           → database-architect

PASSO 3 — security-auditor
  Valida segurança. SEMPRE. Mesmo em features sem auth óbvia.

PASSO 4 — code-reviewer  [LOOP até APROVADO]
  Revê código. Se BLOQUEADO → agentes corrigem → passo 3 → passo 4.
  Não avança sem APROVADO.

PASSO 5 — bug-hunter
  Testa regressões e bugs introduzidos.

PASSO 6 — devops-deployer
  Deploy/commit/push. NUNCA sem APROVADO do code-reviewer.
```

### Agentes por tipo de trabalho neste projecto

| Tipo de pedido | Agente(s) a invocar |
|----------------|---------------------|
| Nova feature ou alteração (backend Python/FastAPI) | `software-architect` → `python-pro` ou `api-designer` |
| Nova feature ou alteração (frontend React) | `software-architect` → `react-specialist` |
| Feature completa (backend + frontend) | `software-architect` → `python-pro` + `react-specialist` em paralelo |
| Bug ou erro reportado | `bug-hunter` |
| Erro em produção / logs | `error-detective` → `bug-hunter` |
| Revisão de segurança | `security-auditor` |
| Code review / "está bem?" | `code-reviewer` |
| Deploy / commit / push | `devops-deployer` (após `code-reviewer` APROVADO) |
| Script Python standalone | `python-pro` |
| Componente React / shadcn / Tailwind | `react-specialist` |
| Schema DB / migração | `database-architect` |

### Formato de anúncio obrigatório antes de cada sequência

Antes de lançar os agentes, anunciar em 2-3 linhas:
```
Plano: software-architect planeia →
       python-pro (backend) + react-specialist (frontend) em paralelo →
       security-auditor → code-reviewer → bug-hunter → devops-deployer.
A começar...
```

---

## Visão Geral

Descarregador de playlists Spotify como MP3 com UI React + API FastAPI.

**Fluxo:** React UI → POST /download → FastAPI → spotify_downloader.py → yt-dlp + ffmpeg → MP3

## Ferramentas do Sistema (Windows)

FerramentaCaminhoyt-dlp`C:\Users\marcu\yt-dlp-new.exe`ffmpeg`C:\Users\marcu\AppData\Local\Microsoft\WinGet\Packages\Gyan.FFmpeg_Microsoft.Winget.Source_8wekyb3d8bbwe\ffmpeg-8.1-full_build\bin\ffmpeg.exe`Python`C:\Users\marcu\AppData\Local\Programs\Python\Python39\python.exe`

## Estrutura

```
spotify-downloader/
├── spotify_downloader.py   ← core: Spotify API + yt-dlp + ffmpeg
├── backend/
│   ├── main.py             ← FastAPI: /download, /progress (SSE), /status, /cancel
│   └── requirements.txt    ← fastapi, uvicorn, python-dotenv
├── frontend/src/
│   └── App.js              ← React: chama API, subscreve SSE (copiar para o repo)
├── .env                    ← SPOTIFY_CLIENT_ID + SPOTIFY_CLIENT_SECRET
├── start.bat               ← inicia backend + frontend
└── requirements.txt        ← spotipy, yt-dlp, mutagen
```

## Comandos

```bash
# Instalar dependências Python (core + backend)
pip install -r requirements.txt
pip install -r backend/requirements.txt

# Iniciar tudo de uma vez (Windows)
start.bat

# Ou manualmente:
# Backend (porta 8000)
python -m uvicorn main:app --host 0.0.0.0 --port 8000 --app-dir backend/

# Frontend (porta 3000) — na pasta do repo clonado
cd ..\Spotify-Playlist-Downloader\frontend && npm start

# CLI standalone (sem UI)
python spotify_downloader.py "https://open.spotify.com/playlist/..." -o "D:/Música" -q 320
```

## Credenciais

```env
SPOTIFY_CLIENT_ID=...
SPOTIFY_CLIENT_SECRET=...
```

O `.env` é carregado automaticamente — o `_load_dotenv()` procura primeiro no CWD, depois na pasta do script.

## Arquitetura do Backend ([main.py](http://main.py))

EndpointMétodoDescrição`/download`POSTBusca playlist Spotify, devolve tracks com `id`/`cover`/`duration`, inicia thread de download`/progress`GETSSE: emite eventos `track_start`, `track_done`, `completed`, `cancelled/status`GETEstado atual (`idle`/`fetching`/`downloading`/`completed`)`/cancel`POSTSinaliza `threading.Event` para cancelar

**Thread model:** download corre em `threading.Thread(daemon=True)`. Eventos são enviados via `loop.call_soon_threadsafe` para `asyncio.Queue`, consumida pelo gerador SSE.

**Formato SSE:**

```json
{"type": "track_start", "index": 0, "current": 1, "total": 20}
{"type": "track_done",  "index": 0, "status": "done|failed|skipped", "current": 1, "total": 20}
{"type": "completed",   "done": 18, "failed": 1, "skipped": 1, "total": 20}
{"type": "cancelled"}
```

## Arquitetura do spotify_downloader.py


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [afrikasa/playlistr](https://github.com/afrikasa/playlistr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-22 -->
