---
trigger: always_on
description: - `docs/ARCHITECTURE.md` — diagrammes Mermaid complets : services, pipeline ingestion, flow recherche (séquence SSE), RRF, 3 stores, multi-tenant, déploiement
---

# CLAUDE.md — Moteur de recherche IA Immobilier

## Documentation du projet

- `docs/ARCHITECTURE.md` — diagrammes Mermaid complets : services, pipeline ingestion, flow recherche (séquence SSE), RRF, 3 stores, multi-tenant, déploiement
- `docs/DATA_MODEL.md` — modèle de données, ERD, règles de design du schéma, pattern adaptateur, spec WordPress complète
- `docs/ROADMAP.md` — 6 sprints détaillés jour par jour avec Test Gates
- `docs/TESTING.md` — stratégie de test 4 niveaux, organisation par package
- `docs/SPRINT_0.md` — instructions de démarrage immédiat
- `docs/assets/` — schémas SVG visuels (pipeline, flow recherche, enrichissement)
- `infrastructure/migrations/001_initial_schema.sql` — schéma SQL exécutable complet
- `packages/core/src/property.types.ts` — types TypeScript + exemple d'adaptateur

## Vision du projet

Moteur de recherche conversationnel IA embarquable sur les sites d'agences immobilières (SaaS multi-tenant). L'utilisateur décrit son projet en langage naturel ("maison familiale, 2 enfants, un chien, du soleil, 45 min de Bordeaux, 350k max"), l'agent IA extrait les critères, effectue une recherche hybride et retourne des biens pertinents avec argumentation, comme un vrai agent immobilier.

**Les deux exigences non-négociables : rapidité (ressenti < 1s grâce au streaming) et pertinence.**

## Architecture

### Principe fondamental

Tout le travail lourd se fait à l'INGESTION, jamais à la recherche. Quand l'utilisateur cherche, tout est déjà calculé, enrichi et indexé.

### Les 3 stores (un bien vit aux 3 endroits)

- **PostgreSQL (Supabase)** : source de vérité complète. Schéma normalisé ~120 colonnes. PostGIS pour le géo.
- **Typesense** : index full-text BM25 allégé. Filtres stricts (budget, pièces, ville, tags). Répond < 10ms.
- **Qdrant** : vecteurs 1536 dim (text-embedding-3-small). Recherche sémantique cosine. Répond < 40ms.
- **Redis** : cache requêtes (TTL 5min) + sessions conversation.

### Flow de recherche (latence cible P95 < 1.2s)

1. Message utilisateur → widget → API (SSE ouvert)
2. Claude Haiku extrait les critères → JSON `{must_have, nice_to_have, rayon_geo}` (~300ms)
3. Typesense (filtres stricts) + Qdrant (similarité sémantique) en PARALLÈLE (~40ms)
4. Fusion RRF (Reciprocal Rank Fusion, k=60) + business rules (exclusion must_have manquant, bonus nice_to_have) → top 5 (~5ms)
5. Claude Sonnet génère la réponse en streaming avec les 5 biens en contexte compact (< 500 tokens de biens)

### Pipeline d'ingestion

1. Source (WordPress REST API en priorité V1) → adaptateur normalise vers le schéma unifié
2. Géocodage via API Adresse data.gouv.fr si pas de lat/lng
3. Enrichissement géo en parallèle (~200ms) : Overpass OSM (écoles, commerces, transports), Open-Meteo (ensoleillement), Géoportail/BRGM (risques)
4. 1 appel Claude : tags lifestyle, profils acheteurs, points forts, texte synthétique pour embedding
5. Dispatch parallèle vers les 3 stores

**Optimisation clé : `source_hash`** (MD5 du payload brut). Si inchangé au sync suivant → zéro retraitement (pas d'appel LLM, pas d'appel géo, pas de réindexation). ~95% d'économie sur les syncs quotidiens.

## Structure du monorepo (pnpm workspaces)

```
immo-ai-search/
├── apps/
│   ├── api/              # Backend Express + TypeScript (SSE streaming)
│   ├── widget/           # React 19 Web Component (Vite, Shadow DOM, < 80kb)
│   └── admin/            # Back-office agences (V1 minimal)
├── packages/
│   ├── core/             # Types partagés, schémas Zod, enums
│   ├── db/               # Client postgres.js + runner migrations SQL
│   ├── search/           # Clients Typesense + Qdrant + algorithme RRF
│   ├── ingestion/        # Pipeline orchestrateur + adaptateurs sources
│   ├── ai/               # Wrappers Claude API + OpenAI embeddings
│   └── geo/              # Clients Overpass, Open-Meteo, Géoportail
├── infrastructure/
│   ├── docker/           # docker-compose.yml dev + prod
│   ├── migrations/       # SQL versionnés (001_initial_schema.sql, ...)
│   └── scripts/          # seed.ts, reset, benchmark
└── .github/workflows/    # CI
```

## Stack technique

- **Runtime** : Node.js 22, TypeScript strict partout, React 19 (widget)
- **DB** : PostgreSQL 16 + PostGIS via Supabase (prod) / Docker local (dev). Driver `postgres.js` — PAS d'ORM, PAS de Prisma (le schéma utilise PostGIS, TEXT[], JSONB, triggers — Prisma gère mal). Migrations SQL manuelles versionnées.
- **Search** : Typesense 27 + Qdrant (les deux self-hosted Docker)
- **IA** : Claude Haiku (extraction critères), Claude Sonnet (agent conversationnel), OpenAI text-embedding-3-small (embeddings)
- **Tests** : Vitest (unit/integration), Testcontainers (vrais Postgres/Redis en test), MSW (mock APIs externes), Supertest (API), React Testing Library (widget), Playwright (E2E), k6 (charge)

## Infrastructure

- **Dev local** : docker-compose (Postgres+PostGIS, Typesense, Qdrant, Redis) + hot reload
- **Prod** : VPS Hostinger (Docker Compose : API + Typesense + Qdrant + Redis + Caddy HTTPS) + Supabase (PostgreSQL, pooler port 6543) + Cloudflare (DNS, cache widget.js)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [HugoBld/omora](https://github.com/HugoBld/omora) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-21 -->
