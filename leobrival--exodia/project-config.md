---
trigger: always_on
description: - Alwaysin plan mode to make a plan
---

# Exodia - Outil moderne pour appels à projets

## Plan & Review & TDD

### Before starting work

- Alwaysin plan mode to make a plan
- After get the plan, make sure you Write the plan to .claude/tasks/TASK_NAME.md
- Use checkboxes to mark tasks as completed and incomplete and headings to structure the plan.

- The plan should be a detailed implementation plan and the reasoning behind them, as well as tasks broken down.
- If the task require external knowledge or
  certain package, also research to get latest knowledge (Use Task tool for research)
- Don't over plan it, always think MVP.
- Once you write the plan, firstly ask me to
  review it. Do not continue until I approve the plan.
- If the plan is approved, you can start implementing.

### While implementing

- You shoul update the plan as your work.
- After you complete tasks in the plan, you should update and append detailed descriptions of the changes you made, so following tasks can be easily hand over to other engineers.

### Before implementing

- You shall not write code until you have written a failing test.
- You shall only write one new failing unit test at a time.
- You shall not write more code than is necessary to make the failing test pass.

The approach to developing in TDD is as follows:

- Write a test that fails because the relevant code does not exist
- Write just enough code to validate the new test and the previous ones
- Optimize the code without altering its behavior

### Test Coverage

- Use pnpm test:coverage to run tests with coverage
- Use pnpm test:ui to run tests with ui

## Project Overview

Create a modern, innovative, and highly intuitive tool for designing and automating call for projects responses with advanced AI features and a modern design inspired by Google Notebook, ChatGPT Canvas.

- Centralization of necessary information and documents
- Automation of repetitive tasks (document generation, tracking, notifications)
- Simple and efficient user interface
- Modular architecture to facilitate scalability
- Security and compliance (authentication, access management)

The goal is to transform call for projects management into a seamless, fast, and collaborative process, leveraging tech and UX best practices.

## Architecture & Technology Stack [`.claude/docs/architecture.md`](.claude/docs/architecture.md)

## Development Commands [`package.json`](package.json)

## Core Features

### 1. Gestion de projet

_See [`.claude/docs/user-management.md`](.claude/docs/user-management.md)_

- CRUD des organisations
- Gestion des membres
- CRUD des projets
- Onboarding des projets

### 2. Automatisation & vectorisation des documents (RAG)

_See [`.claude/docs/rag-system.md`](.claude/docs/rag-system.md) & [`.claude/docs/document-processing.md`](.claude/docs/document-processing.md)_

- Upload de documents
- Vectorisation des documents
- Recherche de documents
- Suppression de documents
- Génération automatisée de documents

### 3. Collaboration & gestion des accès

_See [`.claude/docs/collaboration.md`](.claude/docs/collaboration.md)_

- Accès basé sur les rôles
- Cloisonnement sécurisé des données par projet
- Gestion des documents partagés
- Journaux d'audit

### 4. Notifications & suivi

_See `.claude/docs/collaboration.md`_

- Notifications d'assignation de tâches
- Mises à jour de statut de projet
- Rappels d'échéances
- Fil d'activité

### 5. Sécurité & conformité

_See [`.claude/docs/auth.md`](.claude/docs/auth.md) & [`.claude/docs/database.md`](.claude/docs/database.md)_

- Authentification
- Application du contrôle d'accès
- Chiffrement des données
- Reporting conformité

## User Map

### Epics by Theme

#### Gestion de projet

- **CRUD des organisations** : Créer, lire, modifier et supprimer des organisations
- **Gestion des membres** : Inviter, gérer les rôles et permissions des membres [`.claude/docs/invitation-system.md`](.claude/docs/invitation-system.md)
- **CRUD des projets** : Créer, organiser et gérer les projets d'appels à offres
- **Onboarding des projets** : Guide d'aide pour démarrer efficacement un nouveau projet [`.claude/docs/user-management.md`](.claude/docs/user-management.md)

#### Automatisation & vectorisation des documents

- **Upload de documents** : Interface de glisser-déposer pour les fichiers PDF, DOCX, TXT [`.claude/docs/document-processing.md`](.claude/docs/document-processing.md)
- **Vectorisation des documents** : Traitement automatique et conversion en embeddings [`.claude/docs/document-processing.md`](.claude/docs/document-processing.md)
- **Recherche de documents** : Recherche sémantique et par mot-clé dans la base documentaire [`.claude/docs/rag-system.md`](.claude/docs/rag-system.md)
- **Suppression de documents** : Gestion du cycle de vie et suppression sécurisée [`.claude/docs/document-processing.md`](.claude/docs/document-processing.md)
- **Génération automatisée de documents** : IA pour créer des réponses personnalisées [`.claude/docs/project-generation.md`](.claude/docs/project-generation.md)

#### Collaboration & gestion des accès

- **Accès basé sur les rôles** : Système de permissions granulaires (admin, member) [`.claude/docs/user-management.md`](.claude/docs/user-management.md)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/leobrival) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
