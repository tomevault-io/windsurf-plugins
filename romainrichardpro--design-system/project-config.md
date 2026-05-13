---
trigger: always_on
description: Ce fichier est lu automatiquement par Claude Code à chaque session.
---

# CLAUDE.md — Design System Romain Richard

Ce fichier est lu automatiquement par Claude Code à chaque session.
Il constitue le contrat permanent du projet : règles, conventions, décisions, workflow.
Ne pas modifier sans validation de Romain.

---

## 1. Contexte du projet

Design System open-source de Romain Richard.

- **GitHub** : https://github.com/RomainRichardpro/design-system
- **Figma** : fichier `skRy27piDeBGQwD8Bi0EAU` ("POC-NEW-DS")
- **Licence** : MIT

Romain est **Lead System Designer** (6 ans de design, 2 ans de spécialisation DS).

---

## 2. Rôles

| Qui | Rôle |
|---|---|
| Romain | Lead Designer — conception Figma, décisions design, validation |
| Claude (claude.ai) | Tech Lead / Architecte — réflexion, architecture, composants complexes |
| Claude Code | Exécution — remplacement de fichiers, tâches répétitives, automatisation |

**Claude Code ne prend pas de décisions d'architecture.** Il exécute ce qui a été décidé avec claude.ai.

---

## 3. Stack technique

```
pnpm + Turborepo (monorepo)
TypeScript strict (pas de any)
React
CSS Modules + CSS Variables
Storybook 7
Vitest + Testing Library + jest-axe
```

---

## 4. Structure du monorepo

```
design-system/
├── packages/
│   ├── tokens/          # Design tokens → CSS Variables + JSON
│   ├── react/           # Composants React
│   └── storybook/       # Documentation et vitrine
├── CLAUDE.md
├── turbo.json
├── pnpm-workspace.yaml
└── tsconfig.json
```

**Namespaces** : `@romainrichardpro/tokens`, `@romainrichardpro/react`, `@romainrichardpro/storybook`

---

## 5. Tokens

### Source de vérité
Figma Variables — fichier `skRy27piDeBGQwD8Bi0EAU`

### Collections Figma
- Primitives
- Semantic Numbers
- Semantic Colors (light / dark)
- Typography

### Build (Style Dictionary v4)
- `usesDtcg: true`
- Transformers custom : `color/figma-hex`, `number/px-or-opacity`
- Format Figma JSON propriétaire : `$value` est un objet `{hex, alpha, components}` — toujours lire via `token.original.$value`
- Sorties : `colors-light.css`, `colors-dark.css`, `numbers.css`, `typography.css`, `tokens.json`

### Nomenclature
Échelle numérique (`spacing.01`, `spacing.02`…) — pas de t-shirt sizing.

### Exports package tokens
```
@romainrichardpro/tokens/css/colors-light
@romainrichardpro/tokens/css/colors-dark
@romainrichardpro/tokens/css/numbers
@romainrichardpro/tokens/css/typography
```

### Imports Storybook preview
```js
@romainrichardpro/tokens/css/colors-light
@romainrichardpro/tokens/css/numbers
@romainrichardpro/tokens/css/typography
```

---

## 6. Règles absolues — à ne jamais enfreindre

### 6.1 Figma est la source de vérité
- Toujours lire Figma avant d'implémenter ou modifier un composant.
- Reproduire exactement ce qui est dans Figma : variants, props, états, tailles, tokens.
- Si quelque chose semble étrange ou incohérent : **le signaler, mais l'implémenter quand même**.
- C'est Romain qui décide si c'est une erreur ou une intention design.
- **Ne jamais corriger, améliorer ou interpréter le design de sa propre initiative.**

### 6.2 Anti-régression
- Identifier le périmètre exact de chaque changement avant de toucher au code.
- Ne modifier que ce périmètre — rien d'autre.
- Ne jamais modifier ce qui fonctionne déjà.
- Valider mentalement chaque ligne modifiée avant de l'écrire.

### 6.3 Accessibilité (WCAG 2.1 AA — non négociable)
- Navigation clavier complète
- Focus visible
- ARIA correct
- Compatibilité lecteurs d'écran
- Contrastes suffisants
- Logique d'états accessible

### 6.4 Sécurité
- Pas de `dangerouslySetInnerHTML`
- Pas de patterns XSS
- Pas de dépendances inutiles

### 6.5 Qualité de code
- TypeScript strict — pas de `any`
- CSS Modules + CSS Variables uniquement
- Pas de Tailwind
- Pas de sur-ingénierie
- Code lisible, maintenable, documenté

---

## 7. Workflow composants

### Ordre impératif pour chaque nouveau composant ou modification

```
1. Lire Figma via MCP (get_design_context)
2. Faire le diff avec le code existant
3. Identifier le périmètre exact des changements
4. Implémenter uniquement ce qui a changé
5. Vérifier les tests existants — ne pas les casser
6. Ajouter ou mettre à jour les tests
7. Mettre à jour la story Storybook
8. Push GitHub
```

### IDs Figma — format
- URLs Figma : format tiret (`18-765`)
- Appels MCP : format deux-points (`18:765`)

---

## 8. Conventions composants React

### API
- `children` pour le contenu textuel (pas de prop `label`)
- Props booléennes sans valeur : `<Button loading />` pas `<Button loading={true} />`
- `forwardRef` systématique
- `displayName` défini

### CSS Modules
- Classes : kebab-case avec préfixe sémantique (`level-primary`, `size-m`, `is-loading`)
- États disabled : sélecteur `:disabled` natif uniquement — **pas** `[aria-disabled='true']`
- L'attribut `aria-disabled` sert à la communication avec les lecteurs d'écran, pas au style

### États loading
- Pas de `disabled` HTML natif en état loading
- `aria-busy="true"` + `aria-disabled="true"` exposés
- Click bloqué via handler (`if (loading) return`)
- `loadingLabel` (défaut : `"Chargement en cours"`) dans un `<span class="srOnly">`

### data-attributes
- `data-level` et `data-size` obligatoires sur le `<button>` natif (utilisés par les tests)

---

## 9. Composants existants


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [RomainRichardpro/design-system](https://github.com/RomainRichardpro/design-system) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
