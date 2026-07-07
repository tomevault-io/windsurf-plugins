---
trigger: always_on
description: Guide operationnel pour les agents autonomes dans CIR Cockpit.
---

# AGENTS.md

Guide operationnel pour les agents autonomes dans CIR Cockpit.

## Source et routage

- Ce fichier est l'entree courte pour Codex et les subagents.
- Ne pas lire `CLAUDE.md` par defaut. `CLAUDE.md` est l'adaptateur Claude Code et importe ces regles.
- Lire les documents lourds seulement quand ils sont utiles:
  - `docs/qa-runbook.md`: avant une livraison finale, une PR/merge/deploiement, une modification de QA, ou une demande explicite de verification complete.
  - `docs/testing.md`: quand la demande touche tests, couverture, E2E ou Playwright.
  - `docs/stack.md`: quand la demande touche versions, dependances, runtime, CI ou outillage.
  - `docs/plan.md`: seulement pour une etape majeure produit/architecture demandee.
- Docs volumineux (`docs/plan.md`, `docs/refonte-referentiels-triage.md`): lire d'abord la structure (titres/sommaire) puis seulement les sections utiles, jamais le fichier entier par defaut.
- `.mcp.json` est ignore par Git et local-only. Verifier les MCP reellement exposes par l'environnement actif avant de s'y fier.

## Regles de travail

- Lire d'abord les fichiers directement concernes, puis agir. Ne pas explorer tout le repo si le perimetre est clair.
- Respecter le worktree sale: ne jamais revert les changements non faits par l'agent.
- Modifier le minimum utile. Ne pas ajouter de fonctionnalite, refactor, doc ou fichier non demande.
- Zero donnees mockees, hardcodees, TODO non resolus ou texte decoratif dans le code livre.
- Preferer modifier un fichier existant plutot que creer un nouveau fichier.
- Zod: source unique dans `shared/schemas`, payloads API en `.strict()`, `safeParse` sur entrees/sorties externes, details de validation en francais.
- Erreurs: utiliser `createAppError()` / mappers / `reportError()` / `notifyError()`. Pas de `throw new Error()`, `console.error()` ou `toast.error()` directs hors exceptions existantes documentees.
- Garder les imports via alias `@/*` cote frontend et eviter les imports circulaires.

## Inspiration UI/UX

- Pour toute demande de revue, refonte, polish, audit ou decision UI/UX/design, consulter les inspirations actuelles avant de proposer ou modifier l'interface: Ramp (`https://ramp.com/`), Stripe (`https://stripe.com/fr`), Attio (`https://attio.com/`), Linear (`https://linear.app/`), Mistral (`https://mistral.ai/`) et SmoothUI (`https://smoothui.dev/`) pour les micro-interactions et composants animes.
- Utiliser ces sites comme references de direction visuelle, densite, hierarchie, micro-interactions, clarte SaaS et qualite de finition, sans copier leur contenu, leur marque, leurs assets proprietaires ou leurs textes.
- Si l'acces reseau ou navigateur est indisponible, le signaler explicitement et s'appuyer seulement sur les principes deja connus.

## Skills obligatoires

Invoquer le skill pertinent avant d'ecrire du code:

- `cir-cockpit-agent-router`: orientation repo locale, choix des docs/MCP/skills utiles et reduction du contexte charge.
- `cir-cockpit-qa-validation`: choix de validation CIR Cockpit par impact et rapport QA court.
- `cir-cockpit-api-contracts`: contrats tRPC, schemas Zod partages, services RPC front et routes/actions backend.
- `cir-cockpit-design`: tokens, densite et regles PO du design system local; a invoquer avant les skills design generiques pour toute UI visible.
- `vercel-react-best-practices`: composants, hooks ou pages React.
- `vercel-composition-patterns`: refactoring de composants React.
- `web-design-guidelines`: audit UI, accessibilite, UX.
- `impeccable`: design, redesign, polish, onboarding, empty state, formulaire, dashboard ou composant visible.
- `design-taste-frontend`: decision visuelle d'implementation UI.
- `layers-intro` puis `layers-*`: modele produit, vocabulaire metier, parcours, onboarding complexe, architecture d'information, objets/relations ou decisions UX profondes.
- `supabase-postgres-best-practices`: DB, migrations, RLS, indexes, queries, Edge Functions accedant a la DB.
- `cir-error-handling`: systeme d'erreurs.
- `systematic-debugging`: bug, echec test ou comportement inattendu avant correction.
- `vitest`: creation ou mise a jour de tests front.
- `playwright-cli`: verification de parcours UI/E2E.
- `pnpm`: scripts, workspace, dependances ou package manager.
- `trpc-type-safety`: procedure ou migration tRPC.
- `drizzle-orm`: schema ou queries Drizzle.
- `find-skills`: competence manquante ou capability inconnue.

## Docs et MCP

- Context7 est requis pour une decision d'implementation sur React, TanStack, tRPC, Drizzle, Vitest, Playwright, Zod ou autre librairie/framework. Pas requis pour une simple relecture documentaire ou un changement de texte.
- Supabase MCP est requis avant toute action DB, migration, RLS, Edge Function, deploy ou diagnostic runtime Supabase.
- Si le site doit etre verifie en direct dans Codex, utiliser le navigateur in-app Codex [@Navigateur](plugin://browser@openai-bundled) en priorite pour ouvrir, naviguer et inspecter le rendu. Garder Playwright/E2E pour les scenarios automatises, traces, screenshots reproductibles ou demandes explicites.
- Chrome DevTools/Playwright seulement si un parcours UI doit etre verifie; ne pas lancer d'E2E automatiquement sans demande utilisateur.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Nono8Six/CIR-Cockpit](https://github.com/Nono8Six/CIR-Cockpit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-07 -->
