---
trigger: always_on
description: <!-- nodefony:start -->
---

# AGENTS.md — nodefony

<!-- nodefony:start -->

> **N'invente jamais du code Nodefony : génère-le, imite-le, vérifie-le.**
> Trois actes pour toute tâche : **LIRE** (ce fichier, puis la doc pointée) →
> **GÉNÉRER** (`npx nodefony create …` produit du vrai code, à imiter) →
> **VÉRIFIER** (`npm run verify` — UNE commande : types + lint + tests + câblage).
>
> **Le réflexe, avant d'écrire le MOINDRE fichier** : un générateur le
> produit-il ? Écrire à la main un CRUD, un controller, une entité ou un
> squelette de module, c'est le signal que tu as raté une commande de la
> table ci-dessous — arrête-toi et lance-la.
>
> 🔴 **Charge le skill `nodefony-dev` AVANT ta première modification**, quelle
> que soit la tâche : il porte la conduite complète et dit quel skill
> spécialisé prendre. Et sache ceci dès maintenant — **la référence est
> INSTALLÉE, mais `rg` ne descend pas dans `node_modules`** : 70 pages
> paraissent absentes. Une commande les lit, avec la ligne exacte :
> `node node_modules/@nodefony/devkit/skills/nodefony-dev/scripts/docs.mjs <termes>`.
>
> **Tu RENDS une réponse ?** `return this.renderJson(obj)` pour du JSON ;
> `this.setContextHtml()` puis `return this.render(html)` pour une PAGE — le nonce
> CSP de la requête s'écrit **`this.context?.cspNonce`** (le `?.` n'est pas
> optionnel : `context` est `ContextType | undefined`, sans lui le code ne
> compile pas), à recopier dans tout `<script>` en ligne. Ne touche JAMAIS
> `this.response` à la main : poser `Content-Type` toi-même court-circuite la
> négociation, et un `this.response as any` est le signal que tu as raté la façade.
>
> **Tu LIS une liste ?** Elle se BORNE, toujours. Le service d'une entité hérite
> `findPage({ limit: 25 })` — il ne charge que `limit + 1` lignes et rend
> `{ items, hasNext }` ; sinon `find(criteria, { limit })`. Un `find` sans borne
> matérialise la table ENTIÈRE : indolore sur les quelques lignes du poste de
> développement, fatal sur les dizaines de milliers de la production. Il te faut
> une projection de colonnes, une CTE, une agrégation ? Descends au natif **avec
> son type** — `import type { DrizzleDb } from "@nodefony/drizzle"` puis
> `orm.getNativeConnection<DrizzleDb>()`. Sans le paramètre de type tu reçois
> `unknown`, et il ne te reste qu'un `as any` que le contrôle refuse.
>
> **Tu SERS un fichier ?** Trois façades, jamais `createReadStream` à la main :
> `this.renderMediaStream(f)` pour un média qu'on parcourt (`Range` → 206),
> `this.streamFile(f)` pour le fichier entier, `this.renderFileDownload(f)` pour
> forcer le téléchargement. Le faire soi-même rend une réponse que le client ne
> peut pas lire — le détail, plus bas, est MESURÉ.

## Générateurs — appelle-les, ne recompose jamais leur sortie de mémoire

| Besoin                                                                                                  | Commande                                                                                         |
| ------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------ |
| Module applicatif (workspace npm)                                                                       | `npx nodefony create module <nom>`                                                               |
| Controller HTTP **et** WebSocket (même classe)                                                          | `npx nodefony create controller <nom> --kind hello\|rest\|realtime\|duplex\|example`             |
| Controller **réservé à une habilitation** — garde de classe + rôle déclaré dans la hiérarchie           | `npx nodefony create controller <nom> --role ROLE_X`                                             |
| Ressource REST **complète** — entité + service + controller CRUD + tests (ne JAMAIS l'écrire à la main) | `npx nodefony create entity <Nom> --fields "sku:string:unique price:float"`                      |
| Service métier seul — la logique réutilisable, hors de tout controller                                  | `npx nodefony create service <Nom> [--inject <AutreService>] [--module <m>]`                     |
| Frontend Vite — page, formulaire de connexion et temps réel LIVRÉS                                      | `npx nodefony create front <nom> --frontend <react\|vue\|angular\|svelte> [--module <m>]`        |
| Commande CLI `nodefony <module>:<action>`                                                               | `npx nodefony create command <action> [--module <m>] [--phase onReady\|onRegister\|onPostReady]` |

**Ces dossiers ne s'écrivent JAMAIS à la main** — y déposer un fichier signifie
que tu as raté une commande de la table ci-dessus :

| Tu t'apprêtes à écrire dans…     | Lance plutôt                                                             |
| -------------------------------- | ------------------------------------------------------------------------ |
| `nodefony/entity/`               | `npx nodefony create entity <Nom> --fields "…"`                          |
| `nodefony/controllers/`          | `npx nodefony create controller <nom> --kind …`                          |
| `nodefony/service/`              | `npx nodefony create service <Nom>` (ou `create entity`, qui en pose un) |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nodefony/nodefony](https://github.com/nodefony/nodefony) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
