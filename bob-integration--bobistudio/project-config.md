---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# Projet Bobi.Studio

## Architecture
- **Full-Docker** : conteneurs Docker sur des **nœuds** enrôlés (table `nodes`, pilotés par
  `node_driver` + agent-nœud). Le backend LXC/Proxmox a été **retiré** : plus de `proxmox.py`, plus de
  clonage de template ; création via `docker_driver` (MTL) / `docker_compute` (compute/média).
- Orchestrateur Flask central (rôle contrôleur) ; les nœuds exécutent les conteneurs.
- Pipeline vidéo/audio ST 2110 sur le **bus MXL** (SDK MXL, domaine `/dev/shm/mxl`) — production broadcast.

## Types de containers

**Il n'y a pas de liste fixe, et il ne faut pas en écrire une.** Tout type est un **plugin**
(cf. « Système de plugins » plus bas) : le registre les découvre au scan de `plugins/`, et ce
qui est installé varie d'une instance à l'autre — un paquet `.mxlplugin` ou la page Catalogue
en ajoute sans toucher au cœur. La source de vérité est `plugins/*/plugin.json`, jamais ce
fichier.

Ce qui est stable, ce sont les **rubriques** dans lesquelles ils se rangent (champ `nav.section`
du manifeste) : `sources`, `traitements`, `composition`, `medias`, `streams`, `mesure`. Un
plugin **sans `nav`** est rendu et versionné mais n'émet ni chip de palette ni entrée de menu.

Une seule exception mérite d'être nommée ici, parce qu'elle n'est pas un traitement comme les
autres : **`2110_io`**, le moteur ST 2110 bi-rôle (RX + TX) via MTL/DPDK en kernel-bypass. Il
est Docker-only, tourne sur la PF en AF-XDP, et c'est lui qui porte l'entrée/sortie du signal —
les autres plugins se contentent du bus MXL.

> Les VMID ne sont plus fixes (plage allouée dynamiquement ; un vmid = handle local jetable).

## Stack technique
- Python 3.13, Flask, SQLite
- Scripts vidéo : bus MXL (SDK MXL via `script_templates/bobimxl.py`), FFmpeg/GStreamer, numpy
- Agent dans chaque conteneur : port 8081 (deploy/start/stop) ; agent-nœud : lifecycle Docker + host-ops
- Métriques fps : port 8080

## Structure fichiers

Racine `/opt/bobistudio/` : `main.py` (point d'entrée Flask), les deux AMORCES d'installation
(`install.sh` — source locale ; `get.sh` — depuis GitHub, machine vierge ; toutes deux se bornent à
réunir les prérequis puis à lancer `install/install.py`), `venv/`, la DB
`db_bobistudio.db`, puis les dossiers `app/`, `services/`, `templates/`, `static/`, `plugins/`,
`node_agent/`, `i18n/`, `script_templates/`, `install/` (installeur unifié + flux Proxmox
hérité), `old/`. Le **module Python est `app/`** (tous les
`.py` ci-dessous y vivent ; `main.py` reste à la racine et importe `app`).

```
app/                  ← module Python principal (liste non-exhaustive)
  config.py           ← DB_PATH + défauts neutres (valeurs site : config_local.py, cf. Sécurité)
  database.py         ← SQLite (helpers get/set settings, containers, nodes, alerts, projets…)
  node_driver.py      ← pilote de nœud (agent-nœud : host-exec, images, networks, lifecycle)
  docker_driver.py    ← création/destruction conteneurs MTL (bobi-mtl, AF-XDP)
  docker_compute.py   ← création/destruction conteneurs compute/média (macvlan)
  containers.py       ← détruire/redémarrer (délègue aux drivers Docker)
  metrics.py          ← fps, IP
  scripts.py          ← render scripts (normalize_worker_udp_params, normalize_receiver_params)
  plugins.py          ← registre des plugins (scan, render, wiring, coerce_config, hooks)
  deploy.py           ← déploiement via agent
  allocations.py      ← VMIDs/IPs libres
  routes/             ← API REST Flask, PAQUET de 38 modules (il n'y a PLUS de `routes.py` unique).
                        `routes/__init__.py` enregistre le Blueprint ; chaque module porte
                        un domaine (`plugin_registry.py`, `catalogue_api.py`, `roles_api.py`…)
  catalogue.py        ← catalogue des plugins/services publiés (org GitHub de confiance,
                        `config.CATALOGUE_ORG`) — installe un paquet depuis la page Réglages
  routes/roles_api.py ← EMPLACEMENTS (table `production_roles`) : identité FONCTIONNELLE,
                        stable au remplacement — cf. « Identité d'un conteneur » plus bas
  settings.py         ← settings DB (get/set)
  auth.py             ← login/permissions
  backup.py           ← sauvegarde
  monitor.py          ← moniteur WebRTC par utilisateur (encodeur dédié + reaper)
  projects.py         ← projets snapshot/restore
  ptp.py              ← PTP par nœud (ptp4l/phc2sys via host-ops/agent, sampler, événements)
  io2110_flows.py     ← flux composables Sources/Destinations 2110 (modèle RX/TX du moteur)
  core_pool.py        ← allocateur de cœurs CPU par nœud (profil ressources par type)
  placement.py        ← CONSTAT du placement CPU réel (pendant de core_pool : lui calcule AVANT,
                        celui-ci vérifie APRÈS — bande isolée, cpuset posé, threads par cœur)
  host_ops.py         ← host-ops par nœud (ex-template_recreate ; exec via l'agent)
  node_health.py      ← sampler santé des nœuds (CPU/RAM/disque/PTP/GPU/RDMA/capteurs)
  mtl.py              ← préparation hôte MTL (hugepages, DDP, queues)
  gpu.py / gpu_pool.py← détection + allocation GPU NVIDIA (multiview GPU)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bob-integration/bobistudio](https://github.com/bob-integration/bobistudio) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-12 -->
