---
trigger: always_on
description: Ce fichier est le point d'entrée unique pour tout agent Claude travaillant sur ce dépôt.
---

# CLAUDE.md — Point d'entrée agent

Ce fichier est le point d'entrée unique pour tout agent Claude travaillant sur ce dépôt.
**Lire entièrement avant toute action.**

---

## Session startup checklist

Pour toute tâche non triviale :

1. Lire `README.md`
2. Lire `docs/Architecture.md`
3. Lire `docs/Decisions-Techniques.md`

Pour toute nouvelle fonctionnalité :

4. Lire `docs/Roadmap.md`

Avant de terminer :

5. Vérifier si la documentation doit être mise à jour.

---

## 1 — Base de connaissances

```
docs/
├── Architecture.md        — structure du code, patterns, dépendances
├── Carte-des-Modules.md   — 27 modules : backend, worker, persistence, export
├── Decisions-Techniques.md— pourquoi le code est comme il est (DT-01 à DT-13)
├── Roadmap.md             — fonctionnalités livrées et candidates
├── Maintenance-IA.md      — recettes : nouveau module, i18n, injection, release
└── Journal.md             — jalons techniques par version
```

### Quel document consulter selon la tâche

| Type de tâche | Documents à lire |
|---------------|-----------------|
| Ajouter un module | Architecture.md + Carte-des-Modules.md + Maintenance-IA.md |
| Modifier i18n / help_content | Architecture.md §i18n + Maintenance-IA.md |
| Corriger un bug UI (thème, layout) | Architecture.md §thème + Decisions-Techniques.md DT-02, DT-11 |
| Faire une release | Maintenance-IA.md §Release + Roadmap.md |
| Comprendre une décision existante | Decisions-Techniques.md |
| Ajouter une persistence JSON | Architecture.md §Persistence + Carte-des-Modules.md |
| Première prise en main | Architecture.md en entier |

---

## 2 — Règles de documentation

**La documentation fait partie du livrable. Une tâche est incomplète si les docs ne sont pas à jour.**

### Mettre à jour `docs/Carte-des-Modules.md` quand

- Un nouveau module est ajouté (ajouter une section complète).
- Un module change de backend, de persistence ou d'options d'export.
- Un module est supprimé.

### Enregistrer une décision dans `docs/Decisions-Techniques.md` quand

- Un choix technique non-évident est fait (technologie, architecture, format).
- Une alternative a été explicitement rejetée.
- Un piège ou un bug subtil a été découvert et corrigé (ex : DT-13 injection i18n).
- La règle : si quelqu'un pourrait se demander "pourquoi c'est fait comme ça ?", c'est une DT.

Format obligatoire :
```
## DT-NN — Titre court

**Contexte :** situation avant la décision.
**Décision :** ce qui a été choisi.
**Raisons :** pourquoi (liste).
**Alternatives rejetées :** ce qui n'a pas été retenu et pourquoi.
```

### Mettre à jour `docs/Roadmap.md` quand

- Une version est livrée (ajouter une ligne dans le tableau des versions).
- Une nouvelle fonctionnalité est confirmée par l'utilisateur (ajouter sous "Candidats vérifiés").
- Une idée est évoquée sans décision (ajouter sous "Idées évoquées sans décision formelle").
- Une fonctionnalité candidate est implémentée (la retirer des candidats, l'ajouter au tableau).

### Mettre à jour `docs/Journal.md` quand

- Une version est livrée (ajouter un jalon).
- Une décision architecturale majeure est prise (référencer la DT correspondante).

### Mettre à jour ce fichier (`CLAUDE.md`) quand

- Le nombre de langues i18n change.
- Le nombre de tests change significativement.
- Un nouveau fichier de persistence est ajouté.
- Une recette de workflow change (release, AUR, injection).

---

## 3 — Lancement et build

```bash
# Développement
python3 -m nmlinux.main    # ou ./nmlinux.sh

# Installé
nmlinux

# Build wheel (le sdist échoue — symlink dans aur/)
python -m build --wheel --no-isolation

# Tests (37 tests, logique pure, pas de Qt)
pytest tests/ -v
```

---

## 4 — Architecture (résumé)

Détails complets dans `docs/Architecture.md`.

`window.py` — `MainWindow` : `QListWidget` sidebar + `QStackedWidget`. Chaque page est enregistrée dans `_TOOLS` comme `(icon_names, label, PageClass, tooltip)`. Ajouter une page = append à `_TOOLS` + import + clés i18n + contenu help.

`nmlinux/core/` — utilitaires partagés :
- `i18n.py` — `tr(key, **kwargs)` : 8 langues (fr/en/es/de/it/pt/ja/zh), ~720 clés chacune. `fr` est la référence (toujours complète). Ajouter dans les **8 blocs**.
- `theme.py` — `is_dark()`, `color_ok()`, `color_err()` : appeler à la création des widgets, jamais au chargement du module. Surcharger `changeEvent(ApplicationPaletteChange)` + `update()` sur les widgets avec dessin custom.
- `cli_bar.py` — `get_cli_bar().set_cmd(cmd)` : dans `_update_cli()`, branché sur `showEvent` et les changements de paramètres.
- `settings.py` — singleton `AppSettings`. Accès via `.language`, **pas** `.get()`.
- `help_content.py` — `get_help(label)` : aide contextuelle 8 langues × 27 modules.
- `icons.py` — `themed_icon(*names)` : 21 SVG Lucide bundlés, couleur `#60a5fa`, aucun thème système requis.

Pattern page standard :

```python
class FooPage(QWidget):
    def __init__(self) -> None:
        super().__init__()
        self._build_ui()

    def _build_ui(self) -> None:
        layout = QVBoxLayout(self)
        layout.setContentsMargins(24, 24, 24, 24)
        # addWidget(table, 10) + addStretch(1) — évite le centrage Qt

    def showEvent(self, event) -> None:
        self._update_cli()
        super().showEvent(event)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [thongor77/nmlinux](https://github.com/thongor77/nmlinux) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
