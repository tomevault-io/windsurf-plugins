---
trigger: always_on
description: Ce fichier définit les conventions et règles à suivre lors de modifications sur le projet LevelUp.
---

# Instructions pour GitHub Copilot & Assistants IA

Ce fichier définit les conventions et règles à suivre lors de modifications sur le projet LevelUp.

---

## Contexte du Projet

**LevelUp** est un dashboard Streamlit pour analyser les statistiques Halo Infinite.

- **Stack** : Python 3.10+, Streamlit, DuckDB, SPNKr (API Halo)
- **Langue UI** : Français (traductions dans `src/ui/translations.py`)
- **Architecture** : DuckDB v6 (shared matches + SQL views)

---

## Environnement de référence (Windows)

Objectif : éviter les confusions d'interpréteur (PowerShell vs Git Bash/MSYS2) et les erreurs "module introuvable".

- **Python officiel** : `.venv` à la racine du repo (Python 3.12.10)
- **Interdit** : utiliser le Python MSYS2/MinGW (`pacman ... python/pip`) pour exécuter le projet
- **Règle d'or** : toujours lancer les outils via `python -m ...` (ne pas dépendre du `PATH`)

Packages critiques vérifiés dans `.venv` :
- `pytest==9.0.2`
- `duckdb==1.4.4`
- `polars==1.38.1`
- `pyarrow==23.0.0`
- `pandas==2.3.3`
- `numpy==2.4.2`

Healthcheck (à lancer avant de diagnostiquer un souci d'environnement) :
- `python scripts/check_env.py`

---

## Architecture des Données (v6)

| Données | Stockage | Chemin |
|---------|----------|--------|
| Référentiels | DuckDB | `data/warehouse/metadata.duckdb` |
| Matchs partagés | DuckDB | `data/warehouse/shared_matches_v2.duckdb` |
| Enrichissements joueur | DuckDB | `data/players/{gamertag}/stats.duckdb` |
| Archives | Parquet | `data/players/{gamertag}/archive/` |
| Config | JSON | `db_profiles.json` |
| Auth / token cache | DuckDB (`sync_meta`) + MSAL | `src/auth/` |

### Tables Principales

#### metadata.duckdb (référentiels)

| Table | Description |
|-------|-------------|
| `career_ranks` | Paliers et noms des rangs Halo |
| `citation_mappings` | Mappings médaille→citation |
| `mode_name_tr` / `mode_*` | Traductions et paramètres des modes de jeu |
| `weapon_labels` | Labels EN/FR par weapon_id filmshell (UBIGINT) |

#### shared_matches_v2.duckdb (centralisée)

| Table | Description |
|-------|-------------|
| `match_registry` | Registre central (1 ligne par match unique) |
| `match_participants` | Stats de tous les joueurs de tous les matchs |
| `highlight_events` | Événements filmés (`gamertag` **supprimé** en v6 — résolu via `v_gamertag_lookup`) |
| `medals_earned` | Médailles de tous les joueurs |
| `xuid_aliases` | Mapping global xuid→gamertag |
| `weapon_kills` | Kills par arme par joueur par match (filmshell) |

**Vues SQL garanties présentes en v6** (ne jamais recréer de guards `_has_shared_view`) :

| Vue | Description |
|-----|-------------|
| `v_gamertag_lookup` | FULL OUTER JOIN `xuid_aliases` + `match_participants` + `match_kill_events_latest` — résolution gamertag unifiée |
| `v_match_full` | `match_registry` enrichi avec métadonnées i18n (maps, playlists, game variants) |
| `v_weapon_kills` | `weapon_kills` avec `effective_weapon_id = COALESCE(reconciled_as, weapon_id)` |

> `v_killer_victim_full` a été **supprimée le 2026-08-02** : elle n'est plus garantie et ne doit
> plus être écrite dans une requête. Les paires tueur → victime se lisent dans
> `killer_victim_pairs` (historique) ou dans `match_kill_events` via sa vue
> `match_kill_events_latest` (ADR 0026 — jamais la table brute).

#### stats.duckdb (par joueur) — v5.1 allégée

> 8 tables supprimées : match_stats, match_participants, highlight_events,
> medals_earned, killer_victim_pairs, player_match_stats, xuid_aliases, teammates_aggregate

| Table | Description |
|-------|-------------|
| `player_match_enrichment` | performance_score, session_id, is_with_friends — **SEULE table match** |
| `personal_score_awards` | Awards objectifs (PersonalScores API) |
| `match_citations` | Citations calculées par match |
| `career_progression` | Historique rangs |
| `media_files` | Fichiers médias indexés |
| `media_match_associations` | Associations médias↔matchs |
| `sessions` | Sessions groupées |
| `sync_meta` | Métadonnées sync + cache MSAL sérialisé (token Microsoft) |
| `mv_*` | Vues matérialisées |

### Règles Streamlit v6

- Tout `st.plotly_chart` doit inclure `config=` (PLOTLY_CLEAN_CONFIG ou PLOTLY_STATIC_CONFIG)
- Préférer `@fragment_if_available` pour les sections interactives multi-charts
- Coéquipiers chargés depuis `shared.match_participants` (pas les DBs individuelles)
- `width="stretch"` au lieu de `use_container_width=True` (déprécié)
- Gamertag résolu **exclusivement** via `v_gamertag_lookup` — pas de `LEFT JOIN xuid_aliases` ad hoc
- Armes lues via `v_weapon_kills` — pas la table `weapon_kills` directement
- **Pas de guards** `_has_shared_view` / `_has_shared_table` — les vues V6 sont garanties présentes

---

## Authentification (`src/auth/`)

- **Entry point** : `src/auth/provider.py` — process cache (4 h TTL), MSAL silent refresh, `AuthRequiredError`, `start/complete_device_flow`
- **`LEVELUP_CLIENT_ID`** hardcodé dans `src/auth/_msal.py` — les utilisateurs finaux n'ont **plus** besoin de configurer Azure
- **`SPNKR_AZURE_CLIENT_ID`** (env var) reste supporté comme **fallback backend** — ne pas supprimer les fonctions qui l'exploitent ni celles gérant le refresh token

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [JGtm/LevelUp](https://github.com/JGtm/LevelUp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
