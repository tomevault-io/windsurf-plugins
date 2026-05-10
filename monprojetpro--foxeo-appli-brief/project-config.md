---
trigger: always_on
description: **Nom:** appli-brief-BMAD
---

# CLAUDE.md - Contexte Projet

## Projet
**Nom:** appli-brief-BMAD
**Description:** Application de génération de briefing optimisée pour la méthode BMAD

## Objectif
Résoudre le problème des briefs incomplets ou négligés qui freinent le démarrage des projets. L'application aide les chefs de projet à produire des briefs de qualité, prêts à être exploités par l'Analyst BMAD.

## Utilisateur Cible
**Persona principale : Sophie, Chef de Projet**
- Post-réunion client, bureau ou déplacement
- Frustration : Notes éparses → brief bâclé → Analyst qui pose 10 questions
- Besoin : Produire un brief "propre" rapidement + savoir s'il est complet

## Modes de Fonctionnement

### Mode Guidé
- Rédaction depuis zéro avec questions structurées pas à pas
- 4 questions séquentielles (Problème, Cible, Solution, Différenciant)
- Alternative pour les situations "page blanche"

### Mode Analyse (expérience principale)
- Import d'un document/retranscription brut
- Analyse IA et transformation en brief structuré
- Diagnostic des manques + questions de relance contextuelles
- Score de complétude en temps réel

## Référentiel de Complétude BMAD

| Élément | Requis | Description |
|---------|--------|-------------|
| Problème | Requis | La douleur en 1-2 phrases claires |
| Cible | Requis | Qui + contexte minimum |
| Solution | Requis | L'idée générale, le "quoi" |
| Différenciant | Optionnel | Pourquoi ça n'existe pas encore |

### Scoring visuel
- 🔴 **Incomplet** - Il manque Problème, Cible OU Solution
- 🟠 **À compléter** - Les 3 sont là mais trop vagues
- 🟢 **Prêt pour l'Analyst** - Les 3 éléments sont clairs et compréhensibles
- ⭐ **Optimal** - + le différenciant est renseigné

## Philosophie
> "Le brief initial doit être suffisamment clair pour que l'Analyst comprenne l'intention, suffisamment ouvert pour qu'il puisse creuser."

## Stack Technique (décidé)

| Aspect | Choix |
|--------|-------|
| **Type** | SPA (Single Page Application) |
| **Design System** | shadcn/ui + Tailwind CSS |
| **Icônes** | Lucide Icons |
| **Providers IA** | Claude API, OpenAI GPT, Gemini API |
| **Stockage** | localStorage (briefs) |

## Direction UX (décidée)

**Direction Dashboard** - Interface de type tableau de bord avec :
- Header avec navigation principale
- Zone de statistiques (briefs créés, prêts, à compléter, incomplets)
- Panneau de brief détaillé avec les 4 éléments colorés
- Brief Library intégrée

## Structure du Projet

```
appli-brief-BMAD/
├── app/                # Next.js App Router
│   ├── globals.css     # Tailwind + scoring colors
│   ├── layout.tsx
│   └── page.tsx
├── components/
│   ├── ui/             # shadcn/ui (DO NOT MODIFY)
│   ├── brief/          # Brief feature components
│   ├── dashboard/      # Dashboard components
│   └── layout/         # Layout components
├── lib/
│   ├── providers/      # AI provider abstraction
│   ├── scoring/        # Scoring logic
│   ├── storage/        # localStorage helpers
│   ├── export/         # Export utilities
│   └── utils/          # General utilities
├── types/              # TypeScript types
├── contexts/           # React Contexts
├── hooks/              # Custom React hooks
├── _bmad/              # Configuration BMAD Method
├── _bmad-output/       # Artefacts de planification
│   ├── analysis/       # Sessions de brainstorming
│   ├── project-planning-artifacts/  # Product brief, epics
│   │   ├── epics.md    # Epics & Stories (6 epics, 24 stories)
│   │   └── implementation-readiness-report-2026-01-11.md
│   ├── implementation-artifacts/    # Stories et sprint tracking
│   │   ├── sprint-status.yaml       # Suivi du sprint
│   │   └── 1-1-project-initialization.md  # Story 1.1
│   ├── prd.md          # Product Requirements Document
│   ├── architecture.md # Architecture technique
│   ├── ux-design-specification.md   # Spécifications UX
│   ├── ux-design-directions.html    # Mockups visuels
│   └── bmm-workflow-status.yaml     # Suivi workflow BMAD
└── CLAUDE.md           # Ce fichier
```

## Conventions

- **Langue:** Français pour la documentation et les interfaces
- **Méthode:** BMAD Method (Business-Minded Agile Development)

## Statut du Projet

**Phase actuelle:** ✅ Implementation Complete - Tous les Epics terminés

| Phase | Statut | Artefact |
|-------|--------|----------|
| Brainstorming | ✅ Terminé | `brainstorming-session-2026-01-07.md` |
| Product Brief | ✅ Terminé | `product-brief-appli-brief-BMAD-2026-01-07.md` |
| PRD | ✅ Terminé | `prd.md` |
| UX Design | ✅ Terminé | `ux-design-specification.md` |
| Architecture | ✅ Terminé | `architecture.md` |
| Epics & Stories | ✅ Terminé | `epics.md` (6 epics, 24 stories) |
| Implementation Readiness | ✅ Terminé | `implementation-readiness-report-2026-01-11.md` |
| Sprint Planning | ✅ Terminé | `sprint-status.yaml` |
| **Epic 1** | ✅ Done | 4/4 stories terminées |
| **Epic 2** | ✅ Done | 5/5 stories terminées |
| **Epic 3** | ✅ Done | 5/5 stories terminées |
| **Epic 4** | ✅ Done | 5/5 stories terminées |
| **Epic 5** | ✅ Done | 5/5 stories terminées |
| **Epic 6** | ✅ Done | 5/5 stories terminées |

## Projet Terminé
Toutes les fonctionnalités MVP ont été implémentées.

### Epic 3: AI Core & Analysis Engine (COMPLETED)

#### Story 3.1: AI Provider Abstraction Layer (COMPLETED)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [MonprojetPro/foxeo-appli-brief](https://github.com/MonprojetPro/foxeo-appli-brief) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
