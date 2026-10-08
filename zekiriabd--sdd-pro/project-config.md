---
trigger: always_on
description: > Document chargé **à la demande** (`Read @.sdd/docs/conventions.md`).
---

# SDD_Pro — Conventions strictes (référence)

> Document chargé **à la demande** (`Read @.sdd/docs/conventions.md`).
> Pas en system prompt.

Ce document est l'index des conventions opératoires du framework. Le
détail complet de chaque règle vit dans `.sdd/rules/` ; ce fichier
en est la **TOC commentée**.

> **Voir aussi** (audit mineur #9 v7.0.0-alpha 2026-06-05 — cross-ref) :
> `@.sdd/docs/glossary.md` pour la **taxonomie canonique** (agents,
> rôles, termes métier SDD_Pro). Ce fichier-ci décrit les **règles
> d'exécution** ; le glossaire décrit le **vocabulaire** sur lequel elles
> reposent. Si un terme apparaît ici sans définition explicite, le
> glossaire en est la source.

## 1. Anti-derive (universel)

Aucun agent n'invente :
- des SFD, BR, AC, FD non présents dans la FEAT parente
- une couleur, un libellé, un composant ou une icône non visible dans
  le HTML source (ou non listé dans le stack UI actif)
- une lib, un pattern, un middleware non déclaré dans le stack actif

Sur ambiguïté irrécupérable → `STOP + ERROR`. Pas de devinette.

## 2. Format ERROR — 3 lignes obligatoires

```
ERROR: <agent ou commande> — <résumé court>
CAUSE: <cause précise>
FIX: <action utilisateur concrète>
```

Aucun agent ne produit de stack trace verbeux.

## 3. Idempotence

Toutes les commandes sont idempotentes : relancer `/us-generate {n}`
écrase les US précédentes. Aucun état caché entre invocations. Le
bootstrap arch + scaffolding DB sont idempotents par construction.

## 4. Lecture sélective

Aucun agent ne fait de Glob `workspace/us/*.md` ou `workspace/ui/*.html`
quand il traite UNE US. Chaque agent ne lit que ses fichiers de
travail.

## 5. Parallélisme des agents Dev (borné)

`/dev-run {n}` invoque Dev-Backend ET Dev-Frontend **en parallèle**
sur les US, **par batches de `MaxParallel` US** (default 3,
configurable via `--max-parallel N` ou `MaxParallel: N` dans
`## Project Config`).

Pour `U` US et `MaxParallel = K` :
- `B = ceil(U / K)` batches enchaînés séquentiellement
- chaque batch : jusqu'à `2 × K` invocations dev-* parallèles dans un
  seul message
- les batches `i+1` démarrent quand TOUTES les invocations du batch
  `i` sont terminées

## 6. Plan inline (pas de phase TASKS)

Les agents `dev-backend` et `dev-frontend` planifient eux-mêmes la
liste des fichiers à produire à partir de l'US + (HTML mockup) +
stacks actifs. **Pas de fichier `workspace/tasks/...`**, pas de
Lead-Dev.

## 7. Bootstrap unifié (pas d'agent DB séparé)

L'agent `arch` absorbe l'introspection DB et le scaffolding
Database-First en Phase B. Si `DatabaseType: none`, la phase B est
silencieusement skip. **Pas d'agent `db` séparé**.

## 8. CLAUDE.md par projet (digest, depuis v2.5)

Arch produit en Phase C **un fichier CLAUDE.md par projet généré** :
- `workspace/src/{BackendName}/CLAUDE.md` — architecture backend
- `workspace/src/{AppName}/CLAUDE.md` — architecture frontend + UI
- `workspace/src/{LibName}/CLAUDE.md` (si LibName défini) — contrats
  partagés

Hash-validé (`stack-md-hash` en frontmatter, calculé sur stack.md +
stacks pertinents). Si périmé → fallback automatique sur les stacks
bruts. Régénération au prochain `/arch-init`.

## 9. HTML mockup comme source de vérité visuelle (depuis v4)

`dev-frontend` lit **directement** le fichier HTML statique
`workspace/ui/{n}-{m}-{Name}.html` (texte, pas vision multimodale).

Trois sources de vérité hiérarchisées :
- **HTML mockup** = source de vérité visuelle : libellés exacts,
  structure des zones, classes CSS, couleurs inline ou dans `<style>`,
  ordre des éléments, hiérarchies typographiques
- **Stack UI §2 + §7** = source de mapping vers les primitives du
  design system actif. Le HTML brut est traduit, jamais recopié tel
  quel
- **US** = source de vérité workflow (validation, navigation,
  conditions d'affichage)

Au STEP 11 **Fidelity Check (text-based)** : grep des libellés et
structures clés extraits du HTML source dans le markup généré.

## 10. Mode Plan Only / From Plan (depuis v2.4)

`/dev-plan {n}` invoque les agents dev-* en mode `:plan` : ils
planifient inline puis écrivent le plan dans
`workspace/plans/{n}-{m}-{Name}.{back|front}.md` **sans coder**.

`/dev-run {n}` détecte automatiquement les plans existants et les
**consomme** (mode From Plan).

**Plan-then-review gate** : `/sdd-full {n}` rend `/dev-plan {n}`
**obligatoire** quand `/feat-validate` retourne 🟡 WARN ou 🔴 NO-GO
ET que `--force` est passé. Par défaut, `/sdd-full` **STOP** sur 🟡
WARN ou 🔴 NO-GO.

**Plan-review opt-in sur GO** : `--plan` (ou
`PlanReviewDefault: true` dans `## Project Config`) déclenche le
plan-then-review **même sur GO**.

## 11. Persistence cross-stack

Chaque stack backend déclare une section `## 8. Persistence` avec :
- §8.1 DB Drivers (matrice `DatabaseType → package`)
- §8.2 Connection String Pattern (builder/URL canonique du langage)
- §8.3 Scaffolding tool (`dotnet ef` / `prisma db pull` / `sqlacodegen`)

## 12. Cleanup BREAKING CHANGES post-build

`dev-*` peuvent renommer une section `## BREAKING CHANGES` du
`CLAUDE.md` projet en `## BREAKING CHANGES — RESOLVED {YYYY-MM-DD}`
quand le build est vert et que la dérive est résolue (cf.
`@.sdd/rules/ownership.md §6.bis`).

## 13. Capabilities core vs on-demand


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zekiriabd/SDD-Pro](https://github.com/zekiriabd/SDD-Pro) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
