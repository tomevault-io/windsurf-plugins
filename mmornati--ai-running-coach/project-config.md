---
trigger: always_on
description: Espace de travail de coaching trail-running. Les agents IA gèrent l'entraînement,
---

# AI Running Coach — Workspace

Espace de travail de coaching trail-running. Les agents IA gèrent l'entraînement,
la santé, la nutrition et la stratégie de course, en persistant tout sous forme de
fichiers Markdown en français.

## Mandat linguistique

La langue des documents est **configurable** via `config/workspace.toml`
(versionné) et `config/workspace.user.toml` (gitignoré, overrides personnels) :

- **`[language].documents`** — langue des fichiers Markdown persistés
  (`activities/`, `medical/`, `nutrition/`, `planning/`, `rapports/`).
  Défaut : `fr` (ISO 639-1).
- **`[language].responses`** — langue des réponses à l'utilisateur.
  Défaut : `auto` (même langue que la requête).

**Règle de résolution** : lire `config/workspace.toml` à la racine du projet ;
si `config/workspace.user.toml` existe, ses valeurs priment (par clé). Les
agents et skills appliquent `documents` à tout fichier MD qu'ils persistent,
et `responses` à leurs réponses. Les instructions des agents/skills restent
en français — seule la langue de sortie des documents est paramétrée.

## Carte des dossiers

| Dossier | Contenu | Convention |
|---|---|---|
| `activities/` | Journaux d'entraînement | `YYYY-MM-DD_type.md` (running, trail, strength, indoor_cycling, home_trainer, hiking, elliptical, rest) |
| `medical/` | Sommeil, HRV, récupération, blessures, météo | `YYYY-MM-DD_health.md`, `YYYY-MM-DD_meteo.md` |
| `nutrition/` | Journaux nutrition & plans de ravitaillement | `YYYY-MM-DD_nutrition.md` |
| `planning/` | Plans d'entraînement, objectifs, stratégies de course | `active_objective.md` est la **source de vérité** de l'objectif courant |
| `rapports/` | Rapports de synthèse périodiques (propriété du **coach**) | `YYYY-MM-DD_rapport.md` |
| `resources/` | Base de connaissances (langue des documents) : running, nutrition, santé, récupération | Matériel de référence, citer lors des conseils. **Catalogues produits** (optionnels) : `resources/nutrition/catalogue-produits-*.md` = valeurs nutritionnelles par produit de l'athlète |

> **Note** : ces dossiers sont créés par l'utilisateur dans son espace de travail
> (voir `install.sh`). Ils sont exclus du dépôt public (`.gitignore`).

## Sous-agents

Délégation via l'outil `task` :

| Agent | Utilisation |
|---|---|
| `coach` | Plans d'entraînement, analyse des activités Garmin (**incl. HRR `recovery_hr_bpm` dans chaque retour de séance**), ajustements de séances, **push des séances au calendrier Garmin** (`schedule_workouts`, Garmin d'abord), rapports hebdomadaires. **Inclut toujours la météo + le créneau optimal (matin tôt / midi / soir) dans chaque validation hebdomadaire/journalière (charger le skill `weather-forecast`, résoudre le lieu via la règle de précédence stricte).** |
| `medical` | Analyse sommeil/HRV/récupération (**incl. HRR lors de l'évaluation de l'impact d'une séance**), protocoles blessures, gatekeeper de disponibilité, contraintes de coordination pour coach/nutritionniste |
| `nutritionist` | Macros, poids de course, plans de ravitaillement. **Pas de serveur MyFitnessPal** — les apports viennent des rapports manuels de l'utilisateur ; croiser avec les calories brûlées Garmin |
| `course-strategist` | Analyse GPX/URL de course → plan de course (allures ×3 scénarios, nutrition, météo, équipement), enrichissement points d'eau OSM, upload de parcours Garmin via l'outil `upload_course` |

Lors d'une délégation, écrire le prompt de tâche en anglais mais ajouter
explicitement **« Respond in <langue des documents> »** (résolue via
`config/workspace.toml` → `[language].documents`, défaut FRENCH) si la sortie
est destinée à l'utilisateur.

## Backends MCP

- **`garmin`** — activités, sommeil, HRV, readiness, **calendrier des séances planifiées (destination PRIMAIRE)**, upload parcours/séances. **Mode direct par défaut** : le serveur MCP `garmin` expose `garmin-mcp` avec une liste blanche d'outils (`GARMIN_ENABLED_TOOLS`). **Mode passerelle (optionnel, power user)** : `leanproxy_invoke_tool(server="garmin", tool="...")` via leanproxy-mcp (économie de tokens ~98 %, chargement paresseux des schémas).
- **`Intervals.icu`** — événements, wellness, séances planifiées (**SECONDAIRE** : uniquement si l'utilisateur le demande explicitement)
- **Absents localement** : `myfitnesspal` (utiliser les rapports manuels), `nexus-mcp` (RAG — déploiement Docker VPS uniquement ; localement utiliser `resources/` + historique MD)

## Règles de fraîcheur des données

- Avant d'invoquer les outils Garmin, vérifier si le fichier MD du jour existe déjà — ne récupérer que si la date a changé ou si le fichier manque.
- Après CHAQUE récupération de données, persister immédiatement le fichier MD correspondant (ne jamais sauter, ne jamais dumper le JSON brut dans le chat).

## Skills

- `garmin-workout-scheduling` — push des séances planifiées au calendrier Garmin (schéma DTO exact, détail force, idempotence, vérification après push)
- `intervals-icu-best-practices` — pièges de création/mise à jour d'événements (`workout_doc`, vérification `start_date`) ; secondaire, Garmin d'abord
- `garmin-sync-efficiency` — discipline de récupération pour éviter l'explosion du contexte

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mmornati/ai-running-coach](https://github.com/mmornati/ai-running-coach) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-21 -->
