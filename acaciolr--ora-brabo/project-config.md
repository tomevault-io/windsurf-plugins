---
trigger: always_on
description: > Leia este arquivo inteiro antes de qualquer ação no projeto.
---

# ORA BRABO — CLAUDE CODE BRIEFING
> Leia este arquivo inteiro antes de qualquer ação no projeto.

---

## O QUE É ESTE PROJETO

**ORA BRABO Monitoring Tool** — TUI (Terminal User Interface) para Oracle Database, inspirada no Dolphie para MySQL, com funcionalidades equivalentes ao Oracle Enterprise Manager (OEM). Roda 100% em terminal Linux via SSH.

**Autor:** Acacio Lima Rocha (DBA BRABO)
**Stack:** Python 3.12+, Textual, Rich, oracledb (Thin Mode por padrão, Thick Mode opcional), AsyncIO
**Versão atual:** 1.3.3 — multi-tab, thick mode, cache priming (ver `core/version.py` / `pyproject.toml`)

---

## ESTRUTURA DO PROJETO

```
ora_brabo/
├── app.py                        # Entry point, OraBraboApp (Textual App), bindings F1-F12 + Ctrl+1-9
├── ora_brabo.tcss                # Dark theme CSS (GitHub dark palette)
├── requirements.txt              # oracledb, textual, rich, plotext, keyring
├── requirements-optional.txt     # reportlab+pillow — só para export de PDF (F-Report)
├── README.md
│
├── core/
│   ├── config.py                 # AppConfig dataclass (conexão padrão, wallet/ADB, thick mode, timeouts)
│   ├── connection_manager.py     # Pool async oracledb (Thin ou Thick), execute_query/ddl/fetch_one
│   ├── connection_session.py     # Bundle por aba: 1 conn_manager + 1 cache + 1 scheduler + 1 advisor
│   ├── connections_store.py      # Histórico de conexões (~/.ora_brabo/connections.json)
│   ├── demo_data.py              # DemoRunner — popula o cache com dados simulados (--demo, sem Oracle)
│   ├── cache.py                  # MetricsCache: TTL + ring-buffer 120 pontos, thread-safe
│   ├── scheduler.py              # Async scheduler por sessão, 17 collectors em tasks paralelas
│   └── version.py                # __version__ + BANNER_ART (fonte única de versão)
│
├── collectors/                   # 17 collectors ativos no scheduler (ver tabela abaixo)
│   ├── base.py                   # BaseCollector ABC
│   ├── health.py / sessions.py / sql.py / waits.py / rac.py / sqlmon.py
│   ├── dg.py / asm.py / rman.py / io_activity.py / pdb.py
│   ├── exadata.py / advisor.py / memory_advisor.py
│   └── awr.py / objects.py / alertlog.py
│
├── widgets/
│   ├── panels.py                 # 24 painéis Textual (F1-F12, Ctrl+1-9, x, p)
│   ├── charts.py                 # sparkline() e helpers de gráfico (plotext)
│   ├── add_connection_modal.py   # Tela de nova conexão (multi-tab)
│   ├── confirm_modal.py          # Modal de confirmação para Kill/Trace
│   ├── explain_screen.py         # Overlay de Explain Plan (F3 → 'e')
│   ├── session_detail_screen.py
│   ├── sql_input_screen.py
│   ├── text_view_screen.py
│   └── help_screen.py
│
├── advisor/
│   └── engine.py                 # AdvisorEngine: regras contínuas, Finding dataclass, Severity enum
│                                  # (instanciado por ConnectionSession, um por aba/conexão)
│
└── shell/
    ├── collect_rac.sh            # srvctl / crsctl / olsnodes → JSON stdout
    ├── collect_dg.sh             # dgmgrl show configuration
    └── collect_asm.sh            # asmcmd lsdg
```

> Nota: arquivos com sufixo `" 2.py"` (ex.: `exadata 2.py`, `connection_manager 2.py`) são cópias de conflito criadas pelo sync do iCloud Drive — não são rastreados pelo git (`.gitignore`) e podem ser ignorados/apagados com segurança.

---

## ARQUITETURA MULTI-TAB (mudança estrutural desde a v1.0.0)

O app deixou de ser single-connection. Cada aba é um `ConnectionSession` (`core/connection_session.py`) que empacota:
- seu próprio `ConnectionManager` (pool de conexão)
- seu próprio `MetricsCache`
- seu próprio `Scheduler` (17 collectors)
- seu próprio `AdvisorEngine`
- um health-check loop (`_health_check_loop`) que faz `SELECT 1 FROM DUAL` a cada 15s e tenta reconectar após 3 falhas consecutivas

`OraBraboApp` (`app.py`) gerencia N sessões simultâneas via `Tabs`/`ContentSwitcher`. Nova conexão: tecla `+` (ou `Ctrl+N`/`Ctrl+O` como fallback para terminais que engolem essas teclas) abre `AddConnectionModal`. Fechar aba: `Ctrl+W`.

Modo demo (`--demo`) usa `DemoRunner` (`core/demo_data.py`, ~1100 linhas) em vez de scheduler+conn_manager — popula o cache com dados fake, sem precisar de Oracle.

---

## COLLECTORS ATIVOS E TIERS DE INTERVALO

Definidos em `core/scheduler.py`. Tiers calculados a partir do `refresh_interval` da sessão:
- `rt` (realtime) = `min(refresh, 2)` — Health, Waits
- `fast` = `max(refresh, 3)` — Sessions, SQL, RAC, SQL Monitor
- `med` = `max(refresh*2, 12)` — Data Guard, ASM, RMAN, I/O, PDB
- `slow` = `30` — Exadata, Advisor findings, Memory Advisor
- `heavy` = `60` — AWR, Objects, Alert Log (scans caros, mantidos espaçados para proteger produção)

| Collector | Tier | Painel(is) que consome |
|---|---|---|
| HealthCollector | rt | Dashboard (F1) |
| WaitsCollector | rt | Waits (F4), Dashboard |
| SessionsCollector | fast | Sessions (F2), Locks (F5) |
| SQLCollector | fast | Top SQL (F3) |
| RACCollector | fast | RAC (F6) |
| SQLMonitorCollector | fast | SQL Monitor (Ctrl+4) |
| DataGuardCollector | med | Data Guard (F7) |
| ASMCollector | med | ASM (F8) |
| RMANCollector | med | RMAN (F9) |
| IOActivityCollector | med | I/O (Ctrl+1) |
| PDBCollector | med | PDB (`p`) |
| ExadataCollector | slow | Exadata (`x`) |
| AdvisorCollector | slow | Advisor (F12) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [acaciolr/ora-brabo](https://github.com/acaciolr/ora-brabo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-19 -->
