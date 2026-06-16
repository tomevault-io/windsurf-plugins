---
trigger: always_on
description: - Lis ce document en entier avant de coder.
---

# CLAUDE.md — Fine-Grained Proxy (FGP)

## First things first
- Lis ce document en entier avant de coder.
- Manière de parler : t'es un bro', tu ne prends pas de pincettes. Tu dis les choses telles qu'elles sont, même si c'est brutal. Pas de "peut-être", "il faudrait", "je pense que" — tu affirmes avec confiance et clarté. Tu ne laisses aucune place à l'ambiguïté ou au doute. Tu es direct, franc, et précis. Tu proposes des alternatives quand tu penses que c'est pertinent. Tu me parles comme à un collègue dev expérimenté. Tu peux me parler familièrement, mais toujours avec respect et professionnalisme. Si tu vois un problème ou une amélioration possible, tu le dis sans hésiter.

## Projet
- **Fine-Grained Proxy** : proxy HTTP stateless et API-agnostique qui ajoute des tokens fine-grained (scoping par méthode HTTP, chemin, et contenu du body) devant n'importe quelle API.
- **Zero storage** : aucune base de données. Le token + cible + auth + scopes + TTL sont chiffrés (gzip + AES-256-GCM) dans un blob.
- **Dual mode blob** : le blob peut être dans l'URL (`/{blob}/path`) ou en header (`X-FGP-Blob`). Le mode header est recommandé pour éviter les limites de 255 chars par segment d'URL.
- **Double clé** : le blob est déchiffrable uniquement avec une clé client (header `X-FGP-Key`) + un salt serveur. Le blob seul est inexploitable.
- **TTL** : expiration encodée dans le blob, vérifiée à chaque requête.
- **4 modes d'auth** : bearer, basic, scalingo-exchange, header custom. Scalingo est un cas d'usage parmi d'autres.
- **Blob v2/v3** : v2 = scopes string METHOD:PATH, v3 = scopes mixtes string + ScopeEntry avec body filters.
- **Body filters** (v3) : filtrage du contenu JSON des requêtes POST/PUT/PATCH (types : any, wildcard, stringwildcard, regex, not, and).
- **Logs stream** (ADR-0007) : page `/logs` avec SSE live par blob, opt-in via champ `logs: { enabled, detailed }` dans le blob (non-cassant sur v3). In-memory only (ring buffer par blob + purge inactivité), body `detailed` chiffré AES-256-GCM côté serveur avec la clé client (zero trust). Kill switch global `FGP_LOGS_ENABLED`.

## Stack
- **Runtime** : Deno
- **Framework** : Hono
- **Langage** : TypeScript (strict)
- **Crypto** : Web Crypto API native (AES-256-GCM, PBKDF2)
- **CSS** : Tailwind CSS 3 build-time (pas de CDN) → `static/styles.css`
- **Tests** : `deno test` — structure `tests/testu/` (unit), `tests/testi/` (integration), `tests/teste2e/` (e2e)
- **Lint/Format** : `deno lint` + `deno fmt`

## Scripts (deno task)
- `deno task build:css` — compile `src/ui/tailwind.css` → `static/styles.css` (Tailwind CSS 3, minifié)
- `deno task build:client` — compile `src/ui/client.ts` → `static/client.js` (esbuild, minifié)
- `deno task build:version` — résout le SHA git du commit et l'écrit dans `static/version.txt`
- `deno task build` — build:css + build:client + build:version (à lancer avant deploy)
- `deno task dev` — watch parallèle CSS + client + server (concurrently)
- `deno task start` — build + production
- `deno task deploy` — build + deployctl vers Deno Deploy
- `deno task test` — tous les tests
- `deno task test:unit` — tests unitaires
- `deno task test:integration` — tests intégration
- `deno task test:e2e` — tests e2e
- `deno task lint` — linter
- `deno task fmt` — formatteur
- `deno task fmt:check` — vérification formatage
- `deno task check` — type checking
- `deno task verify` — lint + fmt + check + test (pipeline complète)

## Structure
```
src/
  main.ts           — point d'entrée, Hono app
  constants.ts      — constantes partagées (FGP_SOURCE_HEADER, valeurs proxy/upstream, etc.)
  routes/           — routes Hono (UI, API, OpenAPI/Swagger, logs)
  middleware/        — middlewares (proxy, scopes, body filters, capture logs)
  crypto/           — chiffrement/déchiffrement blob, dérivation clé, gzip
  auth/             — client auth (Scalingo exchange), cache bearer, singleflight
  logs/             — feature /logs : config env, blob-id, capture, events, ip, store (ring buffer in-memory)
  ui/               — pages JSX (config-page, layout, logo/SEO, logs-page)
  ui/client/        — modules TS client (presets, body-filters, apps, generate, ttl, clipboard, scopes, test-scope, share-config, import-config, tabs, logs-tab, elements, types)
  ui/tailwind.css   — source Tailwind (build-time → static/styles.css)
tailwind.config.js  — config Tailwind (couleurs fgp, dark mode media)
static/             — assets compilés (client.js, styles.css) — gitignored
tests/
  testu/            — tests unitaires
  testi/            — tests intégration
  teste2e/          — tests e2e
docs/
  adr/              — Architecture Decision Records
  team/             — fiches de poste par rôle (dev, po, testeur, designer, lead)
  specs.md          — spécifications fonctionnelles v3
  limits.md         — limites fonctionnelles body filters
```

## Conventions code
- TypeScript strict, pas de `any`
- Pas de commentaires sauf POURQUOI non-évident
- Pas de default exports sauf `src/main.ts`
- Imports triés : deps externes, puis internes, ligne vide entre les deux
- Nommage : camelCase pour variables/fonctions, PascalCase pour types/interfaces
- Erreurs : utiliser `HTTPException` de Hono pour les erreurs HTTP

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [lsagetlethias/fine-grained-proxy](https://github.com/lsagetlethias/fine-grained-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-15 -->
