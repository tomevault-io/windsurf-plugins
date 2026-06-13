---
trigger: always_on
description: * Règles Strictes pour l'Assistant AI
---

/**
 * Règles Strictes pour l'Assistant AI
 * ---------------------------------
 * CONTRAT STRICT ET NON NÉGOCIABLE
 * 
 * L'assistant DOIT :
 * 1. Respecter TOUTES les conventions définies dans ce document
 * 2. Vérifier SYSTÉMATIQUEMENT la conformité avec CHAQUE section avant toute réponse
 * 3. Refuser CATÉGORIQUEMENT toute action qui dévierait de ces conventions
 * 4. Pousser automatiquement les changements après un commit sans demander de confirmation
 * 5. Consulter SYSTÉMATIQUEMENT la documentation dans /docs pour :
 *    - Vérifier la cohérence des changements avec les patterns existants
 *    - Suivre les conventions de code établies
 *    - Respecter la structure des modules
 *    - S'assurer que les tests requis sont implémentés
 * 
 * Documentation du Projet
 * ---------------------
 * Structure :
 * /docs/
 * ├── architecture/     # Patterns et conventions techniques
 * │   ├── patterns.md   # Patterns communs (DTO, Controllers, Services)
 * │   ├── validation.md # Règles de validation
 * │   └── api.md       # Format des réponses API
 * ├── modules/         # Documentation des modules
 * │   ├── users.md     # Module Users
 * │   └── auth.md      # Module Auth
 * └── guides/          # Guides pratiques
 *     ├── setup.md     # Installation et configuration
 *     └── dev.md       # Guide du développeur
 * 
 * Sections à respecter obligatoirement :
 * - Git Commit Conventions
 * - GitFlow Conventions
 * - TypeScript General Guidelines
 * - NestJS Specific Guidelines
 * - Test Conventions
 * 
 * Points de contrôle spécifiques :
 * 1. Framework de Test
 *    - Utilisation EXCLUSIVE de Vitest
 *    - Interdiction formelle d'utiliser Jest ou tout autre framework de test
 *    - Tous les imports doivent utiliser { describe, it, expect, vi } from 'vitest'
 *    - Utilisation obligatoire de vi.fn(), vi.spyOn() etc.
 * 
 * 2. Commits Git
 *    - Format strict: <type>(<scope>): <subject>
 *    - Types autorisés UNIQUEMENT:
 *      feat, fix, docs, style, refactor, perf, test, chore
 *    - Aucune exception ou variation n'est permise
 *    - Le sujet doit être:
 *      * À l'impératif présent
 *      * Sans majuscule initiale
 *      * Sans point final
 * 
 * AUCUNE DÉROGATION N'EST AUTORISÉE
 * L'assistant doit refuser toute demande qui ne respecterait pas
 * l'intégralité des conventions définies dans ce document et dans /docs.
 */

/**
 * Conventions de Développement Fimaris API
 * --------------------------------------
 */

/**
 * Git Commit Conventions
 * ---------------------
 * 
 * Format: <type>(<scope>): <subject>
 * 
 * Types:
 * - feat: New feature
 * - fix: Bug fix
 * - docs: Documentation only changes
 * - style: Changes that do not affect the meaning of the code
 * - refactor: Code change that neither fixes a bug nor adds a feature
 * - perf: Code change that improves performance
 * - test: Adding missing tests or correcting existing tests
 * - chore: Changes to the build process or auxiliary tools
 * 
 * Scope:
 * - Module or feature area affected (e.g., auth, users, core)
 * - Optional, can be omitted if change affects multiple areas
 * 
 * Subject:
 * - Use imperative, present tense: "change" not "changed" nor "changes"
 * - Don't capitalize first letter
 * - No dot (.) at the end
 * 
 * Body (optional):
 * - Use "-" for bullet points
 * - Should explain the what and why, not the how
 * - Can be multiple lines
 * 
 * Example:
 * feat(auth): add jwt authentication
 * - Implement JWT token generation
 * - Add token validation middleware
 * - Setup refresh token mechanism
 */

/**
 * GitFlow Conventions
 * ------------------
 * 
 * Main Branches
 * ------------
 * main: Production code
 *   - Protected branch
 *   - Only accepts merges from release/* and hotfix/*
 *   - Each merge creates a version tag
 * 
 * develop: Development code
 *   - Protected branch
 *   - Base branch for feature branches
 *   - Contains latest delivered development changes
 * 
 * Feature Branches
 * ---------------
 * feature/*: New features
 *   - Branch from: develop
 *   - Merge to: develop
 *   - Naming: feature/[issue-id]-feature-name
 *   - Example: feature/123-user-authentication
 * 
 * Release Branches
 * --------------
 * release/*: Release preparation
 *   - Branch from: develop
 *   - Merge to: main and develop
 *   - Naming: release/v[version]
 *   - Example: release/v1.2.0
 *   - Only bugfixes, no new features
 * 
 * Hotfix Branches
 * -------------
 * hotfix/*: Production fixes
 *   - Branch from: main
 *   - Merge to: main and develop
 *   - Naming: hotfix/v[version]-fix-name
 *   - Example: hotfix/v1.2.1-fix-login
 * 
 * Workflow
 * -------
 * 1. Create feature branch from develop
 *    git flow feature start feature-name
 * 
 * 2. Work on feature
 *    git add .
 *    git commit -m "feat(scope): description"
 * 
 * 3. Finish feature
 *    git flow feature finish feature-name
 * 
 * 4. Create release
 *    git flow release start v1.2.0
 * 
 * 5. Finish release
 *    git flow release finish v1.2.0
 * 
 * 6. For hotfixes
 *    git flow hotfix start v1.2.1
 *    git flow hotfix finish v1.2.1
 * 
 * Protection Rules
 * --------------
 * main:
 *   - No direct pushes
 *   - Requires PR approval
 *   - Requires passing CI
 * 
 * develop:
 *   - No direct pushes
 *   - Requires PR approval
 *   - Requires passing CI

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HachemiH/prepa-cursor](https://github.com/HachemiH/prepa-cursor) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-13 -->
