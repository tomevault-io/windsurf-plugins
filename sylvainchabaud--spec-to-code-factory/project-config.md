---
trigger: always_on
description: Pipeline automatisé qui transforme un requirements.md en projet livrable.
---

# CLAUDE.md — Spec-to-Code Factory

## Vision
Pipeline automatisé qui transforme un requirements.md en projet livrable.

## Workflow obligatoire
```
       BREAK → MODEL → ACT → DEBRIEF
  │      │        │       │       │
Gate 0  Gate 1  Gate 2  Gate 3+4  Gate 5
```

### Validations par Gate
| Gate | Phase | Validations |
|------|-------|-------------|
| 0 | →BREAK | **requirements.md complet** (12 sections obligatoires) |
| 1 | BREAK→MODEL | Fichiers brief/scope/acceptance + **structure projet** |
| 2 | MODEL→ACT | Specs + ADR + **scan secrets/PII** |
| 3 | PLAN→BUILD | Epics + US + Tasks avec DoD |
| 4 | BUILD→QA | Tests passants + **code quality strict** + **app assembly** + **boundary check** |
| 5 | QA→RELEASE | QA report + checklist + CHANGELOG + **export release** |

### Comportement des Gates
- **Mode JSON** : `node tools/gate-check.js <N> --json` retourne `{ gate, status, errors[], summary }` avec classification (`category`, `fixable`)
- **Gate d'entree** (prereq phase) : Si FAIL → STOP immediat, marqueur `GATE_FAIL|<N>|<erreurs>|0`
- **Gate de sortie** (outputs phase) : Auto-remediation 3x puis `GATE_FAIL|<N>|<erreurs>|3`
- **Orchestrateur** : Detecte `GATE_FAIL` et propose a l'utilisateur : relancer / corriger / abandonner

## Phases
1. **BREAK** : Normaliser le besoin → brief + scope + acceptance
2. **MODEL** : Spécifier → specs + ADR + rules
3. **ACT** : Planifier + Construire → epics + US + tasks + code + tests
4. **DEBRIEF** : Valider + Livrer → QA + checklist + CHANGELOG

## Invariants (ABSOLUS)
- **No Spec, No Code** : Pas de code sans specs validées
- **No Task, No Commit** : Chaque commit référence TASK-XXXX
- **Tasks auto-suffisantes** : Chaque task est 100% indépendante (principe BMAD)

## Conventions de nommage
- Epics : `EPIC-XXX` (dans `docs/planning/epics.md`)
- User Stories : `US-XXXX-titre.md`
- Tasks : `TASK-XXXX-titre.md`
- ADR : `ADR-XXXX-titre.md`

## Commands disponibles
### Skills (workflows)
- `/factory-intake` : Phase BREAK
- `/factory-spec` : Phase MODEL
- `/factory-plan` : Phase ACT (planning)
- `/factory-build` : Phase ACT (build)
- `/factory-qa` : Phase DEBRIEF
- `/factory` : Pipeline complet (auto-detect greenfield V1 / brownfield V2+)
- `/factory-quick` : Quick fix/tweak sans pipeline complet (BMAD Quick Flow)
- `/factory-resume` : Reprend le pipeline apres interruption
- `/gate-check [0-5]` : Verifie un gate
- `/clean` : Remet le projet en état "starter" (supprime tous les artefacts)

### Commands
- `/status` : État du pipeline (dashboard détaillé)
- `/reset [phase]` : Réinitialise une phase
- `/help` : Affiche l'aide

## Outils de support
- `tools/validate-requirements.js` : Validation requirements.md complet (Gate 0)
- `tools/factory-state.js` : Gestion de l'etat machine-readable + compteurs
- `tools/factory-reset.js` : Reset des phases
- `tools/detect-requirements.js` : Detection automatique du dernier requirements-N.md
- `tools/get-planning-version.js` : Obtenir le repertoire planning actif (vN)
- `tools/set-current-task.js` : Tracking de la task courante
- `tools/validate-code-quality.js` : Validation code vs specs (mode STRICT)
- `tools/validate-structure.js` : Validation structure projet (Gate 1)
- `tools/scan-secrets.js` : Scan secrets et PII (Gate 2)
- `tools/validate-app-assembly.js` : Validation assemblage App.tsx (Gate 4) - **supporte Clean Arch**
- `tools/validate-boundaries.js` : Validation des regles d'import inter-couches (Gate 4)
- `tools/export-release.js` : Export du projet livrable (Gate 5)
- `tools/list-active-adrs.js` : Listing et filtrage des ADR (actifs, superseded, par version)
- `tools/extract-version-delta.js` : Extraction du delta d'une version depuis les specs (inline + marqueurs de bloc)
- `tools/verify-pipeline.js` : Verification post-pipeline complete (toutes phases)
- `tools/clean.js` : Reset projet en etat starter (`--force`, `--dry-run`)

## Permissions simplifiées

Les permissions dans `.claude/settings.json` ont été simplifiées pour réduire les prompts :

```json
"permissions": {
  "allow": ["Read(*)", "Write(*)", "Edit(*)", "Glob(*)", "Grep(*)", "Bash(node tools/*)", ...],
  "ask": ["Bash(git add:*)", "Bash(git commit:*)"],
  "deny": ["Read(.env)", "Write(.env)", ...]
}
```

Seuls les commits git demandent confirmation. Les opérations Read/Write/Edit sont autorisées par défaut.

## Hook Git optionnel

`tools/validate-commit-msg.js` valide le format des commits (TASK-XXXX: description).

**Installation (optionnel)** :
```bash
# Avec Husky
npx husky add .husky/commit-msg "node tools/validate-commit-msg.js $1"

# Ou manuellement
echo 'node tools/validate-commit-msg.js "$1"' > .git/hooks/commit-msg
chmod +x .git/hooks/commit-msg
```

## Instrumentation (optionnel)

Le pipeline peut tracer tous les événements pour debugging ou audit.

**Activation** dans `.claude/settings.json` :
```json
"env": {
  "FACTORY_INSTRUMENTATION": "true"
}
```

**Evenements trackes** :

Les evenements sont collectes par deux mecanismes complementaires :
- **Hooks** (automatiques) : evenements generiques fires par Claude Code
- **Tools** (explicites) : evenements metier qui necessitent le resultat de l'operation

| Type | Description | Source | Mecanisme |
|------|-------------|--------|-----------|

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [SylvainChabaud/spec-to-code-factory](https://github.com/SylvainChabaud/spec-to-code-factory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
