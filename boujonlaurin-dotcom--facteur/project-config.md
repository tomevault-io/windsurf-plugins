---
trigger: always_on
description: > App mobile digest quotidien (5 articles, "moment de fermeture"). Flutter + FastAPI + PostgreSQL (Supabase) + Railway.
---

# CLAUDE.md — Facteur

> App mobile digest quotidien (5 articles, "moment de fermeture"). Flutter + FastAPI + PostgreSQL (Supabase) + Railway.

---

## BRANCHES & ENVIRONNEMENTS — RÈGLE ABSOLUE

> **`main` = environnement STAGING CONTINU** (backend `api-staging-40d3`, APK flavor
> `staging` = `com.example.facteur.staging`, canal `beta`). C'est l'env que TU testes.
> **Toute PR DOIT cibler `main` avec `--base main`** — le workflow quotidien est inchangé.
>
> **`production` = branche HEBDO** (backend `facteur-production`, APK flavor `prod`,
> canal `stable` → vrais users). Elle n'est **JAMAIS** une cible de PR et ne reçoit
> **jamais** de commit direct : elle est avancée **uniquement** par le bouton manuel
> GitHub Actions **« Weekly Production Release »** (`weekly-release.yml`).
>
> **`staging` (l'ancienne branche) est abandonnée** — ne plus l'utiliser.
> Un hook (`pre-bash-no-staging.sh`) bloque tout `gh pr create` sans `--base main`.

---

## Contraintes Techniques (LOCKED)

- Python **3.12.x** uniquement (3.13+ casse pydantic)
- `list[]`, `dict[]`, `X | None` natifs (jamais `from typing import List, Dict, Optional`)
- JWT secret identique mobile ↔ backend
- **Alembic est la seule source de vérité pour le schéma DB.** Tout DDL (ALTER, CREATE, DROP) passe par une migration générée via `alembic revision --autogenerate -m "<desc>"` dans la PR qui le requiert. **Pas de SQL manuel via Supabase SQL Editor** — c'est le pattern qui a causé l'incident de drift d'avril 2026 (cf. [runbook de récupération](docs/runbooks/recover-from-alembic-drift.md)). Exactement 1 head Alembic. Le `Dockerfile` exécute `alembic upgrade head` au démarrage de chaque conteneur Railway — donc une migration cassée plante le déploiement, et stamper prod *avant* de merger une refonte de la chaîne est obligatoire (cf. runbook).
- **Migrations additives / expand-contract OBLIGATOIRE.** `main` (staging) et `production` (prod) **partagent la DB Supabase de prod**. Le `Dockerfile` joue `alembic upgrade head` au boot des **deux** services : une migration mergée dans `main` touche donc la DB partagée **dès le boot staging**, pendant que le backend prod (ancien code `production`) tourne encore dessus jusqu'à 1 semaine. ⇒ jamais de `DROP`/rename/`NOT NULL`-sur-peuplé en une étape ; étaler sur 2 cycles hebdo (semaine N ajoute, semaine N+1 retire). Migrations idempotentes (no-op si déjà à head), gardées sur les deux backends.
- **UI/Design → design-system-first** : repartir des tokens/composants existants ([Front-end Spec §5-6](docs/front-end-spec.md) ; `apps/mobile/lib/config/theme.dart`) ; minimalisme et respect du layout (budgets snap/fit `section_fit.dart`) comme premières boussoles **avant toute nouvelle UI**.
- Zones à risque (Auth, Router, DB, Infra) → lire [Safety Guardrails](docs/agent-brain/safety-guardrails.md) AVANT modif

---

## Workflow : PLAN → CODE+TEST → PR

### 1. PLAN (confirmation requise)

1. Classifie la tâche : **Feature** / **Bug** / **Maintenance**
2. Lis les docs nécessaires via la [Navigation Matrix](docs/agent-brain/navigation-matrix.md)
3. Crée la documentation :
   - Feature → `docs/stories/core/{epic}.{story}.{nom}.md`
   - Bug → `docs/bugs/bug-{nom}.md`
   - Maintenance → `docs/maintenance/maintenance-{nom}.md`
4. Rédige le plan technique dans la Story/Bug doc
5. **STOP → Présente le plan à l'utilisateur → Attends GO**

### 2. CODE + TEST (autonome)

Après le GO utilisateur, implémente et teste en autonomie :

1. **Code** : implémente atomiquement, MAJ story (tasks ✓, fichiers modifiés)
2. **Tests unitaires** : les hooks `post-edit-auto-test.sh` lancent automatiquement les tests liés à chaque fichier modifié. Corrige les échecs immédiatement.
3. **Tests E2E / UI** : utilise le **Playwright Agent CLI** (`playwright-cli`, skills `facteur-qa-web` + `playwright-cli`) pour tester les flux visuels sur le build web :
   - Démarre l'API locale si besoin (`uvicorn app.main:app --port 8080`)
   - Navigue dans l'app, remplit les formulaires, vérifie les réponses
   - Valide les cas nominaux + cas limites
   - ⚠️ Flutter web = canvas : active la sémantique au boot (cf. skill `facteur-qa-web`) avant tout `snapshot`
4. **Suite complète** : lance la suite de tests complète (`pytest -v` backend, `flutter test` mobile) et corrige tout échec
5. Le hook `stop-verify-tests.sh` vérifie automatiquement que les tests passent avant de terminer — si un test échoue, l'agent doit corriger avant de pouvoir conclure

> **Raccourci recommandé** : une fois le code écrit, lance **[`/go`](.claude/commands/go.md)** pour enchaîner automatiquement VERIFY (tests + Playwright + scripts QA) → SIMPLIFY (skill `simplify`) → PR vers `main`. Voir [/go — chaîne verify→simplify→PR](#go--chaîne-verifysimplifypr) plus bas.

### 3. PR (confirmation requise)

1. Crée la PR vers `main` — **OBLIGATOIRE : `--base main`** (env staging continu ; le hook `pre-bash-no-staging.sh` bloquera toute PR sans `--base main`). **Ne jamais cibler `production`** (avancée seulement par le bouton hebdo).
2. **STOP → Notifie "PR #XX prête pour review"**
3. Attends CI green + Peer Review APPROVED avant merge


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [boujonlaurin-dotcom/facteur](https://github.com/boujonlaurin-dotcom/facteur) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-28 -->
