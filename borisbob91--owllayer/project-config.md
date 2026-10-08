---
trigger: always_on
description: Ce document définit les règles et conventions à respecter pour tout agent IA intervenant dans le code d'OwlLayer.
---

# AGENTS.md — Instructions pour agents IA dans OwlLayer


Ce document définit les règles et conventions à respecter pour tout agent IA intervenant dans le code d'OwlLayer.
---

## Identité du projet

**OwlLayer** est un framework *Agentic UI* : il connecte un agent IA à une interface web via un protocole WebSocket propriétaire appelé **AITP** (Agent-to-Interface Transfer Protocol).

Les composants React/Vue/Svelte/Angular déclarent des **outils** (`useAgentTool`) que l'IA peut invoquer en temps réel. Le serveur orchestre les LLMs (OpenAI, Gemini). Le HITL (Human-in-the-Loop) protège les actions à risque.

Dépôt : monorepo **pnpm + Turborepo**. Toutes les commandes s'exécutent depuis la racine du monorepo.

---

## Structure du monorepo

```
owllayer/
├── packages/
│   ├── core/          # Types partagés, VoiceStateMachine, generateWidgetStyles, protocole AITP, OwlLayerClient
│   ├── server/        # Serveur WebSocket, orchestration LLM, sessions, HITL, OwlLayerServer
│   ├── adapter-openai/  # Adapter LLM OpenAI (GPT-4o, etc.)
│   ├── adapter-google/  # Adapter LLM Google (Gemini Live)
│   ├── react/         # SDK React : OwlLayerProvider, useAgentTool, useVoiceMode, WidgetInner
│   ├── ui/            # Runtime partage cross-framework, dashboard et devtools embarques
│   ├── vue/           # SDK Vue : OwlLayerWidget.vue, useVoiceMode composable
│   ├── svelte/        # SDK Svelte : OwlLayerWidget.svelte, createVoiceMode
│   ├── browser/       # SDK vanilla JS/Preact (Shadow DOM) : BrowserOwlLayer, VoiceManager
│   ├── shopify/       # Plugin Shopify (Liquid + JS) : OwlLayerShopify
│   └── woocommerce/   # Plugin WooCommerce : OwlLayerWoo
├── apps/
│   ├── demo-react/    # Démo React (e-commerce ShopMate)
│   ├── demo-vue/      # Démo Vue
│   ├── demo-svelte/   # Démo Svelte
│   ├── demo-browser/  # Démo vanilla JS (HTML + OwlLayer browser SDK)
│   ├── demo-server/   # Serveur de démo
│   └── docs-site/     # Site de documentation Astro / Starlight
├── issues/            # Analyses de bugs (canvas obligatoire avant fix)
├── features/          # Specs de nouvelles features (canvas obligatoire avant code)
├── docs/              # Documentation protocole, guides, règles audio
├── CONTRIBUTING.md    # Règles de contribution — LIRE EN PRIORITÉ
└── AGENTS.md          # Ce fichier
```

---

## Règles absolues — NE JAMAIS faire

Ces actions sont interdites sans exception, même si elles semblent améliorer le code :

```
✗ Renommer une variable, fonction, classe, type ou interface hors demande explicite
✗ Refactoriser du code non explicitement demandé
✗ Modifier des commentaires existants rédigés par d'autres
✗ Reformater du code non modifié (indentation, quotes, virgules...)
✗ Supprimer du code "mort" ou "inutilisé" de votre propre initiative
✗ Ajouter des dépendances npm non demandées
✗ Toucher des fichiers hors du périmètre de la demande
✗ Créer des abstractions ou helpers pour un usage unique
✗ Ajouter de la gestion d'erreur défensive sur du code interne
✗ "Améliorer" du code fonctionnel sans demande explicite
```

Si vous détectez quelque chose d'améliorable **hors scope** : signalez-le en commentaire de réponse uniquement — ne le modifiez pas.

---

## Règles de travail

### 1. Lisez CONTRIBUTING.md avant toute modification

[CONTRIBUTING.md](CONTRIBUTING.md) définit le workflow complet : canvas issue avant bug fix, canvas feature avant nouvelle fonctionnalité, domaines de responsabilité.

### 2. Respectez les domaines

Un agent IA doit travailler sur **un seul domaine par tâche** :

| Domaine | Périmètre |
|---|---|
| `core` | `packages/core/` uniquement |
| `server` | `packages/server/`, `packages/adapter-*/` |
| `react` | `packages/react/`, `apps/demo-react/` |
| `ui` | `packages/ui/` |
| `vue` | `packages/vue/`, `apps/demo-vue/` |
| `svelte` | `packages/svelte/`, `apps/demo-svelte/` |
| `browser` | `packages/browser/`, `apps/demo-browser/` |
| `angular` | `packages/angular/`, `apps/demo-angular/` |

Si une tâche touche `core` ET un SDK → intervenir séquentiellement, pas en même temps.

### 3. Listez les fichiers avant de coder

Avant de modifier quoi que ce soit, listez explicitement les fichiers qui seront touchés et obtenez confirmation si la demande est ambiguë.

### 4. Un fichier modifié = une raison explicite

Chaque fichier modifié doit avoir une justification directe dans la demande ou dans le document issue/feature référencé. Aucun fichier collatéral.

---

## Architecture technique — points critiques

### Protocole AITP

Communication WebSocket JSON entre `OwlLayerClient` (packages SDK) et `OwlLayerServer` (`packages/server`).
Messages clés : `TOOL_CALL`, `TOOL_RESULT`, `AGENT_RESPONSE`, `AUDIO_STREAM`, `AUDIO_OUTPUT`, `VOICE_INPUT_END`, `INTERRUPT`.

### Tools — cycle de vie

Les tools sont enregistrés via `useAgentTool` (React/Vue/Svelte) ou `OwlLayer.registerTool` (browser).  
**Ils n'existent côté serveur que quand le composant est monté.** Naviguer = tools changent.

```ts
useAgentTool({ name, description, risk }, handler)
// → enregistré au mount, désenregistré à l'unmount
```

### Shadow DOM — règle critique React

`WidgetInner.tsx` crée un `ShadowContainer` qui instancie son propre `createRoot` React **sans contexte OwlLayer**.  

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [borisbob91/owllayer](https://github.com/borisbob91/owllayer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
