---
trigger: always_on
description: Guidance for Claude Code working with the CoursIA repository.
---

# CLAUDE.md

Guidance for Claude Code working with the CoursIA repository.

---

## ⚠️ RÈGLE D'OR - Coordination Inter-Machines

**TOUTE coordination entre machines (po-2023, po-2026, etc.) DOIT se faire via RooSync**

### ❌ JAMAIS:
- Commit de rapports de coordination sur GitHub
- Messages via SSH
- Fichiers `*_TEST_REPORT.md`, `*_COORDINATION.md` dans le repo
- Tout fichier de coordination hors RooSync

### ✅ TOUJOURS:
- Thread RooSync pour toute coordination inter-machine
- Les messages RooSync sont persistants et suivis
- GitHub = code uniquement, pas de coordination

**VIOLATION DE CETTE RÈGLE = ERREUR CRITIQUE**

---

## RÈGLE - RooSync Dashboard : toujours lire le contenu complet

Le **dashboard RooSync workspace CoursIA** (`roosync_dashboard` type=workspace, workspace=CoursIA) est le canal central de coordination cross-machine. Il contient le status du coordinateur (ai-01), les missions assignees, les resultats backtests, les decisions de consolidation, et l'intercom entre agents.

### Obligations lors d'un tour de coordination :
1. **Lire le contenu complet** du dashboard — l'API retourne parfois un JSON tronque en preview ("Output too large"). Dans ce cas, TOUJOURS utiliser `Read` sur le fichier persiste pour lire l'integralite.
2. **Verifier l'inbox** RooSync pour les messages directs non lus.
3. **Verifier le heartbeat** cluster pour savoir quelles machines sont actives.
4. **Si aucune mission assignee** : envoyer un message RooSync a ai-01 (coordinateur) pour demander des instructions. Ne pas attendre passivement.

### Ne JAMAIS :
- Resumer le dashboard a partir du seul preview tronque
- Omettre la lecture de l'intercom (contient l'historique des decisions)
- Se considerer "disponible" sans prevenir le coordinateur

---

## 🚨 RÈGLE CRITIQUE - Git Force Push INTERDIT

**INCIDENT 2026-03-13** : Force push accidentel sur main a potentiellement écrasé des commits

### ❌ JAMAIS:
- `git push --force` ou `--force-with-lease` sur main
- `git reset --hard` sur main sans validation
- Modifier l'historique public après publication

### ✅ TOUJOURS:
- Pour une PR : créer feature branch FROM main, ne pas reset main
- Utiliser cherry-pick, revert, ou nouveaux commits
- Demander validation explicite du user pour toute opération destructive
- En cas d'urgence extrême : user doit valider AVANT le force push

**Exception** : Uniquement urgence confirmée avec validation préalable du user

**Voir aussi** : `.claude/rules/git-workflow.md` pour les règles git complètes

**VIOLATION DE CETTE RÈGLE = ERREUR CRITIQUE**

---

## Regles Agents (Roo Code / machines distantes)

Les agents Roo sur les machines po-2023, po-2024, po-2025, po-2026 travaillent sur ce depot via RooSync. Ces regles sont **OBLIGATOIRES** pour tout agent.

### Git : PRs obligatoires

- **JAMAIS** de push direct sur `main`. Creer une feature branch, pousser, creer une PR
- Nommage branche : `feature/<sujet>` ou `fix/<sujet>`
- Un seul sujet par PR (pas de mega-PR multi-issues)
- Le coordinateur (ai-01) review et merge. Les agents ne mergent pas eux-memes

### Qualite : code avant documentation

- **Priorite** : code fonctionnel > tests/validation > documentation
- Ne pas generer de markdown (README, MAPPING, RAPPORT) sans code fonctionnel associe
- Ne pas creer de fichiers de planification (EXTEND_*.md, PROCEDURE_*.md) dans le repo — utiliser RooSync
- Les rapports d'audit, inventaires, et status vont sur le **dashboard RooSync**, pas dans le repo

### Review PR : validation explicite des objectifs

Toute PR doit faire l'objet d'une **review complete** avant merge :

1. **Relire l'issue ou le contexte d'origine** : identifier les objectifs precis (layout, positionnement, contenu, format)
2. **Valider chaque objectif explicitement** : verifier dans le code ET visuellement que chaque exigence est satisfaite
3. **Verification visuelle obligatoire pour les slides** :
   - Lancer le serveur Slidev et verifier CHAQUE slide modifie (pas un echantillon)
   - Verifier avec `?clicks=99` pour voir le contenu complet revele
   - Verifier l'absence d'overflow (contenu coupe en bas)
   - Verifier les layouts d'images (overlay uniquement, jamais colonne droite)
4. **Verification visuelle obligatoire pour les notebooks** : executer ou au moins valider la structure
5. **Ne jamais merger une PR sur la seule base du CI green** : CI valide la syntaxe, pas le rendu visuel ni la conformite aux specifications

**Slides : images en overlay uniquement**
- Les images de fond doivent utiliser `layout: image-overlay` avec le texte par-dessus, jamais en colonne droite
- Cette regle a ete specifiee dans l'issue #221 et confirmee 5+ fois dans les discussions
- Verifier que chaque image est bien positionnee sur le bon slide

**VIOLATION = PR a rejeter, meme si le code compile**

### Pas de duplication

- Avant de creer un fichier (README, docs, shared library), verifier qu'il n'existe pas deja
- Utiliser `grep` et `find` pour chercher les doublons
- Si un fichier similaire existe, le mettre a jour plutot qu'en creer un nouveau

### Enrichissement notebooks : regles strictes

- Chaque cellule de transition doit avoir du **contenu pedagogique specifique** (pas de "Suite du traitement" generique)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/jsboige) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
