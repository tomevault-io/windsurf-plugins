---
trigger: always_on
description: **SkillsWeaver** est un moteur de jeu de rôle interactif basé sur les règles de **D&D 5e** (5ème édition). Il combine une interface web moderne (sw-web), un système d'agents autonomes (sw-dm), et un ensemble de skills CLI pour créer une expérience de jeu complète.
---

# SkillsWeaver - Moteur de Jeu de Rôle D&D 5e

## Description

**SkillsWeaver** est un moteur de jeu de rôle interactif basé sur les règles de **D&D 5e** (5ème édition). Il combine une interface web moderne (sw-web), un système d'agents autonomes (sw-dm), et un ensemble de skills CLI pour créer une expérience de jeu complète.

Le préfixe `sw-` identifie toutes les commandes CLI du projet.

## 🚨 Guidelines Claude Code

**Ces règles s'appliquent quand Claude Code modifie cette codebase. Elles sont distinctes du comportement interne des agents sw-dm.**

### Quand une Session de Jeu Live est Active

- **Ne PAS utiliser les outils MCP du navigateur** si l'utilisateur mentionne qu'il joue une session live (sw-dm ou sw-web en cours d'exécution).
- **Utiliser uniquement l'analyse de fichiers** : lire les logs (`sw-dm-session-N.log`), `agent-states.json`, et le code source. Ne pas ouvrir d'onglets navigateur.
- Si incertain qu'une session live est active, **demander avant d'utiliser tout outil navigateur**.

### Règles de Développement Go

- **Après toute modification de fichier `.go`**, exécuter `go build ./...` immédiatement. Si la compilation échoue, corriger avant toute autre action. **Ne jamais committer avec une compilation cassée**.
- **Si des fichiers `.templ` ont été modifiés**, exécuter `templ generate` d'abord, puis `go build ./...`.
- **Après ajout d'un nouveau tool sw-dm**, vérifier qu'il est accessible via le registry (checker `internal/agent/register_tools.go`) avant de déclarer terminé.

### Sécurité Git

- **Ne jamais exécuter `git add .` ou `git add -A`** : Stager uniquement les fichiers explicitement modifiés. Les hooks pre-commit gofmt et templ peuvent reformater des fichiers non liés, causant un staging accidentel depuis des sessions parallèles.
- **Ne jamais exécuter `git reset --hard` sans confirmation utilisateur**.
- Avant tout commit, exécuter `git status` pour vérifier que seuls les fichiers intentionnels sont stagés.

### Débogage du Comportement des Agents

- **Avant de suggérer un changement de prompt**, lire le fichier persona pertinent dans `core_agents/agents/` et tracer le chemin d'exécution dans `internal/agent/`. Les causes racines sont généralement architecturales (mauvaise registration d'outil, contexte non chargé, session non démarrée) — pas des problèmes de formulation.
- **Lors de modification des tools sw-dm**, suivre le processus en 5 étapes dans "Ajout de nouveaux tools pour sw-dm" ci-dessous. Ne pas sauter le CLI mapper ou l'étape de documentation dans dungeon-master.md.

---

## Architecture Globale

```
┌─────────────────────────────────────────────────────────┐
│                   UTILISATEUR                           │
└────────────┬────────────────────────────────────────────┘
             │
             ├──► sw-web (Interface Web Gin/HTMX/SSE)
             │    └──► internal/web/ + web/templates/
             │
             └──► sw-dm (REPL autonome)
                  └──► internal/agent/ (boucle d'agent complète)
                       ├──► dungeon-master (main agent, 50K tokens)
                       ├──► rules-keeper (nested, 20K tokens)
                       ├──► character-creator (nested, 20K tokens)
                       └──► world-keeper (nested, 20K tokens)
                            │
                            ▼
                  ┌─────────────────────────────────────────┐
                  │  SKILLS REGISTRY (12 skills)            │
                  │  dice-roller, character-generator,      │
                  │  adventure-manager, name-generator,     │
                  │  npc-generator, image-generator,        │
                  │  journal-illustrator, monster-manual,   │
                  │  treasure-generator, equipment-browser, │
                  │  spell-reference, map-generator         │
                  └─────────────────────────────────────────┘
                            │
                            ▼
                  ┌─────────────────────────────────────────┐
                  │  CLI BINARIES (sw-*)                    │
                  │  sw-dice, sw-character, sw-adventure,   │
                  │  sw-names, sw-npc, sw-location-names,   │
                  │  sw-image, sw-monster, sw-treasure,     │
                  │  sw-equipment, sw-spell, sw-map         │
                  └─────────────────────────────────────────┘
```

### Concepts Clés

**Skills** = Outils automatisables avec CLI
- Invoqués via `/skill-name` ou automatiquement par agents
- Exécutent des commandes `sw-*` (Go binaries)
- Retournent des données structurées JSON
- Autonomes : peuvent fonctionner seuls ou être utilisés par agents

**Agents** = Personnalités/Rôles spécialisés avec IA
- Guident l'utilisateur avec contexte narratif
- Utilisent les skills comme outils
- Maintiennent un style et ton cohérent
- Orchestrent plusieurs skills pour tâches complexes

**Agent-to-Agent Communication** :
- Le dungeon-master (main agent) peut invoquer des agents imbriqués via `invoke_agent`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nicmarti/skills-weaver](https://github.com/nicmarti/skills-weaver) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
