---
trigger: always_on
description: > Ce fichier est le point d'entrée pour tout LLM de codage intervenant sur TranscrIA.
---

# AGENTS.md — Guide pour agents de codage

> Ce fichier est le point d'entrée pour tout LLM de codage intervenant sur TranscrIA.
> Lis-le intégralement avant de modifier le code.

## Commandes essentielles

```bash
# Installation complète (méthode recommandée)
./install.sh                         # Venv, PyTorch, dépendances, config, service systemd
./install.sh --no-service --no-torch # Réinstallation partielle

# Installation manuelle (si install.sh non adapté)
python3 -m venv venv && source venv/bin/activate
# Adapter le tag CUDA au driver local (cu121/cu124/cu126) ou utiliser ./install.sh --cuda.
pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu126
pip install -r requirements.txt
pip install -r requirements-dev.txt  # pytest, pytest-cov
python scripts/bootstrap_config.py --output config.yaml

# Préflight de diagnostic (GPU-free, sans effet de bord) — à lancer après config.yaml
venv/bin/python scripts/doctor.py            # config, schéma DB, script/serveur LLM, opencode, nœuds, dossiers
venv/bin/python scripts/doctor.py --strict   # avertissements = échec (code ≠ 0)
venv/bin/python scripts/doctor.py --llm-smoke # opt-in : test RÉEL opencode→LLM→texte (LLM up + VRAM, non GPU-free)

# Lancer l'application (dev)
source venv/bin/activate
python app.py

# Lancer l'application (production — service systemd)
sudo systemctl restart transcria.service   # redémarre proprement
sudo systemctl stop transcria.service
sudo systemctl status transcria.service
sudo truncate -s 0 /var/log/transcrIA.log  # remet le log à zéro (débogage)

# Scripts legacy (si systemd non disponible)
./start.sh    # log: /var/log/transcrIA.log, PID: /run/transcrIA.pid
./stop.sh
./status.sh

# Déploiement conteneurisé (cf. docs/DOCKER.md)
scripts/setup_docker_gpu.sh          # active l'accès GPU Docker (nvidia-container-toolkit + CDI) ; --check pour vérifier
scripts/docker_quickstart.sh         # turnkey : prérequis + config/.env + build + compose up + /health (--cpu, --down)
# Entrypoint conteneur par rôle (jamais install.sh) — web|scheduler|resource-node|migrate|all
python -m transcria.deploy.entrypoint <role>

# Tests — ⚠️ TOUJOURS via le venv (python système = pas de python-docx → 21 faux échecs)
venv/bin/python -m pytest tests/ -q              # suite mockée majoritaire, pas de GPU requis
venv/bin/python -m pytest tests/test_auth.py -v

# ⚠️ PostgreSQL de test : par défaut conftest lance un Postgres éphémère via pg_ctl/initdb
#    — qui ÉCHOUE en root (« initdb erreur 1 »). En local/root, pointer vers un Postgres
#    EXISTANT (mode noproc, sans initdb) en exportant :
#      export TRANSCRIA_TEST_PG_HOST=127.0.0.1 TRANSCRIA_TEST_PG_PORT=5432 \
#             TRANSCRIA_TEST_PG_USER=postgres TRANSCRIA_TEST_PG_PASSWORD=...
#    (le serveur doit autoriser CREATE DATABASE ; chaque run crée/détruit une base jetable).

# CI (.github/workflows/tests.yml) — 3 gates, reproductibles en local :
ruff check transcria/ inference_service/ --line-length 140 --select E,W,F,I
mypy transcria/ inference_service/ --ignore-missing-imports
venv/bin/python -m pytest tests/ -q --cov=transcria --cov-fail-under=75   # seuil 75 % (actuel ~80 %)
# Tests réseau (faux serveurs sur vrai socket) : marqueur "integration" → -m integration / -m "not integration"
# ⚠️  Tests E2E : TOUJOURS utiliser le python du venv (pyannote et Cohere n'y sont que là)
venv/bin/python tests/test_e2e_workflow.py --skip-llm               # E2E rapide (1 GPU)
venv/bin/python tests/test_e2e_workflow.py                          # E2E complet (GPUs + LLM requis)
venv/bin/python tests/test_e2e_workflow.py --keep                   # Conserve le job pour inspection
venv/bin/python tests/test_e2e_workflow.py --audio tests/test2.mp3  # Autre fichier audio

# Lint / format (cf. CI ci-dessus pour les commandes exactes qui gatent)
# black n'est PAS utilisé. Respecte le style du fichier que tu modifies.
```

## Gates de vérification (rituel obligatoire avant de déclarer « fini »)

Leçons durement acquises (beta.8/beta.9), NON négociables :

```bash
# Les commandes EXACTES de la CI, sur l'ARBRE ENTIER (jamais une version ciblée) :
set -o pipefail   # OBLIGATOIRE : `mypy … | tail -1` a déjà laissé passer une CI rouge
venv/bin/python -m ruff check transcria/ inference_service/ --line-length 140 --select E,W,F,I
venv/bin/python -m mypy transcria/ inference_service/ --ignore-missing-imports
venv/bin/python -m pytest tests/ -q --cov=transcria --cov-fail-under=75
```

1. **Jamais de pipe qui masque un code de sortie** sur un gate (`cmd | tail` → l'échec
   devient invisible). `set -o pipefail` ou pas de pipe du tout.
2. **UI : piloter ET voir.** Un banc Playwright de GESTES réels (pas des GET), puis
   **revue visuelle de chaque capture** — la revue attrape ce que les assertions ratent
   (7 vrais défauts attrapés ainsi sur l'éditeur SRT en une semaine).
3. **Tests aux limites** sur tout champ nouveau : vide / 1 car. / très long / unicode
   exotique / type incorrect. Oracle : jamais de 500, message FR qui guide.
4. **E2E GPU réel** pour tout ce qui touche le pipeline (instance jetable, config de
   prod copiée en scratch, PG jetable) — le mock ne prouve rien sur les phases LLM.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Martossien/transcria](https://github.com/Martossien/transcria) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-11 -->
