---
trigger: always_on
description: Contexto estático para desenvolvimento assistido por IA. Leia antes de qualquer mudança.
---

# CLAUDE.md — 3DSort

Contexto estático para desenvolvimento assistido por IA. Leia antes de qualquer mudança.
Última revisão: 2026-08-16 (suporte a CARTÃO MULTI-CONSOLE: id0 desempatado pelo
movable, script de dump console-agnóstico, container alheio degrada para read-only,
ver §5.1/§5.8/§10. Antes: Fase 4 e gates de hardware 0B/0C, §10).

## 1. O que é o projeto

**3DSort** é um app desktop (Windows-first) que reorganiza o layout do HOME menu do
Nintendo 3DS editando o SD card do console montado no PC: reordenar ícones por
drag-and-drop, mover jogos entre pastas, presets de ordenação, preview ao vivo de como
ficará no console, escrita staged com backup automático e histórico restaurável.

- **Público**: comunidade 3DS (consoles com CFW — Luma3DS + GodMode9). Distribuído
  como **exe portátil sem instalador** (PyInstaller onefile, `3DSort.spec`, §10).
  Windows-first no empacotamento; o código roda no Linux a partir do fonte
  (detecção de mount + nome do binário são platform-aware, §10/README).
- **Protótipo visual de referência**: `prototype/3DSort Prototype.dc.html` (com
  `prototype/support.js`, que é apenas o runtime do mockup — ignorar como código de
  produção). A UI real em `ui/` porta esse visual; em dúvida de UX/estética, consultar o
  protótipo.
- **Escopo v1**: abas GRID, SYNC, INSTRUCTIONS e SETTINGS; ícones reais dos jogos (requisito firme do
  usuário); staging/undo/redo; backups. Abas RULES (auto-sort por regras) e THEMES/badges
  ficaram para v2. v1.1 (implementado e validado em hardware, §10): reordenar apps
  NAND/pastas/Game Card e criar/renomear/apagar pastas via escrita do Launcher.dat com
  injeção assistida por GodMode9; desembrulho automático e preservação de tema.

## 2. Arquitetura (decidida e aprovada — não reabrir sem motivo novo)

**Backend Python + UI HTML via pywebview + binário `save3ds` embarcado.**

- Webapp hospedado foi **descartado**: servidor não acessa o SD local do usuário.
- Tauri/Rust descartado: save3ds já resolve a parte difícil como CLI; reescrever o backend
  em Rust não traz ganho funcional.
- Webapp estático + File System Access API descartado: Chromium-only e exigiria
  reimplementar a criptografia do 3DS em JS.
- A UI conversa com a MESMA classe `Api` por dois canais: ponte `js_api` do pywebview
  (app real, janela nativa via WebView2) e modo dev `--serve` (stdlib `http.server`,
  `POST /api/<metodo>` com corpo `{"args": [...]}` posicional). O modo `--serve` existe
  para testes Playwright e desenvolvimento.

### Mapa do repositório

```
F:\Projects\3DSort\
├── CLAUDE.md               ← este arquivo
├── 3DSort.spec             ← build PyInstaller onefile/windowed (Fase 4)
├── LICENSE                 ← GPL-3.0 (mesma do Cthulhu, de onde veio o desembrulho §5.4)
├── requirements.txt        ← runtime (pywebview, Pillow, pyctr)
├── requirements-dev.txt    ← -r requirements.txt + pytest + pyinstaller
├── docs/images/            ← screenshots do README (capturadas em --mock: SEM dados
│                             do console real; `/*.png` do .gitignore é ancorado na raiz)
├── prototype/              ← mockup visual de referência (não é código de produção)
├── app.py                  ← Api (camada única UI↔core), FakeSave3ds/mock, --serve, --selftest, main
├── spike.py                ← prova de viabilidade da Fase 1 (histórico; já cumpriu o papel)
├── conftest.py             ← vazio; existe só para o pytest achar core/ no sys.path
├── core/
│   ├── savedata.py         ← parse/serialize do SaveData.dat (layout do HOME menu)
│   ├── launcher.py         ← classe Launcher: parse/serialize do Launcher.dat (NAND)
│   ├── icons.py            ← Cache.dat/CacheD.dat → nomes + ícones PNG base64 (SMDH)
│   ├── store.py            ← Staging (undo/redo por snapshot) e Backups (.3dsl + jsonl)
│   ├── sdcard.py           ← detecção SD/console/região + wrapper save3ds (--sdext e --nandsave)
│   ├── titledates.py       ← tid → data de lançamento (tabela offline embutida)
│   └── titledates.json.gz  ← tabela gerada por tools/build_titledates.py (COMMITADA; ~16KB)
├── ui/
│   ├── index.html          ← tela única, CSS fiel ao protótipo (paleta creme/DotGothic16)
│   ├── app.js              ← JS puro; render por innerHTML + bind(); estado P (prefs) + S (backend)
│   ├── fonts/              ← Nunito (variável) + DotGothic16 woff2 latin, servidas offline
│   └── 3dsort.ico          ← ícone do exe e favicon (gerado por tools/make_icon.py)
├── tests/
│   ├── test_savedata.py    ← unit: round-trip binário, invariantes
│   ├── test_launcher.py    ← unit: parse + round-trip/diff-locality/lifecycle do Launcher
│   ├── test_icons.py       ← unit: decode Morton/RGB565 (com encoder inverso), nomes SMDH
│   ├── test_store.py       ← unit: staging, backups, prune, histórico
│   ├── test_sdcard.py      ← unit: console/região; id0 do movable; árvore NAND sintética
│   ├── test_api_state.py   ← unit: merge launcher/SD no get_state (mock)
│   ├── test_api_launcher_edit.py ← unit: swaps entre tipos, lifecycle de pastas, inject
│   ├── test_titledates.py  ← unit: tabela de datas + presets de sort por data
│   ├── test_api_setup.py   ← unit: get_setup_state (estágios do wizard de onboarding)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SalustLab/3DSort](https://github.com/SalustLab/3DSort) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
