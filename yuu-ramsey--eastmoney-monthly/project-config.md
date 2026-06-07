---
trigger: always_on
description: > **Project Charter**: [.claude-charter.md](.claude-charter.md) (must read before every task)
---

# Eastmoney Monthly Chart AI Analysis Assistant

> **Project Charter**: [.claude-charter.md](.claude-charter.md) (must read before every task)

Chrome MV3 extension, injected into Eastmoney stock pages (quote.eastmoney.com),
analyzing stock trends via LLM API. Local personal use only, not published to Chrome Web Store.

## Regime-Adaptive MoE Architecture (designed 2026-06-05)

### Design Principles

- **No single point of failure**: Any component failure does not affect the whole
- **LLM isolation**: LLM is outside the signal chain, only does interpretation
- **Mechanical lookup table**: regime→weights are hardcoded, not passing through any model
- **3D expert grid**: Expert = regime × scale × perspective intersection, not model names

### Expert Registry (3D key)

```python
expert_registry = {
    # key = (signal_id, regime, scale)
    # status: verified | anti_signal | pending_validation | pending_feature
    ('momentum', 'bear', 'monthly'):     {'spread': +20.6, 'ci': [10.8, 32.4], 'status': 'verified'},
    ('momentum', 'bull', 'monthly'):     {'spread': -42.2, 'ci': [-94.5, -2.5], 'status': 'anti_signal'},
    ('kronos', 'all', 'monthly'):        {'spread': +9.7, 'ci': [5.1, 15.3], 'status': 'verified'},
    ('reversal', 'sideways', 'monthly'): {'spread': None, 'status': 'pending_validation'},
    ('llm_strong', 'all', 'monthly'):    {'spread': None, 'status': 'pending_deconservative'},
    ('gru_wf', 'bear', 'daily'):         {'spread': None, 'status': 'pending_validation'},
    ('lgb', 'bear', 'daily'):            {'spread': None, 'status': 'pending_validation'},
    ('debate', 'high_vol', 'monthly'):   {'spread': None, 'status': 'pending_validation'},
    ('resonance', 'transition', 'multi'):{'spread': None, 'status': 'pending_validation'},
    ('quant_32d', 'all', 'monthly'):     {'spread': None, 'status': 'pending_validation'},
    ('sector_alpha', 'bull', 'monthly'): {'spread': None, 'status': 'pending_validation'},
    ('money_flow', 'all', 'daily'):      {'spread': None, 'status': 'pending_feature'},
}
```

### Gating Lookup Table (mechanical, zero LLM)

```python
def get_weights(regime, scale):
    """Only activate experts with status=='verified' under this (regime, scale)"""
    active = {k: v for k, v in expert_registry.items()
              if k[1] in (regime, 'all') and k[2] in (scale, 'multi')
              and v['status'] == 'verified'}
    # Assign weights by spread magnitude, each ≤40%, normalize
    return normalize_by_spread(active)
```

### LLM Role in Signal Chain

| Layer | Has LLM? | Role |
|----|---------|------|
| Regime Detection | ❌ | Three-detector statistical vote |
| Gating | ❌ | Hardcoded lookup table |
| Signal Aggregation | ❌ | Weighted average |
| Interpretation Output | ✅ | Translate to plain language |

### Pending Validation Slots (per-regime split tasks)

| Signal | Current | Target |
|------|------|------|
| S8 Reversal | 24tp overall −19.2% | per-regime breakdown, may be effective under sideways |
| S6 LGB | bull market −42.2% / bear market +20.6% | split into (lgb, bear, daily) and (lgb, bull, daily) |
| S7 Momentum | already has per-regime | already split→verified |
| S5 GRU | Level 2 boundary | split into (gru_wf, bear, daily) re-verify |
| S1 LLM | 70% neutral | extract (llm_strong, all, monthly) strong labels only |

## Core Architecture

- **content.js**: Injects Shadow DOM-isolated FAB + side panel, scrapes "Major Events"
  from page DOM
- **background.js** (service worker): Fetches Eastmoney monthly/weekly/daily charts + capital flow + calls
  LLM API + cache
- **lib/llm/**: Provider abstraction layer, supports Anthropic and DeepSeek
- **lib/agents/**: Multi-agent architecture Bull/Bear/Predictor/Judge
- **lib/build-prompt.js**: 4 styles (technical/chanlun/value/comprehensive)
  + multi-period resonance prompt assembly

## Key Design Decisions (do not overturn lightly)

### Path Y: Decision aid, not investment advice

The prompt does not directly say "recommend buy/sell"; conclusions use conditional expressions like "if X then verify Y".
Only when `decisionMode = true` does it append a "Personal Decision Perspective" paragraph with explicit suggestions (local
personal use only, to avoid compliance risk).

### Cache Key Structure

`analysis:<market>.<code>:<period>:<bucket>:<style>:<mode>:<decision>`

- bucket uses different granularity by period (monthly=YYYY-MM / weekly=YYYY-WW /
  daily=YYYY-MM-DD)
- No provider dimension——avoids massive duplicate token consumption when user switches providers
- Switching style / period / decisionMode all trigger new cache

### Multi-Provider Isolation

- API keys stored separately: apiKey:anthropic / apiKey:deepseek
- Model fields stored separately: model:anthropic / model:deepseek
- Anthropic defaults to claude-sonnet-4-6, DeepSeek defaults to deepseek-chat
- When switching providers in popup, onProviderChange reads storage directly, **does NOT call
  loadSettings** (avoids the bug where select.value gets overwritten by old value)

### Chanlun style uses FULL strictness constraints (not differentiated by provider)

Previously tried using LITE relaxed constraints for DeepSeek, but output quality was worse (produced

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [yuu-ramsey/eastmoney-monthly](https://github.com/yuu-ramsey/eastmoney-monthly) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-07 -->
