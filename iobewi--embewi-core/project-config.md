---
trigger: always_on
description: Contrôleur Kubernetes qui pilote des devices ESP32 via le contrat
---

# CLAUDE.md — Embewi Core (contrôleur Kubernetes)

Contrôleur Kubernetes qui pilote des devices ESP32 via le contrat
[`embewi`](https://github.com/iobewi/embewi) (`v1alpha1`). Les devices
(firmware [`embewi-agent-esp`](https://github.com/iobewi/embewi-agent-esp))
sont joignables en HTTPS :443 sur leur IP Wi-Fi, exposés comme endpoints d'un
Service Kubernetes selectorless.

Langue du projet : **français** (commentaires, doc, messages). Garder cette langue.

## Contrat de référence

Spec normative : **`contract/docs/embewi-contract-v2.md`** (protocole `v1alpha1`).
C'est la **source de vérité** pour toute interaction Core ↔ Agent.
`git submodule update --init` si `contract/` est vide.

Les sections **[NORMATIF]** sont des contraintes d'implémentation.
Les sections **[RÉSERVE]** sont hors MVP — ne pas implémenter sans décision.

## CRDs à implémenter

### `McuNode` — device physique enrôlé

```yaml
apiVersion: embewi.io/v1alpha1
kind: McuNode
metadata:
  name: embewi-a1b2c3
spec:
  nodeId: embewi-a1b2c3
  tokenRef:                        # référence au Secret K8s portant le token Bearer
    name: embewi-a1b2c3-token
    namespace: embewi
  tlsSecretRef:                    # optionnel — Secret kubernetes.io/tls (compat cert-manager)
    name: embewi-a1b2c3-tls        # absent = agent garde son cert auto-signé de build
status:
  ip: "192.168.10.42"             # mis à jour depuis heartbeat.ip à chaque réception
  lastHeartbeat: "2026-06-29T10:00:03Z"
  state: running
  apiVersion: v1alpha1             # négocié depuis GET /info (api_versions), absent → v1alpha1 supposé
  tlsCertDigest: "a3f9..."         # sha256(tls.crt+tls.key) du dernier cert TLS appliqué (suivi Core-side)
  lastRebootRequested: ""          # miroir de l'annotation embewi.io/reboot-requested traitée
  conditions:
    - type: Provisioned
      status: "True"
      reason: ProvisioningComplete
    - type: Ready
      status: "True"
      reason: HeartbeatOK
      message: "Heartbeat reçu il y a 3s"
```

Conditions McuNode (§8a) :

| Condition | `reason` | `status` | Déclencheur |
|---|---|---|---|
| `Provisioned` | `ProvisioningComplete` | True | node_id + token établis |
| `Provisioned` | `ProvisioningPending` | False | premier boot, portail AP actif |
| `Ready` | `HeartbeatOK` | True | heartbeat reçu < 2× période (10 s) |
| `Ready` | `HeartbeatTimeout` | False | aucun heartbeat > seuil |
| `Ready` | `NotProvisioned` | Unknown | jamais enrôlé |

### `McuDeployment` — déploiement firmware + workload

```yaml
apiVersion: embewi.io/v1alpha1
kind: McuDeployment
metadata:
  name: wheel-left
spec:
  nodeName: embewi-a1b2c3          # pin explicite privilégié
  firmware: registry.local/embewi/wheel-controller:v1.1.0
  configMapRef: wheel-left-gpio    # optionnel — absent = défauts build
  appPort: 9090                    # optionnel — 0/absent = pas de reconfiguration (POST /app/port)
status:
  deploymentId: wheel-controller-1.1.0
  activeSlot: ota_0
  firmwareDigest: "sha256:..."
  conditions:
    - type: Progressing
      status: "False"
      reason: DeploymentComplete
    - type: Available
      status: "True"
      reason: WorkloadReady
```

Conditions McuDeployment (§8a) :

| Condition | `reason` | `status` | Déclencheur |
|---|---|---|---|
| `Progressing` | `OTAInProgress` | True | PUT /ota/write ou pending_verify |
| `Progressing` | `DeploymentComplete` | False | OTA terminé, firmware stable |
| `Progressing` | `OTAFailed` | False | rollback ou état failed |
| `Available` | `WorkloadReady` | True | EndpointSlice.ready=true |
| `Available` | `PendingVerification` | False | state=pending_verify |
| `Available` | `DeviceDegraded` | False | state ∈ {degraded, rollback, failed} |
| `Available` | `HeartbeatTimeout` | False | device plus joignable |

**`Ready` synthétique** : `Ready=True` ← `Progressing=False` ET `Available=True`.
Compatible `kubectl wait mcudeployment/wheel-left --for=condition=Available`.

### `McuConfigMap` — config runtime

```yaml
apiVersion: embewi.io/v1alpha1
kind: McuConfigMap
metadata:
  name: wheel-left-gpio
data:
  gpio_button: "9"
  gpio_ws2812: "48"
  ntp_server: "ntp.local"
  # clés arbitraires — opaques pour l'agent, sémantique côté app
```

Limites NVS agent (à valider côté Core avant push) :
- Clé : 15 caractères max
- Valeur : 63 caractères max

Clés réservées agent (préfixe `_`) : ignorées silencieusement par `POST /config`.

## API agent à appeler (Core → ESP, contrat §4)

Préfixe : `/v1alpha1`. HTTPS :443. `Authorization: Bearer <token>` sur **tous** les endpoints.

### Séquence de réconciliation complète

```text
1. GET /info          → lit staged.state (idempotence), config_generation, app_port
2. GET /health        → optionnel, confirme l'état local avant OTA
3. POST /config       → si McuConfigMap diverge du NVS courant
4. POST /ota/prepare  → annonce le firmware (chip, size, digest, deployment_id)
5. PUT  /ota/write    → stream du .bin par chunks (Content-Range)
6. POST /ota/activate → set_boot_partition + reboot
7. [heartbeat] state=pending_verify → self-check en cours
8. [heartbeat] state=running + ota_validated=true → déploiement confirmé
```

Ordre canonique si config + OTA dans la même réconciliation :
**POST /config d'abord, OTA ensuite** — un seul reboot couvre les deux.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [iobewi/embewi-core](https://github.com/iobewi/embewi-core) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
