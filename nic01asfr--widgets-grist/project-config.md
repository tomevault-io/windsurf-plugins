---
trigger: always_on
description: Guide de développement pour le repository de widgets Grist.
---

# CLAUDE.md

Guide de développement pour le repository de widgets Grist.

---

## Instructions pour agents IA

**Ce repo suit une séparation stricte développement / production.**

### Règles essentielles

1. **Ne jamais modifier `published/`** sauf demande explicite de publication
2. **Développer dans `projects/`** — tous les projets sont dans ce dossier
3. **Chaque projet a son propre CLAUDE.md** — le lire avant toute intervention
4. **Le manifest.json est auto-généré** — ne jamais l'éditer manuellement
5. **Consulter `skills/`** — patterns de code réutilisables pour Grist

### Checklist avant de coder

```
□ Lire le CLAUDE.md du projet concerné
□ Consulter skills/ pour les patterns standards
□ Comprendre l'architecture existante
□ Identifier les fonctions/patterns déjà présents
□ Ne pas dupliquer ce qui existe
```

### Workflow de travail

```
DÉVELOPPEMENT                         PUBLICATION
─────────────────────────────────────────────────────────
projects/mon-widget/     ──promote──►  published/mon-widget/
    ├── fichiers.html                      ├── package.json (obligatoire)
    └── CLAUDE.md                          └── index.html
```

### Avant de coder sur un projet

1. Lire le `CLAUDE.md` du projet (ex: `projects/tasks_app/CLAUDE.md`)
2. Comprendre l'architecture existante
3. Ne pas publier sans demande explicite

### Quand l'utilisateur demande de "publier"

1. Créer `published/nom-widget/package.json` avec la section `grist`
2. Copier les fichiers finaux vers `published/nom-widget/`
3. Exécuter `npm run manifest` pour régénérer le catalogue
4. Commit avec message descriptif

### Conventions de code

- **Widgets statiques** : HTML autonome avec `<script src="grist-plugin-api.js">`
- **Français** pour les commentaires et messages utilisateur
- **Pas de frameworks** sauf si le projet le spécifie
- **grist.ready()** obligatoire avec `requiredAccess` approprié

---

## Vue d'ensemble

Ce repository contient des widgets personnalisés pour [Grist](https://www.getgrist.com/). Il est structuré pour supporter :
- Le **développement** de widgets (zone de travail)
- La **publication** de widgets stables (zone déployée sur GitHub Pages)
- Les deux types de widgets Grist : **statiques** (HTML pur) et **build** (npm/React)

## Structure du repository

```
Widgets-Grist/
├── .github/
│   └── workflows/
│       └── publish.yml           # CI/CD : build + deploy sur GitHub Pages
│
├── .nojekyll                     # Désactive Jekyll sur GitHub Pages
├── .gitignore
├── package.json                  # Config npm workspaces
├── CLAUDE.md                     # Ce fichier
├── README.md                     # Documentation publique
│
├── projects/                     # ZONE DE DÉVELOPPEMENT
│   ├── tasks_app/               # TaskFlow (kanban, gantt, calendar)
│   │   ├── CLAUDE.md
│   │   ├── kanban.html
│   │   ├── gantt.html
│   │   ├── calendar.html
│   │   └── ...
│   │
│   └── widget_app/              # Artefactory (IDE no-code)
│       ├── CLAUDE.md
│       ├── app.html
│       ├── app_runtime.html
│       └── templates/
│
├── published/                    # ZONE PUBLIÉE (déployée sur gh-pages)
│   ├── manifest.json            # Catalogue des widgets (auto-généré)
│   │
│   ├── taskflow/                # Widgets TaskFlow publiés
│   │   ├── package.json
│   │   ├── kanban/
│   │   │   └── index.html
│   │   ├── gantt/
│   │   │   └── index.html
│   │   └── calendar/
│   │       └── index.html
│   │
│   ├── artefactory/             # Widgets Artefactory publiés
│   │   ├── package.json
│   │   ├── admin/
│   │   │   └── index.html
│   │   ├── runtime/
│   │   │   └── index.html
│   │   ├── registry.json
│   │   └── components/
│   │       └── ...
│   │
│   └── [autres-widgets]/
│
├── packages/                     # WIDGETS AVEC BUILD (optionnel)
│   └── [widget-react]/
│       ├── package.json
│       ├── src/
│       └── dist/                # Output → copié dans published/
│
├── skills/                       # PATTERNS DE CODE RÉUTILISABLES
│   ├── README.md                # Index des skills
│   ├── schema.md                # ⭐ Création schéma (tables, colonnes, refs, labels)
│   ├── grist-api.md             # API Grist CRUD
│   ├── data-conversion.md       # Conversion colonaire, dates, RefList
│   ├── inter-widget.md          # Communication entre widgets
│   ├── bridge.md                # GristBridge pour iframes
│   └── patterns.md              # Modales, filtres, UI patterns
│
└── scripts/
    ├── generate-manifest.js     # Génère manifest.json depuis published/
    └── promote.js               # Copie de projects/ vers published/
```

## Zones du repository

### `projects/` — Développement

Zone de travail pour les widgets en cours de développement. **Non déployée** sur GitHub Pages.

- Chaque projet a son propre `CLAUDE.md` avec les spécificités
- Les fichiers peuvent être testés localement (mode démo) ou via URL raw GitHub
- Pas de contrainte de structure stricte

### `published/` — Production

Zone des widgets stables publiés. **Déployée sur GitHub Pages** via CI/CD.

- Chaque widget a un `package.json` avec la section `grist` (métadonnées)
- Structure requise : `widget-name/index.html` (ou `widget-name.html`)
- Le `manifest.json` est auto-généré par le script


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nic01asFr/Widgets-Grist](https://github.com/nic01asFr/Widgets-Grist) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
