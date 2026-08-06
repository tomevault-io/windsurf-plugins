---
trigger: always_on
description: Derniere mise a jour : 2026-07-26
---

# AGENTS.md - Guide de travail Leopardo RH

Derniere mise a jour : 2026-07-26

Ce fichier doit etre lu au debut de chaque nouvelle session agent. Il doit aussi etre mis a jour a chaque push ou merge vers `main`, comme le `CHANGELOG.md`, des qu'une lecon operationnelle peut eviter de perdre du temps plus tard.

> **NOUVEL AGENT ? Commence par lire `dev-hub/prompts/00_AGENT_QUICK_CARD.md` (2 min) pour une carte de reference rapide. Ce fichier AGENTS.md est le guide complet.**

## Regles obligatoires

- **REGLE D'OR POUR LES NOUVEAUX MODULES** : Avant de commencer a coder un nouveau module ou de generer des tickets (GitHub Issues) pour celui-ci, un agent DOIT OBLIGATOIREMENT creer un fichier Markdown de specification dans le dossier `docs/specifications/` (ex: `docs/specifications/MODULE_RECRUTEMENT.md`). Ce n'est qu'apres validation explicite de ce document par le proprietaire que les issues GitHub peuvent etre creees.

- Avant de travailler sur une branche existante, faire `git fetch origin main` puis comparer avec `origin/main`.
- `main` distant est la source de verite. Le local doit rester aligne sur `origin/main` apres chaque intervention terminee.
- Ne pas pousser directement sur `main` si la branche est protegee. Creer un PR, attendre les checks GitHub Actions, puis merger et supprimer la branche.
- Apres un merge dans `main`, supprimer la branche distante et nettoyer les branches locales devenues inutiles.
- Ne jamais perdre les stashes existants. Verifier `git stash list` avant toute operation destructive.
- Chaque changement de comportement, migration, CI ou procedure doit avoir une entree `CHANGELOG.md`.
- Chaque connaissance utile pour les prochains agents doit etre ajoutee ici.

## 🗺️ Cartographie de l'Ecosysteme Leopardo RH (A respecter strictement)

Le projet est une **Suite d'Applications** (1 App = 1 Metier). Voici les roles definis "noir sur blanc" :

### Les 5 Applications Mobiles Flutter (`front/mobile_apps/`)
- **`leopardo_manager`** : Application dediee a la gestion du tenant (entreprise). Vue globale, affectation des roles, evolution.
- **`leopardo_hr`** : Application dediee aux Ressources Humaines. Suivi des employes, presences/absences, taches, et gestion du recrutement (ATS).
- **`leopardo_marketing`** : Application dediee aux marketeurs. Planification et publication en "1-clic" sur les differents reseaux sociaux.
- **`leopardo_platform_admin`** : Application ultra-securisee pour le Super-Admin (proprietaire du SaaS) pour gerer les abonnements et l'infrastructure.
- **`leopardo_kiosk`** : Application dediee au materiel physique (tablette pointeuse/biometrie).

### L'Ecosysteme Web (`front/`)
- **La Web App Client (`front/web` et admin-dashboard)** : Le portail web client est **unique**. Un employe, un RH ou un Manager se connecte au meme portail, mais l'interface s'adapte dynamiquement et change completement en fonction du role (RBAC).
- **La Web App Super-Admin** : Interface web reservee exclusivement a l'administration de la plateforme Leopardo (SaaS).

## ⚠️ NOUVELLE METHODE DE GESTION DE PROJET (Juillet 2026)

**ATTENTION AGENTS** : Les anciens dossiers `docs/PLAN_ACTION/` et `docs/PLAN_ACTION2/` sont **obsoletes et archives**. Il est **strictement interdit** de lire ces dossiers pour chercher du travail ou d'y creer de nouveaux fichiers Markdown de planification.

La gestion du projet Leopardo RH se fait desormais **exclusivement via GitHub Issues et GitHub Projects**.

### Regles de selection d'une tache (GitHub Issues)

1. **Lister les tickets ouverts** : `gh issue list --limit 50 --state open --json number,title,labels,assignees`.
2. **Filtrer** : Ne choisissez **que** les issues qui n'ont pas d'assignes (`assignees` vide) ET qui possedent des criteres d'acceptation clairs dans leur description (`gh issue view <number>`). Idealement, cherchez le label `Agent-Ready` ou `good first issue`.
3. **S'assigner** : Avant de coder, vous DEVEZ vous assigner l'issue, ou annoncer que vous la prenez pour eviter que deux agents ne fassent la meme chose.
4. **Fermeture automatique (CRITIQUE)** : Votre Pull Request (PR) **doit obligatoirement** contenir `Closes #<numero_issue>` dans sa description pour fermer l'issue automatiquement au merge.

### Comment demander une review

- Une fois le travail complet et verifie localement (tests pertinents, `shellcheck`/lint si applicable), passez la PR draft en "Ready for review" : `gh pr ready <numero>`.
- Ne jamais merger sa propre PR sans que les checks CI obligatoires (`gh pr checks <numero>`) soient verts.
- Assurez-vous que la description de la PR indique clairement quelle issue P0/P1 est resolue.

### Bibliotheque de prompts operationnels

Le dossier `dev-hub/prompts/` contient des prompts numerotes prets a l'emploi pour piloter les agents. Chaque prompt est un fichier Markdown autonome avec des instructions executables.

- **Carte rapide** : `dev-hub/prompts/00_AGENT_QUICK_CARD.md` — resume des regles vitales (2 min)
- **Vider le backlog** : `dev-hub/prompts/01_DRAIN_BACKLOG.md` — traiter tous les tickets
- **Audits** : prompts 02, 05-09 — auditer chaque surface du projet
- **CI/Merge** : prompts 03, 12 — reparer la CI, merger les branches
- **Anti-regression** : `dev-hub/prompts/13_REGRESSION_GUARD.md` — traquer les patterns interdits

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kitokoh/leopardo-hr](https://github.com/kitokoh/leopardo-hr) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
