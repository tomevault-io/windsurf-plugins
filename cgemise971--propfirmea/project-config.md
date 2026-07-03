---
trigger: always_on
description: Ce projet vise a creer un systeme automatise d'Expert Advisors (EA) optimise pour:
---

# CLAUDE.md - Projet PropFirm EA Fleet

## Vue d'Ensemble du Projet

Ce projet vise a creer un systeme automatise d'Expert Advisors (EA) optimise pour:
1. **Passer les challenges** des principales prop firms (FTMO, E8 Markets, Funding Pips, The5ers)
2. **Generer des profits stables** de 5-10% mensuel une fois finance
3. **Scaler** vers une flotte de comptes multi-prop firms

---

## Architecture du Projet

```
PropFirmEA_Project/
├── CLAUDE.md                           # Ce fichier - Directives principales
├── README.md                           # Guide d'installation rapide
│
├── EA/MQL5/                            # Expert Advisors
│   ├── PropFirm_Scalper_v8.mq5        # Scalper V8 [RECOMMANDE CHALLENGES]
│   ├── PropFirm_SMC_EA_v1.mq5         # Strategie SMC/ICT
│   ├── PropFirm_SessionBreakout_v1.mq5 # Breakout V1 (basique)
│   ├── PropFirm_SessionBreakout_v2.mq5 # Breakout V2 (dynamique)
│   ├── PropFirm_SessionBreakout_v3.mq5 # Breakout V3 (qualite)
│   ├── PropFirm_SessionBreakout_v4.mq5 # Breakout V4 (scanner)
│   ├── PropFirm_SessionBreakout_v5.mq5 # Breakout V5 (structure)
│   ├── PropFirm_SessionBreakout_v6.mq5 # Breakout V6 (optimizer)
│   └── PropFirm_SessionBreakout_v7.mq5 # Breakout V7 (adaptive)
│
├── EA/MQL5/Include/                    # Modules partages
│   └── Dashboard_v2.mqh               # Dashboard compact lisible
│
├── strategies/                         # Documentation des strategies
│   ├── SMC_ICT_Strategy.md
│   ├── Session_Breakout.md
│   └── Scalper_Strategy.md            # Scalper V8 documentation
│
├── config/profiles/                    # Presets par prop firm
│   ├── Scalper_FTMO_Challenge.set     # V8 Scalper presets
│   ├── Scalper_FTMO_Funded.set
│   ├── Scalper_E8_OneStep.set
│   ├── Scalper_FundingPips_1Step.set
│   ├── Scalper_The5ers_Bootcamp.set
│   ├── FTMO_Normal_Challenge.set
│   ├── FTMO_Normal_Funded.set
│   ├── E8_One_Step.set
│   ├── FundingPips_1Step.set
│   ├── The5ers_Bootcamp.set
│   ├── SessionBreakout_FTMO_Challenge.set
│   └── SessionBreakout_E8_OneStep.set
│
├── deployment/                         # Scripts de deploiement RDP
│   ├── bootstrap_rdp.ps1              # Installation one-liner
│   ├── setup_auto_deploy.ps1          # Config auto-sync
│   └── RDP_INSTALLATION_GUIDE.md
│
├── backtests/                          # Outils de backtest
│   ├── BacktestAnalyzer.mq5
│   ├── BacktestConfig.mqh
│   ├── analyze_backtest.py
│   └── propfirm_validator.py
│
├── docs/                               # Documentation
│   ├── PROP_FIRMS_RULES.md
│   ├── RISK_MANAGEMENT.md
│   └── FLEET_SCALING_STRATEGY.md
│
└── Logs/                               # Logs de deploiement
```

---

## Expert Advisors Disponibles

### 1. PropFirm_Scalper_v8 (RECOMMANDE CHALLENGES)
**Statut**: Implementé - Scalping haute frequence

| Element | Description |
|---------|-------------|
| Strategie | Scalping multi-paires, 4 types d'entrees |
| Timeframe | M5 |
| Paires | EURUSD, GBPUSD, USDJPY, XAUUSD |
| Sessions | London Open/Peak, NY Open/Peak, London Close |
| Risk | 0.5-0.8% par trade |
| Capacite | 12-15 trades/jour, 2 positions max |

**4 Types d'Entrees:**
- **MOMENTUM** - Bougie forte > 40% range horaire
- **MICRO_BREAKOUT** - Cassure range 1H + 3 pips
- **PULLBACK** - Retour sur EMA21 dans tendance
- **REVERSAL** - RSI extreme + pin bar (optionnel)

**Caracteristiques Cles:**
- Compounding agressif (+25% apres 3 wins, +50% apres 5 wins)
- Mode Turbo si retard challenge (auto-adaptatif)
- Dashboard compact V2 (lisible, 8 lignes)
- Exit time-based (20 min max)
- Partial close 50% a TP1, trail le reste

**Performance Cible:** 10-15% mensuel

### 2. PropFirm_SMC_EA_v1 (Smart Money Concepts)
**Statut**: Implementé - En test

| Element | Description |
|---------|-------------|
| Strategie | Order Blocks, FVG, BOS/CHoCH |
| Timeframes | H4 (tendance) + M15 (entree) |
| Sessions | London & NY Kill Zones |
| Risk | 1.5% challenge / 0.75% funded |

### 3. PropFirm_SessionBreakout (v1-v7)
**Statut**: Anciennes versions - Remplacees par V8

| Version | Description | Limite |
|---------|-------------|--------|
| V7 | Adaptive mode | Trop conservateur |
| V6 | Challenge optimizer | ~2-3% mensuel |
| V5 | Structure-based | Peu de trades |
| V4 | Multi-opportunity | 6 trades/jour max |
| V3 | Quality scoring | Tres selectif |
| V2 | Dynamic range | Multiple sessions |
| V1 | Basic breakout | Range fixe |

> Note: Ces versions sont conservees pour reference mais le **Scalper V8** est recommande pour les challenges.

---

## Deploiement RDP (Auto-Sync)

### Installation initiale (une seule fois)
```powershell
Set-ExecutionPolicy Bypass -Scope Process -Force
irm "https://raw.githubusercontent.com/cgemise971/PropFirmEA/main/deployment/bootstrap_rdp.ps1" | iex
```

### Activer l'auto-sync
```powershell
cd C:\PropFirmEA\Project
git pull
powershell -EP Bypass -File deployment\setup_auto_deploy.ps1
```

### Verifier le service
```powershell
Get-ScheduledTask -TaskName "PropFirmEA_AutoSync" | Select-Object State
```

**Fonctionnement:**
- Verifie GitHub toutes les 60 secondes
- Deploie automatiquement les changements vers MT5
- Logs dans `C:\PropFirmEA\Logs\sync.log`

---

## Workflow de Developpement

```

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cgemise971/PropFirmEA](https://github.com/cgemise971/PropFirmEA) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
