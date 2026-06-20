---
trigger: always_on
description: Manage EasyPanel server - deploy apps, manage projects, services, databases, domains, ports, mounts, backups, monitoring and infrastructure via tRPC API. All endpoints validated 2026-04-06.
---


# EasyPanel API Skill (v2 - Validated)

**Purpose**: Interact with EasyPanel server to manage projects, apps, databases, domains, ports, mounts, backups, monitoring, and deployments via tRPC API.

> All endpoints below were **tested and validated** against a live EasyPanel instance. Invalid procedures from v1 have been corrected.

## Connection Details

- **URL**: `https://<YOUR_EASYPANEL_HOST>:3000`
- **API Base**: `https://<YOUR_EASYPANEL_HOST>:3000/api/trpc`
- **Auth**: Bearer token from EasyPanel > Settings > Users > API Token
- **Use `-k` flag** if your instance uses self-signed certs

## Authentication

All requests use Bearer token in header:
```bash
-H "Authorization: Bearer <TOKEN>"
-H "Content-Type: application/json"
```

Generate a token in EasyPanel UI: Settings > Users > Generate API Token.
Or via API: `users.generateApiToken` (see Users section below).

## API Pattern

EasyPanel uses **tRPC**. All calls follow this pattern:

### Query (GET - read operations)
```bash
curl -sk "<API_BASE>/<procedure>?input=<URL_ENCODED_JSON>" \
  -H "Authorization: Bearer <TOKEN>"
```

Input must be URL-encoded: `{"json":{"key":"value"}}` -> `%7B%22json%22%3A%7B%22key%22%3A%22value%22%7D%7D`

### Mutation (POST - write operations)
```bash
curl -sk "<API_BASE>/<procedure>" \
  -H "Authorization: Bearer <TOKEN>" \
  -H "Content-Type: application/json" \
  -d '{"json": { ...params }}'
```

### Response Pattern
- Success: `{"result":{"data":{"json": <data>}}}`
- Success (void): `{"result":{"data":{"json": null}}}`
- Error: `{"error":{"json":{"message":"...","code":-32600,"data":{"zodErrors":{...}}}}}`
- Zod errors reveal required fields and types — use them to discover missing parameters

---

## PROJECTS

| Action | Procedure | Method | Validated Input |
|--------|-----------|--------|-----------------|
| List projects | `projects.listProjects` | GET | none |
| List projects + services | `projects.listProjectsAndServices` | GET | none |
| Can create project | `projects.canCreateProject` | GET | none |
| Create project | `projects.createProject` | POST | `{"json":{"name":"my-project"}}` |
| Inspect project | `projects.inspectProject` | GET | `{"json":{"projectName":"my-project"}}` |
| Update project env | `projects.updateProjectEnv` | POST | `{"json":{"projectName":"x","env":"KEY=value"}}` |
| Get docker containers | `projects.getDockerContainers` | GET | `{"json":{"projectName":"x","service":"svc-name"}}` |
| Destroy project | `projects.destroyProject` | POST | `{"json":{"name":"my-project"}}` |

> **Note**: `destroyProject` uses `name`, NOT `projectName`

---

## APP SERVICES (services.app.*)

### Lifecycle
| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| Create app | `services.app.createService` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Deploy | `services.app.deployService` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Start | `services.app.startService` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Stop | `services.app.stopService` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Restart | `services.app.restartService` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Destroy | `services.app.destroyService` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Inspect | `services.app.inspectService` | GET | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Get exposed ports | `services.app.getExposedPorts` | GET | `{"json":{"projectName":"x","serviceName":"y"}}` |
| Refresh deploy token | `services.app.refreshDeployToken` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` |

### Source Configuration
| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| Set image source | `services.app.updateSourceImage` | POST | `{"json":{"projectName":"x","serviceName":"y","image":"nginx:alpine","username":"","password":""}}` |
| Set GitHub source | `services.app.updateSourceGithub` | POST | `{"json":{"projectName":"x","serviceName":"y","owner":"org","repo":"repo","ref":"main","path":"/","autoDeploy":false}}` |
| Set Git source | `services.app.updateSourceGit` | POST | `{"json":{"projectName":"x","serviceName":"y","repo":"https://...","ref":"main","path":"/"}}` |
| Set Dockerfile source | `services.app.updateSourceDockerfile` | POST | `{"json":{"projectName":"x","serviceName":"y","dockerfile":"FROM nginx:alpine"}}` |
| Enable GitHub deploy | `services.app.enableGithubDeploy` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` (source must be GitHub) |
| Disable GitHub deploy | `services.app.disableGithubDeploy` | POST | `{"json":{"projectName":"x","serviceName":"y"}}` (source must be GitHub) |

### App Settings
| Action | Procedure | Method | Input |
|--------|-----------|--------|-------|
| Update env vars | `services.app.updateEnv` | POST | `{"json":{"projectName":"x","serviceName":"y","env":"KEY=val\nKEY2=val2"}}` |
| Update deploy config | `services.app.updateDeploy` | POST | `{"json":{"projectName":"x","serviceName":"y","replicas":1,"command":"","zeroDowntime":true}}` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [integralmarketingmx/easypanel-api-skill-for-claude](https://github.com/integralmarketingmx/easypanel-api-skill-for-claude) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
