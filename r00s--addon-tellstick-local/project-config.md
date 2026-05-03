---
trigger: always_on
description: ████████████████████████████████████████████████████████████████████████████████
---

# TellStick Local – Copilot Instructions

```
████████████████████████████████████████████████████████████████████████████████
█                                                                              █
█   🛑🛑🛑 READ THIS FIRST — VERSION BUMPING RULES 🛑🛑🛑                    █
█                                                                              █
█   manifest.json version = A.B.C.D                                           █
█                                                                              █
█   A, B  — set by the USER only.  Agents NEVER touch A or B.                █
█   C     — bump when the git branch name changes; reset D to 0.              █
█   D     — bump whenever making code changes on the current branch.          █
█                                                                              █
█   ⚠️  DO NOT hardcode a version here — it will always be stale.             █
█   ALWAYS read manifest.json to get the current version, then bump D.        █
█   Run `git branch --show-current`:                                          █
█     same branch name  → bump D only                                         █
█     different branch  → bump C, reset D to 0                                █
█                                                                              █
█   config.yaml version MUST ALWAYS be 'dev' on branches (linter-enforced)    █
█                                                                              █
████████████████████████████████████████████████████████████████████████████████
```

```
████████████████████████████████████████████████████████████████████████████████
█                                                                              █
█   🛑 CRITICAL: SOURCE OF TRUTH FOR DIFFERENT DATA 🛑                        █
█                                                                              █
█   ADD-ON CONFIG (devices, protocols):                                        █
█     → Edit: tellsticklive/config.yaml                                        █
█     → Reflected in: tellsticklive/rootfs/etc/cont-init.d/telldusd.sh         █
█                                                                              █
█   INTEGRATION CONFIG (HA platforms, entities):                               █
█     → Edit: custom_components/tellstick_local/<platform>.py                  █
█     → Constants: custom_components/tellstick_local/const.py                  █
█                                                                              █
█   PROTOCOL: TEXT-BASED (telldusd socket encoding):                        █
█     → Edit: custom_components/tellstick_local/client.py                      █
█     → NEVER use binary framing — protocol is text-based                      █
█                                                                              █
████████████████████████████████████████████████████████████████████████████████
```

## Branch Timeline Files

```
████████████████████████████████████████████████████████████████████████████████
█                                                                              █
█   🛑 READ THE BRANCH TIMELINE BEFORE DOING ANYTHING 🛑                      █
█                                                                              █
█   Every branch has a timeline file:                                         █
█     docs/<branch-name-without-prefix>-<A.B.C.x>.md                         █
█                                                                              █
█   Example: branch copilot/retrieve-rtl-conf-luxorparts, version 3.1.12.5   █
█     → docs/retrieve-rtl-conf-luxorparts-3.1.12.x.md                        █
█                                                                              █
█   HOW TO FIND IT:                                                            █
█     BRANCH=$(git branch --show-current | sed 's|.*/||')                     █
█     VERSION=$(jq -r '.version' custom_components/tellstick_local/manifest.json | cut -d. -f1-3) █
█     FILE="docs/${BRANCH}-${VERSION}.x.md"                                   █
█                                                                              █
█   RULES:                                                                     █
█     1. Read it FIRST — before reading any other file or writing code.        █
█     2. If it does not exist, CREATE it before doing anything else.           █
█     3. After each discovery, failed attempt, or implemented fix — UPDATE it. █
█     4. This prevents debug loops and lost discoveries across agent sessions. █
█                                                                              █
████████████████████████████████████████████████████████████████████████████████
```

## Quick Commands

```bash
# Lint YAML files
yamllint tellsticklive/config.yaml

# Lint shell scripts (-s bash because of bashio shebang)
shellcheck -s bash tellsticklive/rootfs/etc/services.d/telldusd/run
shellcheck -s bash tellsticklive/rootfs/etc/cont-init.d/telldusd.sh

# Check Python syntax and unused imports
python -m py_compile custom_components/tellstick_local/*.py
python -m pyflakes custom_components/tellstick_local/

# Check integration version

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [R00S/addon-tellstick-local](https://github.com/R00S/addon-tellstick-local) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-02 -->
