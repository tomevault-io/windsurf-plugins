---
trigger: always_on
description: This file is the source of truth for AI agent instructions across all supported LLM providers.
---

<!--
This file is the source of truth for AI agent instructions across all supported LLM providers.
CLAUDE.md (Claude Code), GEMINI.md (Gemini CLI) et .github/copilot-instructions.md (GitHub Copilot)
sont des symlinks vers AGENTS.md. Le fichier .cursor/rules/project.mdc l'importe via `@AGENTS.md`.
Les skills vivent dans .agents/skills/<name>/SKILL.md au format agentskills.io — convention
cross-client pour interoperabilite multi-outil (Claude Code via symlink .claude/skills/,
Cursor, Codex, Copilot coding agent, Gemini CLI, Junie).
Voir SKILLS.md à la racine pour l'index humain des skills.
-->

# Redface 2

## Git et GitHub

- **Organisation GitHub** : ForumHFR
- Utiliser la config git locale (repo), jamais `--global`

## Projet

- Phase actuelle : **Phase 4 — UI & hygiène + Extensions** ([roadmap](docs/specs/roadmap.md)). Phases 0 à 3 livrées (bootstrap ; lecture du forum ; écriture poster/citer/upload ; messages MP + DT/MultiMP), bêta **0.16.0** publiée (Play open testing + F-Droid). En cours : refontes UI (vue Drapeaux #603, vue Topic #604), aide & réglages, architecture d'extensions (#6 MPStorage, #7).
- Licence : GPL-3.0-only
- Documentation : GitHub Pages via `docs/` (Jekyll + just-the-docs)
- Langue : code en anglais, issues et docs en francais

## Setup

```bash
# Build applicatif local — image Docker pin (cf. .devcontainer/devcontainer.json)
# Variantes dev/prod (#233) : le :app:assembleDebug non flavoré ne résout plus.
./gradlew :app:assembleProdDebug

# Preview Jekyll (necessite Ruby + Bundler)
cd docs && bundle install && bundle exec jekyll serve
```

## Structure des specs

```
docs/
  index.md           # Accueil du site
  specs/             # Pages canoniques qui font foi
    methodology.md   # Methode canonique : spec/prototype/TDD
    scope.md         # Scope produit et use cases
    stack.md         # Choix techniques justifies
    architecture.md  # Modules Gradle, couches, data flow, securite, erreurs
    navigation.md    # Ecrans, deep linking, back stack
    models.md        # Data classes Kotlin (source de verite pour les types)
    mvi.md           # Pattern MVI, exemples ViewModel/Screen/State
    protocol-hfr.md  # Contrats externes, endpoints, edge cases
    roadmap.md       # Phases de developpement
    extensions.md    # Extensions communautaires et architecture d'extensions
  adr/               # Architecture Decision Records (depuis v0.5.1)
  guides/            # Pages d'accompagnement
    contributing.md  # Conventions, tests, accessibilite, localisation
    rationale.md     # Pourquoi la reecriture
    naming.md        # Candidats pour le nom de l'app
  _config.yml        # Config Jekyll (version des specs dans le footer)
```

## Stack (verrouillee)

Kotlin, Jetpack Compose, MVI, Compose Navigation 3, Hilt (KSP), OkHttp 5, Jsoup, Room, Coil, Coroutines + Flow, minSdk 29

## Tests

Tests bootstrappes en Phase 0 et consommes des Phase 1 :
- **Konsist** (`app/src/test/.../ArchitectureKonsistTest.kt`) — frontières architecture, scope non vide.
- **JUnit 4** + **Turbine** sur le parser (`core/parser/src/test/.../PostContentParserTest.kt`, `TopicPageParserTest.kt`) et les ViewModel slice (`feature/topic/src/test/.../TopicViewModelTest.kt`).
- **MockK** et **Robolectric** câblés dans `gradle/libs.versions.toml` mais pas encore consommés (arrivent avec les ViewModels et écrans réels Phase 1+).

Strategie complete (TDD/spec/prototype par sous-chantier, fixtures, couverture differenciee) definie dans `docs/specs/methodology.md` et `docs/guides/contributing.md` :
- Couverture **hybride differenciee** : 100% sur les transformers du parser HFR (fixtures dictent l'exhaustivite), guidee par risque ailleurs (ViewModels, mappers, repositories). **Pas d'objectif 100% global.**
- Fixtures HTML capturees depuis HFR reel via `hfr-mcp`, jamais fabriquees a la main.

## Conventions

- Issues et commentaires : toujours mentionner qui a demande l'action si generee par IA
- Conventional Commits : `feat:`, `fix:`, `docs:`, `chore:`, `test:`
- Branche principale : `main` (branche de **release**). **Branche d'intégration : `dev`** — les branches de feature partent de `origin/dev` (fraîchement pull) et y sont mergées ; promotion `dev → main` à la release. Ne jamais brancher depuis une branche feature locale non squashée.
- **Pas de push direct sur `main`** : toute modification (code, doc, skill, ADR, fixture) passe par une **pull request** avec CI verte avant merge. Cette règle vaut aussi pour les changements jugés triviaux (typo, ajout d'une ligne dans une table) : la PR donne un point d'historique reviewable et fait tourner la CI. La branch protection actuelle n'enforce pas techniquement cette règle (`enforce_admins: false` pour garder un échappatoire d'urgence), c'est une discipline de projet. Toute exception (revert critique, hotfix bloquant, ou absence prolongée de reviewer humain) doit être : (1) justifiée dans le message du commit ou du merge, et (2) tracée dans une issue post-mortem ouverte le jour même si l'exception devient récurrente.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ForumHFR/redface2](https://github.com/ForumHFR/redface2) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
