---
trigger: always_on
description: | Environnement | URL |
---

# Landing Site - Claude Code Ultimate Guide

## URLs

| Environnement | URL |
|---------------|-----|
| **Production** | https://cc.bruniaux.com/ |
| **Custom Domain** | https://claudecode.bruniaux.com (à configurer) |
| **GitHub Repo** | https://github.com/FlorianBruniaux/claude-code-ultimate-guide-landing |

## Configuration Custom Domain

**Domaine cible**: `claudecode.bruniaux.com`

### Étapes de configuration

1. **Créer fichier CNAME** (à la racine du repo):
   ```
   claudecode.bruniaux.com
   ```

2. **Configurer DNS** (chez le registrar de bruniaux.com):
   - Type: `CNAME`
   - Host: `claudecode`
   - Value: `florianbruniaux.github.io`

3. **Activer HTTPS** dans GitHub Pages settings (après propagation DNS)

## Source de vérité

**Ce site est SECONDAIRE**. La source de vérité est le guide principal:
`/Users/florianbruniaux/Sites/perso/claude-code-ultimate-guide/`

**Workflow obligatoire**:
1. Modifier d'abord le guide principal
2. Puis synchroniser ici

Ne JAMAIS modifier les stats ou le contenu ici sans avoir d'abord mis à jour le guide principal.

## Éléments synchronisés depuis le guide

| Élément | Source (guide) | Fichiers landing |
|---------|----------------|------------------|
| Version | `VERSION` | index.html (footer + FAQ) |
| Templates count | `find examples/ -type f` | index.html (title, meta, badges, section), examples.html |
| Quiz questions | questions.json count | quiz.html, index.html |
| Guide lines | `wc -l guide/ultimate-guide.md` | index.html badges |
| Golden Rules | README.md | index.html section |
| FAQ | README.md | index.html (schema + HTML) |
| **Guide search index** | `guide/*.md` headings | `guide-data.js` (45+ entrées) |
| **Claude Code Releases** | `machine-readable/claude-code-releases.yaml` | index.html (banner + #releases section) |
| **AI Ecosystem tools** | `guide/ai-ecosystem.md` | index.html (#ecosystem section) |
| **Voice-to-Text** | `guide/ai-ecosystem.md#7-voice-to-text-wispr-flow` | index.html (#ecosystem card) |

## Valeurs actuelles (à maintenir synchronisées)

| Métrique | Valeur | Source |
|----------|--------|--------|
| Version | `3.39.1` | VERSION file |
| Templates | `247` | Count of examples/ files (excl. README/index) |
| Quiz questions | `271` | questions.json |
| Guide lines | `25,298` | ultimate-guide.md |

## Stack technique (post-migration Astro 5)

Le site a migré de HTML statique vers Astro 5. Les références à `index.html`, `examples.html`, `styles.css` ci-dessous sont **obsolètes** — voir les fichiers Astro dans `src/`.

| Ancien (statique) | Nouveau (Astro) |
|-------------------|-----------------|
| `index.html` | `src/pages/index.astro` + composants landing |
| `examples.html` | `src/pages/examples/index.astro` |
| `quiz.html` | `src/pages/quiz/index.astro` |
| `styles.css` | `src/styles/global.css` + `src/styles/components.css` |
| `search.js` + `guide-data.js` | `src/scripts/search.ts` + `src/data/search-index.ts` |

## Fichiers critiques

- **`src/data/examples-data.ts`**: 88 templates (source of truth pour examples + search)
- **`src/data/releases.ts`**: Changelog Claude Code
- **`src/data/security-data.ts`**: CVEs, campaigns, threats
- **`src/data/search-index.ts`**: Index de recherche (landing entries) — éditer ici pour ajouter des entrées
- **`src/data/guide-search-entries.ts`**: **GÉNÉRÉ** — ne jamais éditer directement (162 entrées depuis reference.yaml)
- **`src/data/guide-content-entries.ts`**: **GÉNÉRÉ** — ne jamais éditer directement (848 entrées H2 depuis src/content/docs/guide/)
- **`src/components/global/SearchModal.astro`**: Modal Cmd+K
- **`src/components/global/AnnouncementBanner.astro`**: Bandeau dismissible sous le nav (voir section dédiée ci-dessous)

## Announcement Banner — Workflow de mise à jour

Bandeau dismissible affiché sur toutes les pages, juste sous le header fixe (dans `<main>`, avant `<slot />`).

**Quand mettre à jour :** nouvelle page majeure, section importante ajoutée, milestone notable.

**Comment mettre à jour :**
1. Modifier le texte dans `src/components/global/AnnouncementBanner.astro`
2. Bumper `BANNER_ID` (ex: `banner-roles-2026-03` → `banner-guide-2026-04`) pour reset l'état dismissed chez tous les visiteurs
3. `pnpm build` + commit + push

**Contenu actuel (mars 2026) :** `banner-roles-2026-03` — AI Roles + guide en ligne sur /guide/

## Vérification avant modification

```bash
# Build complet (inclut rebuild search index)
pnpm build

# Dev local
pnpm dev
```

## Workflow obligatoire avant push sur main

Le contenu guide (`src/content/docs/guide/`) est **gitignored** — généré à chaque build depuis le repo guide voisin.

```bash
# 1. Préparer le contenu guide (lit ../claude-code-ultimate-guide/guide/)
node scripts/prepare-guide-content.mjs

# 2. Build complet pour valider
pnpm build

# 3. Si tout passe → push
git push
```

**Pourquoi** : le CI clone le repo guide et régénère le contenu à chaque deploy. Vérifier en local évite de casser le build en prod.

## Quiz Workflow (Markdown + Frontmatter)

**Source de vérité**: 256 fichiers Markdown dans `questions/`

### Architecture

```
questions/
├── _categories.yaml           # 15 catégories
├── {XX-slug}/
│   └── YYY-question-slug.md  # Frontmatter + question + explanation
scripts/
├── migrate-to-markdown.py     # One-time migration (historique)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [FlorianBruniaux/claude-code-ultimate-guide-landing](https://github.com/FlorianBruniaux/claude-code-ultimate-guide-landing) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-03 -->
