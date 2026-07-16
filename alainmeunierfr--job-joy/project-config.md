---
trigger: always_on
description: Après chaque modification de code, sans exception :
---

# CLAUDE.md — Job Joy

## RÈGLE ABSOLUE — Qualité non négociable

Après chaque modification de code, sans exception :
1. `npm run lint` → 0 erreur, 0 warning sur les fichiers modifiés
2. Tout e2eID renommé/supprimé → mettre à jour les steps BDD correspondants
3. Lancer `C:\dev\job-joy\.claude\craft-ctrl\craft-ctrl.exe` → traiter **toutes** les anomalies. Boy scout : laisser le rapport meilleur qu'à l'arrivée.

Pas de "dette pré-existante" : ClaudeCode est le seul développeur, toutes les non-conformités sont de sa responsabilité.

---

## Présentation

**Job Joy** — Application Electron + Node.js (TypeScript) : collecte offres d'emploi (IMAP, HTML), analyse IA (Claude/Mistral), base SQLite locale, interface React embarquée.
**Stack** : TypeScript · Jest · Playwright/Cucumber (Gherkin fr) · SQLite (`better-sqlite3`) · Electron · Express · React 19 + Tailwind v4 + shadcn/ui.

---

## Mode autonome

Projet personnel — opérer sans confirmation pour les actions locales (git, fichiers, npm). Confirmer uniquement si l'action est irréversible et à fort impact.

**Une branche locale par US** : `us/X-NN` créée au `GO US`, fusionnée à `US VALIDEE`.
- `git checkout -b us/X-NN` au départ · commits WIP libres sur la branche · `git merge && git branch -d` à la clôture
- **Avant tout geste destructif** : `git commit -m "wip: checkpoint avant [opération]"` — obligatoire

---

## Commandes de pilotage

Tu es le **Lead Dev** — orchestre le tunnel, fait les revues, maintient l'état des US en session.
Avant chaque délégation : résumer l'étape, proposer le plan, attendre validation (sauf en `GO FIN`).

### Tunnel

| Commande | Effet |
|----------|-------|
| **`GO US`** | Agent US : reformuler en US + CA, écrire le fichier MD, créer branche `us/X-NN`. Voir `.claude/commands/agent-us.md`. |
| **`GO NEXT`** | Avancer d'une étape : US → IMPACT → BDD → TDD-back → TDD-front → IMPACT-VERIFY → US REVIEW. |
| **`GO FIN`** | Enchaîner GO NEXT sans pause jusqu'à US CONTROL (back-end pur) ou US REVIEW (avec UI) — revues en autonome, s'interrompre si arbitrage nécessaire. |
| **`US REVIEW`** | Présenter les CA livrés, points notables. `@workflow: en review`. Attendre feedback utilisateur. **Protocole feedback → voir ci-dessous.** Quand l'utilisateur valide → US CONTROL. |
| **`US CONTROL`** | DoD complète (`.claude/Software craftsmanship/7.…`). Invoquer `/go-tests <US-X.NN>`. Vérifier lien US↔BDD, captures, guardrail CQRS:ES (`grep db.prepare.*INSERT\|UPDATE` hors `src/infrastructure/` ou `src/kernel/`). 0 anomalie craft-ctrl. Rapport pass/fail + **plan de test manuel** (voir ci-dessous). |
| **`US VALIDEE`** | Clôture. Prérequis : US CONTROL ✅. `git checkout release-windows && git merge us/X-NN && git branch -d us/X-NN`. Puis déplacer le fichier US : `git mv ".claude/sprints/Sprint XX/US-X.NN - Titre.md" "commun/docs/in-appli/documentation/product-management/Sprint XX/US-X.NN - Titre.md"` + commit. |

### Skills (invoquer avec `/nom`)

| Skill | Comportement |
|-------|--------------|
| `/go-tests <US-X.NN>` | Lint + Jest pattern US + BDD `@tag` — pour US CONTROL. `--all` = suite complète pour `/go-publish`. |
| `/go-publish [0-3]` | Version → tests → BDD → doc → metrics → bump → build → commit → tag → push. |
| `/go-doc` | Doc référence + changelog utilisateur. |
| `/go-ds <chemin>` | Audit AST + corrections design system (shadcn, G01–G05). |
| `/go-control-bdd` | Doublons steps + orphelins. Avant toute refacto steps. |
| `/go-controles` | Registre outils qualité. Obligatoire avant modif hook/script/skill. |
| `/go-audit-id-us` | Renommages US en attente → audit références → régénère plan sprints. |

> Menu.ps1 pour process bloquants : serveurs dev (1-2), publication Electron (3-6), BDD site (7), publication site (8).

---

## Le tunnel

| Livrable | Étapes |
|----------|--------|
| Domaine + CLI | US → IMPACT → TDD-back → IMPACT-VERIFY → US CONTROL → US VALIDEE |
| Domaine + UI | US → IMPACT → BDD → TDD-back → TDD-front → IMPACT-VERIFY → US REVIEW → US CONTROL → US VALIDEE |

**US REVIEW existe uniquement pour les US avec UI.** Pour les US back-end pures (pas de TDD-front, `@BDD: non applicable` ou `@BC: Infra`) : IMPACT-VERIFY → US CONTROL directement, sans pause. Alain n'a pas de compétence TypeScript pour faire une revue de code — la REVIEW n'a de valeur que quand il y a une UI à valider fonctionnellement.

Rôles détaillés : `.claude/commands/agent-us.md` · `agent-impact.md` · `agent-bdd-appli.md` · `agent-tdd-back.md` · `agent-tdd-front.md`

---

## Protocole US REVIEW

La phase REVIEW n'est pas une zone libre. Chaque retour utilisateur est classifié avant d'agir.

### Classifier le feedback

| Type | Critère | Traitement |
|------|---------|------------|
| **Nouveau comportement** | L'utilisateur veut quelque chose qui n'était pas dans les CA | Ajouter/modifier le CA dans le fichier US → mini-cycle TDD (RED→GREEN) → BDD si observable → retour REVIEW |
| **Mauvais arbitrage** | Un CA existant décrit mal ce qui est attendu | Corriger le CA dans le fichier US → adapter l'implémentation en TDD → retour REVIEW |
| **Refactoring** | Même comportement, implémentation à retravailler | Refactorer avec couverture de tests maintenue → commit → retour REVIEW |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AlainMeunierFr/job-joy](https://github.com/AlainMeunierFr/job-joy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-16 -->
