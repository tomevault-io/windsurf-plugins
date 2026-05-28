---
trigger: always_on
description: Licensed to the Apache Software Foundation (ASF) under one
---

<!--
Licensed to the Apache Software Foundation (ASF) under one
or more contributor license agreements.  See the NOTICE file
distributed with this work for additional information
regarding copyright ownership.  The ASF licenses this file
to you under the Apache License, Version 2.0 (the
"License"); you may not use this file except in compliance
with the License.  You may obtain a copy of the License at

  http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing,
software distributed under the License is distributed on an
"AS IS" BASIS, WITHOUT WARRANTIES OR CONDITIONS OF ANY
KIND, either express or implied.  See the License for the
specific language governing permissions and limitations
under the License.
-->

# CLAUDE.md

## Project Overview

Kubernetes operator for Apache Superset, built with the Go-based Operator SDK and Go 1.26+.
- Module: `github.com/apache/superset-kubernetes-operator`
- API group: `superset.apache.org/v1alpha1`
- License: Apache 2.0

## Developer Guide

See `docs/contributing/development-setup.md` and `docs/contributing/development-guidelines.md` for development setup, make commands, testing philosophy, code generation workflow, linting, CI/supply chain, and contributing guidelines. Adhere to the conventions documented there.

## Architecture

The operator uses a **single public CRD architecture** where the parent `Superset` resource resolves shared top-level and per-component configuration into parent-owned Kubernetes resources. See `docs/architecture/overview.md` for detailed design.

### CRD Hierarchy

- **Superset** (parent) — top-level CR with shared spec (top-level + per-component), environment, secretKey/secretKeyFrom, metastore (with uriFrom/passwordFrom), valkey (cache/broker/results), config, LifecycleSpec, NetworkingSpec, MonitoringSpec
- **Lifecycle tasks** — parent-owned Jobs + ConfigMap. Four sequential tasks: "clone" (database snapshot from external source), "migrate" (`superset db upgrade`), "rotate" (`superset re-encrypt-secrets` for secret key rotation), and "init" (`superset init`). Each task can be independently disabled via `disabled: true`. Clone supports `cronSchedule` for periodic re-execution.
- **Deployment components** — web server, Celery worker, Celery beat, Flower, websocket, and MCP server Deployments with Services/ConfigMaps/HPA/PDB as applicable.

**Key principles:**
- **Parent resolves and reconciles.** All layering logic lives in the parent controller, which writes Kubernetes resources directly.
- **Presence = enabled.** No `enabled: true/false`. If `celeryWorker: {}` is set, workers deploy. Lifecycle tasks (migrate, init) run by default; disable individual tasks via `disabled: true`. Clone runs when `spec.lifecycle.clone` is set. Rotate runs when `spec.lifecycle.rotate` is set.
- **Secrets never touch ConfigMaps.** In prod mode, CRD CEL validation rejects inline `secretKey`, `metastore.uri`, `metastore.password`, and `valkey.password`. Use `secretKeyFrom`, `metastore.uriFrom`, `metastore.passwordFrom`, or `valkey.passwordFrom` to reference Kubernetes Secrets (operator injects `valueFrom.secretKeyRef` env vars). In dev mode, inline secrets are allowed.
- **Per-component config rendering.** All Python components get `SECRET_KEY` rendered from `SUPERSET_OPERATOR__SECRET_KEY`. Web gets port config. Structured metastore renders an f-string URI from `SUPERSET_OPERATOR__DB_*` env vars. When `spec.valkey` is set, operator renders all cache configs (`CACHE_CONFIG`, `DATA_CACHE_CONFIG`, etc.), `CeleryConfig`, and `RESULTS_BACKEND` from `SUPERSET_OPERATOR__VALKEY_*` env vars. Websocket gets nothing (Node.js).

## Directory Layout

- `api/v1alpha1/` — CRD type definitions
  - `shared_types.go` — ImageSpec, MetastoreSpec, ValkeySpec (ValkeySSLSpec, ValkeyCacheSpec, ValkeyCelerySpec, ValkeyResultsBackendSpec), GunicornSpec, CeleryWorkerProcessSpec, SQLAlchemyEngineOptionsSpec, FlatComponentSpec, DeploymentTemplate, PodTemplate, ContainerTemplate, ScalableComponentSpec, ComponentSpec, AutoscalingSpec, PDBSpec
  - `superset_types.go` — SupersetSpec (environment, secretKey/secretKeyFrom, metastore with uriFrom/passwordFrom, valkey, config, sqlaEngineOptions, autoscaling, podDisruptionBudget), component specs (GunicornSpec on webServer, CeleryWorkerProcessSpec on celeryWorker, SQLAlchemyEngineOptionsSpec on all Python components except Flower), LifecycleSpec (clone/migrate/rotate/init tasks, upgradeMode, maintenancePage), AdminUserSpec, NetworkingSpec, MonitoringSpec, status types (LifecycleStatus, ComponentStatusMap, LastLifecycleImage)

- `internal/resolution/` — Pure Go spec resolution engine (zero controller-runtime deps)
  - `merge.go` — MergeMaps, MergeEnvVars, MergeVolumes, MergeVolumeMounts, MergeHostAliases, MergeContainers
  - `resolve.go` — ResolveScalar, ResolveOverridableMap/Slice/Value
  - `resolver.go` — ResolveComponentSpec() — core flattening engine
- `internal/config/` — Pure Go config rendering pipeline (zero controller-runtime deps)
  - `renderer.go` — Per-component superset_config.py generation
  - `gunicorn.go` — Gunicorn preset resolution, env var generation
  - `celery.go` — Celery worker preset resolution, command construction

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [apache/superset-kubernetes-operator](https://github.com/apache/superset-kubernetes-operator) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-28 -->
