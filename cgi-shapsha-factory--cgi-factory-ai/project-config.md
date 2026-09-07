---
trigger: always_on
description: always answer in summary in less than 10 lines
---

# CLAUDE.md

always answer in summary in less than 10 lines

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Nature du dépôt
Ce dépôt n'est **pas un projet applicatif** : c'est une **collection de plugins Claude Code** formant une "Factory IA" (fabrique logicielle à forte assistance IA). Les livrables sont des **skills Markdown** + un `plugin.json` par plugin. **Ni build, ni lint, ni tests** - la seule "compilation" est la validité JSON des manifestes.

- Dépôt : `github.com/NASSWIEL/Factory-IA` (**privé depuis 2026-06-26**), branche `main`. Documents de conception dans le parent (`../context-complet.txt`, `../architecture.md`).
- Distribution : marketplace **`git-subdir`** (`.claude-plugin/marketplace.json`) -> **Shapsha-Factory**. Le repo pro CGI (`CGI-Shapsha-Factory/CGI-Factory-AI`) reçoit les évolutions par **fast-forward direct sur `cgi/main`** (commit -> branche `optimisation` -> `git push cgi optimisation` -> `git push cgi optimisation:main`, jamais de force-push). **Poussé par le compte `NASSWIEL`** (owner de l'org ; l'ancien compte `asswiel` a été retiré de l'org le 2026-07-06). Détails (remotes, token, identité) en **mémoire projet interne**, pas dans l'arbre versionné. Côté utilisateur : `/plugin marketplace update Shapsha-Factory`.
- Installation : **uniquement via la marketplace** (`.claude-plugin/marketplace.json` -> Shapsha-Factory). Côté utilisateur : `/plugin marketplace add` puis `/plugin install <plugin>@Shapsha-Factory`. La marketplace expose aussi `category`/`tags` par plugin (groupement par rôle dans `/plugin` Discover). *(L'ancien installeur façon BMAD - `install.py` / wrapper npx / `INSTALL.md` - a été retiré : la marketplace suffit.)*
- Git : branche `main`, email `naifsaleem20@gmail.com`, compte GitHub `NASSWIEL` (origin **et** cgi). Commit/push uniquement sur demande.

## Les plugins (4 contrats amont + 2 aval + 1 transversal)
| Dossier | Rôle | État | Détail |
|---------|------|------|--------|
| `cadrage/` | Contrat **fonctionnel** (captation -> pack repris par l'architecte) | **construit** | voir `cadrage/CLAUDE.md` |
| `architecte/` | Contrat **technique** (drivers, attributs qualité, composants, stack, ADR, walking skeleton, conventions/linters, diagrammes, **enforcement tests + formatage** ; protection de branche gérée côté GitHub, pas de hook local) | **construit** | voir `architecte/CLAUDE.md` |
| `designer/` | Contrat **design** - **atelier de couverture** (checklist fondation/expérience/technique pré-remplie par les handoffs) qui produit le **prompt Claude Design** + rapport de couverture + handoff ; le design system naît dans **Claude Design**, son export est committé dans `designer-out/maquette-de-claude-design/`, pas généré par le plugin | **construit** | voir `designer/CLAUDE.md` |
| `assembleur/` | Convergence des 3 contrats par feature + **tickets Linear (`premier-alimente-linear` : 1 ticket `Feature`/feature + 1 sous-ticket `Task`/Functional Requirement, tout en Backlog, label `Feature` seul ; `creation-taches-par-phase-de-spec` : 1 sous-ticket `Task`/phase de `tasks.md` après `/speckit.tasks` ; MAJ d'état via `update-issue-linear` ; via MCP linear-prism)** + **`create-cowork-md`** (contexte de supervision `init-cowork.md` à la racine pour le PO/Quark : liens GitHub + Linear) + amorçage repo SpecKit (constitution, CLAUDE.md, briefs 3-faces, glossaire consolidé, MEMORY.md, seeds spec.md) ; **n'écrit pas dans le repo cible** (4 exceptions bornées : Linear externe, `specify init`, `init-cowork.md` à la racine, et `CLAUDE.md` + `memory/` déployés dans le `.claude/` du projet) | **construit** | voir `assembleur/CLAUDE.md` |
| `validation/` | Phase **aval** (recette fonctionnelle d'une feature livrée, **avant** le traitement des écarts) - **plan de test dérivé de la spec** (`specs/<feature>/spec.md` : 1 cas `TC-<feature>-NNN` par critère d'acceptation, tracé, critère non testable = `A CLARIFIER`, jamais interprété) + **exécution navigateur** contre l'environnement de recette (extension Chrome "Claude in Chrome" prioritaire, MCP Playwright en repli, ou mission différée `mission-cowork.md` pour Claude Cowork - contrat de résultats commun, **déroulé effectif** de chaque cas en langage naturel) + **rapport de recette tracé** exigence par exigence, livré en **PDF présentable** (`rapport-de-validation.pdf`), avec **porte de recette humaine** (verdict du testeur dans le rapport committé + commentaire Linear, jamais dans le manifeste). **Détecte et trace ; le traitement passe par `maintenance/`** (renvois vers `creation-anomalie`/`creation-evolution`, contenu pré-rempli, jamais de création directe) | **construit** | voir `validation/CLAUDE.md` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [CGI-Shapsha-Factory/CGI-Factory-AI](https://github.com/CGI-Shapsha-Factory/CGI-Factory-AI) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-07 -->
