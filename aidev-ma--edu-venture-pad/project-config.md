---
trigger: always_on
description: Standards Git et versioning pour IFES
---


# Git & Versioning Standards - Enterprise Grade

## Branches Strategy

IFES utilise **Git Flow** avec branches features numérotées.

### Structure des Branches

```
main (production)
  ├── develop (intégration)
  │   ├── 001-jwt-auth-impl (feature JWT auth)
  │   ├── 002-payment-integration (feature paiements)
  │   ├── 003-course-management (feature gestion cours)
  │   └── hotfix/fix-login-bug
```

### Branches Principales

**main:**
- Code production stable
- Déployé automatiquement
- Protected: Require PR + review
- Tag pour chaque release (v1.0.0, v1.1.0)

**develop:**
- Code intégration (pre-production)
- Toujours stable et testable
- Merge features via PR
- Protected: Require PR

### Speckit et specs depuis `develop` : `SPECIFY_FEATURE`

Pour travailler sur **`specs/NNN-feature/`** tout en restant sur **`develop`** (sans `git checkout NNN-feature` pour chaque spec) :

- Définir dans PowerShell : **`$env:SPECIFY_FEATURE = 'NNN-feature'`** (nom du dossier sous `specs/`) **avant** les scripts `.specify/scripts/powershell/*.ps1` qui résolvent le répertoire de la feature.
- Les **agents** ne doivent pas supposer que le nom de branche Git = la spec active ; en cas de doute, utiliser ou demander **`SPECIFY_FEATURE`**.
- Détail et obligations agents : [.cursor/rules/spec-driven-development.mdc](mdc:.cursor/rules/spec-driven-development.mdc) (section « Mode intégration : branche develop + SPECIFY_FEATURE »).

### Feature Branches

**Naming Convention:**
```
<number>-<feature-name>

Exemples:
001-jwt-auth-impl
002-payment-integration
003-course-management
004-multi-agent-system
```

**Règles:**
- Créer depuis `develop`
- Nom descriptif et numéroté
- Une feature = une branche
- Delete après merge

**Workflow:**
```bash
# Créer feature branch
git checkout develop
git pull origin develop
git checkout -b 003-course-management

# Développer
git add .
git commit -m "feat: add course creation endpoint"

# Pousser régulièrement
git push origin 003-course-management

# Merge request vers develop
# Après review et merge, supprimer branch
git checkout develop
git pull origin develop
git branch -d 003-course-management
```

### Hotfix Branches

**Naming Convention:**
```
hotfix/<bug-name>

Exemples:
hotfix/fix-login-redirect
hotfix/cors-error
hotfix/payment-validation
```

**Workflow:**
```bash
# Créer depuis main
git checkout main
git pull origin main
git checkout -b hotfix/fix-login-redirect

# Fix
git add .
git commit -m "fix: correct login redirect logic"

# Merge vers main ET develop
git checkout main
git merge hotfix/fix-login-redirect
git push origin main

git checkout develop
git merge hotfix/fix-login-redirect
git push origin develop

# Tag release
git tag v1.0.1
git push origin v1.0.1

# Delete hotfix
git branch -d hotfix/fix-login-redirect
```

## Commit Messages

### Conventional Commits

IFES utilise **Conventional Commits 1.0.0**.

**Format:**
```
<type>(<scope>): <subject>

<body>

<footer>
```

### Types

```
feat:     Nouvelle fonctionnalité
fix:      Correction de bug
docs:     Documentation uniquement
style:    Formatage (pas de changement code)
refactor: Refactoring (pas de nouvelle feature ni fix)
perf:     Amélioration performance
test:     Ajout/modification tests
chore:    Maintenance (build, dépendances, etc.)
ci:       Configuration CI/CD
revert:   Revert d'un commit précédent
```

### Scopes

```
frontend:  Code frontend (React)
backend:   Code backend (Express)
auth:      Authentification
courses:   Gestion cours
payments:  Paiements
database:  Migrations Supabase
docs:      Documentation
config:    Configuration
```

### Exemples

✅ **CORRECT:**

```bash
# Feature simple
git commit -m "feat(auth): implement JWT authentication"

# Fix avec description
git commit -m "fix(backend): correct CORS configuration

CORS was blocking requests from Lovable subdomains.
Updated origin validation to support wildcard subdomains.

Closes #42"

# Breaking change
git commit -m "feat(courses)!: change course pricing structure

BREAKING CHANGE: Course price is now stored in cents instead of MAD.
Migration required for existing data."

# Multiple changes
git commit -m "refactor(frontend): reorganize service layer

- Move authService to services/authService.ts
- Update imports in components
- Add JSDoc comments"
```

❌ **INCORRECT:**

```bash
# Trop vague
git commit -m "fix bug"
git commit -m "update code"
git commit -m "changes"

# Pas de type
git commit -m "authentication implementation"

# Plusieurs types
git commit -m "feat: add login + fix bug + update docs"

# Capitalized subject
git commit -m "feat: Add Login Feature"
```

### Règles Commits

1. **Type obligatoire:** Toujours commencer par type
2. **Subject lowercase:** Pas de majuscule après `:` (sauf noms propres)
3. **No period:** Pas de point final dans subject
4. **Imperative mood:** "add", "fix", pas "added", "fixed"
5. **Max 72 chars:** Subject ≤ 72 caractères
6. **Body si nécessaire:** Expliquer WHY et WHAT, pas HOW
7. **Breaking changes:** Ajouter `!` après type et `BREAKING CHANGE:` dans footer
8. **Reference issues:** `Closes #42`, `Fixes #123`, `Refs #456`

## Pull Requests

### PR Title

**Format:** Même que commit (Conventional Commits)

```
feat(auth): implement JWT authentication
fix(cors): support Lovable subdomains
docs: update API documentation
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/aidev-ma) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
