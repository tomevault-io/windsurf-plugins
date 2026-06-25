---
trigger: always_on
description: > _Last updated : 2026-06-24, app v5.101.0_
---

# Gungnir — Project Guide & Conventions

> _Last updated : 2026-06-24, app v5.101.0_
> _Ce fichier est lu automatiquement par Claude Code à chaque session. Garde-le frais._

---

## Overview

Gungnir est une plateforme AI super-assistant full-stack avec architecture modulaire à plugins.

| Couche | Stack |
|---|---|
| **Backend** | FastAPI 0.115+ / Python 3.12 / SQLAlchemy 2 async / asyncpg / Postgres 16 + pgvector |
| **Frontend** | React 18.3 / TypeScript 5.6 / Vite 8 / Tailwind 3.4 / Zustand 5 / react-router 7 / i18next 24 |
| **Plugins** | 12 plugins backend (`backend/plugins/`) + frontend (`frontend/src/plugins/`) — pattern `manifest.json` + `routes.py` |
| **Infra** | Docker Compose / Nginx / runner GitHub Actions self-hosted / systemd |
| **IA / ML** | Anthropic / OpenAI / Google / OpenRouter SDKs / Ollama / `@huggingface/transformers` v3 (Whisper browser ONNX, WASM + WebGPU) |
| **Versioning** | SemVer dans `backend/core/__version__.py`, `frontend/package.json`, chaque `manifest.json` |
| **Branding** | ScarletWolf — `--scarlet` `#9B8260` (Dark Bronze par défaut), wolf identity |

---

## Quick Start

```bash
# 1. Postgres (dev)
docker compose -f compose.dev.yml up -d

# 2. Backend
export DATABASE_URL=postgresql+asyncpg://gungnir:gungnir@localhost:5432/gungnir
python -m uvicorn backend.core.main:app --host 127.0.0.1 --port 8000 --reload

# 3. Frontend
cd frontend && npm run dev
# → http://localhost:5173 (proxy /api → :8000)
```

---

## Architecture clé

- **Plugin system** : chaque plugin a un `manifest.json` (metadata + version + permissions) + `routes.py` (FastAPI). Frontend lazy-load depuis `src/plugins/`.
- **State frontend** : Zustand (`appStore`, `pluginStore`, `sidebarStore`)
- **API client** : `frontend/src/core/services/api.ts` — un seul fichier qui groupe tous les appels backend
- **Auth** : cookie HttpOnly `gungnir_session` depuis v5.0.0 (Bearer header en fallback pour scripts/tests/CI)
- **Agent tools (WOLF)** : `backend/core/agents/` — bash, filesystem, git, browser, web_fetch, etc.
- **Vite aliases** : `@` → `src/`, `@core` → `src/core/`, `@plugins` → `src/plugins/`

---

## 📋 Process & conventions globales

### Versioning & releases

- **SemVer strict** :
  - `MAJOR` = breaking change (DB migration incompatible, auth break, plugin API rewrite)
  - `MINOR` = nouvelle feature, ajout plugin, migration backward-compat
  - `PATCH` = fix, refacto sans impact utilisateur
- **Tag systématique au bump** : modifier `__version__.py` = créer immédiatement le tag `vX.Y.Z` + push. Pas optionnel.
- **CHANGELOG.md avant tag** : le push du tag déclenche le déploiement automatique VPS via `deploy-prod.yml`. Si le CHANGELOG n'est pas à jour AVANT, le release publié sera incomplet.
- **Source unique du `__version__`** : `backend/core/__version__.py` est la référence ; Vite expose `__APP_VERSION__` depuis `frontend/package.json` (à garder synchro manuellement).

### Git hygiene

- **Jamais `git add -A`** : staging explicite uniquement. `data/` Gungnir grouille de fichiers runtime untracked qu'on ne veut pas committer par accident.
- **PR systématique pour features et fix non-triviaux** : 1 PR = 1 chantier ciblé. Commit messages au format Conventional Commits-ish (`feat(scope): …`, `fix(scope): …`, `release: X.Y.Z — …`).
- **Squash merge** : on garde un historique linéaire sur `main`. `gh pr merge <N> --squash --delete-branch`.
- **Pas de force-push sur `main`**. Jamais.
- **Pré-commit pas en place** — TypeScript check (`npx tsc --noEmit`) et `python -c "import ast; ast.parse(open(f).read())"` à la main avant push pour les gros chantiers.

### Communication & process

- **Brainstorm-first** : sur tout sujet non trivial, proposer options/tradeoffs/reco AVANT toute exécution. Pas de fonçage. Cf. mémoire `brainstorm-first-then-act`.
- **Ship-then-await-feedback** : cœur livré = stop aux extras spéculatifs ; retours réels d'abord.
- **Preview HTML standalone pour UX** : quand on doit comparer 2-4 options visuelles, créer un fichier HTML autonome dans `~/` que l'user ouvre dans son browser. Évite de coder à l'aveugle et de devoir refaire.
- **Audit/clean en sub-agents parallèles** : pour les chantiers de revue (sécu, qualité), découper en 4-6 axes et lancer des sub-agents en parallèle qui écrivent leur rapport dans `audits/YYYY-MM-DD/` (gitignored).

### Sécurité (résumé des invariants)

- **Cookie HttpOnly + SameSite=Lax + Secure** pour la session — jamais de token en `localStorage` (XSS).
- **Aucun secret dans le code ni dans git** — `.env`, `.env.local`, `secrets.json` sont gitignored ; le redaction filter root masque les patterns connus dans les logs.
- **OAuth state HMAC + cookie binding** (RFC 6749 §10.12) sur tous les login flows externes.
- **`require_admin` / `require_curator` / `require_permission`** dans `auth_helpers.py` pour gater les routes sensibles.
- **`module:<plugin>` enforcement** dans `token_auth_middleware` (main.py) pour les routes `/api/plugins/<plugin>/*` quand l'user a un `role_id` assigné.
- **`_is_private_url`** (`web_fetch.py`) sur toute URL fetchée server-side — IPv4 + IPv6, fail-closed sur exception.
- **Whitelist regex stricte** sur tout identifiant utilisé en path filesystem (`plugin_id`, etc.) — pattern `^[a-z0-9][a-z0-9_.-]{0,63}$`.

### RGPD (résumé des invariants)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kevinggraphiste-hub/Gungnir](https://github.com/kevinggraphiste-hub/Gungnir) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-25 -->
