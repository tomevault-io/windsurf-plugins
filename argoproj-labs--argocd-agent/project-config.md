---
trigger: always_on
description: Complete reference for all agent component configuration parameters.
---

# Agent Configuration Reference

Complete reference for all agent component configuration parameters.

## Server Connection

### Server Address

| | |
|---|---|
| **CLI Flag** | `--server-address` |
| **Environment Variable** | `ARGOCD_AGENT_REMOTE_SERVER` |
| **ConfigMap Entry** | `agent.server.address` |
| **Type** | String |
| **Default** | `""` |
| **Required** | Yes |

Address of the principal server to connect to.

**Example:** `argocd-agent-principal.example.com`

### Server Port

| | |
|---|---|
| **CLI Flag** | `--server-port` |
| **Environment Variable** | `ARGOCD_AGENT_REMOTE_PORT` |
| **ConfigMap Entry** | `agent.server.port` |
| **Type** | Integer |
| **Default** | `443` |
| **Range** | 1-65535 |

Port on the principal server to connect to.

## Agent Operation

### Agent Mode

| | |
|---|---|
| **CLI Flag** | `--agent-mode` |
| **Environment Variable** | `ARGOCD_AGENT_MODE` |
| **ConfigMap Entry** | `agent.mode` |
| **Type** | String |
| **Default** | `autonomous` |
| **Valid Values** | `autonomous`, `managed` |

Mode of operation for the agent.

### Namespace

| | |
|---|---|
| **CLI Flag** | `--namespace`, `-n` |
| **Environment Variable** | `ARGOCD_AGENT_NAMESPACE` |
| **ConfigMap Entry** | `agent.namespace` |
| **Type** | String |
| **Default** | `argocd` |
| **Required** | Yes |

Namespace to manage applications in.

### Credentials

| | |
|---|---|
| **CLI Flag** | `--creds` |
| **Environment Variable** | `ARGOCD_AGENT_CREDS` |
| **ConfigMap Entry** | `agent.creds` |
| **Type** | String |
| **Default** | `""` |
| **Format** | `<method>:<configuration>` |

Credentials to use when connecting to server.

**Valid Methods:**

| Method | Format | Description |
|--------|--------|-------------|
| `mtls` | `mtls:` | Mutual TLS authentication using client certificate |
| `header` | `header:` | Header-based authentication for service mesh environments |
| `userpass` | `userpass:<path>` | **[DEPRECATED]** Username/password authentication |

**Examples:**

- mTLS: `mtls:`
- Service mesh: `header:`
- Userpass (deprecated): `userpass:/app/config/creds/userpass.creds`

## TLS Configuration

### Insecure TLS

| | |
|---|---|
| **CLI Flag** | `--insecure-tls` |
| **Environment Variable** | `ARGOCD_AGENT_TLS_INSECURE` |
| **ConfigMap Entry** | `agent.tls.client.insecure` |
| **Type** | Boolean |
| **Default** | `false` |

Skip verification of remote TLS certificate. **Development only.**

### Root CA Secret Name

| | |
|---|---|
| **CLI Flag** | `--root-ca-secret-name` |
| **Environment Variable** | `ARGOCD_AGENT_TLS_ROOT_CA_SECRET_NAME` |
| **ConfigMap Entry** | `agent.tls.root-ca-secret-name` |
| **Type** | String |
| **Default** | `argocd-agent-ca` |

Name of the secret containing the root CA certificate.

### Root CA Path

| | |
|---|---|
| **CLI Flag** | `--root-ca-path` |
| **Environment Variable** | `ARGOCD_AGENT_TLS_ROOT_CA_PATH` |
| **ConfigMap Entry** | `agent.tls.root-ca-path` |
| **Type** | String |
| **Default** | `""` |

Path to file containing root CA certificate for verifying remote TLS.

### TLS Secret Name

| | |
|---|---|
| **CLI Flag** | `--tls-secret-name` |
| **Environment Variable** | `ARGOCD_AGENT_TLS_SECRET_NAME` |
| **ConfigMap Entry** | `agent.tls.secret-name` |
| **Type** | String |
| **Default** | `argocd-agent-client-tls` |

Name of the secret containing the TLS client certificate.

### TLS Client Certificate

| | |
|---|---|
| **CLI Flag** | `--tls-client-cert` |
| **Environment Variable** | `ARGOCD_AGENT_TLS_CLIENT_CERT_PATH` |
| **ConfigMap Entry** | `agent.tls.client.cert-path` |
| **Type** | String |
| **Default** | `""` |

Path to TLS client certificate file.

### TLS Client Key

| | |
|---|---|
| **CLI Flag** | `--tls-client-key` |
| **Environment Variable** | `ARGOCD_AGENT_TLS_CLIENT_KEY_PATH` |
| **ConfigMap Entry** | `agent.tls.client.key-path` |
| **Type** | String |
| **Default** | `""` |

Path to TLS client private key file.

### TLS Minimum Version

| | |
|---|---|
| **CLI Flag** | `--tls-min-version` |
| **Environment Variable** | `ARGOCD_AGENT_TLS_MIN_VERSION` |
| **ConfigMap Entry** | `agent.tls.min-version` |
| **Type** | String |
| **Default** | `""` (Go default) |
| **Valid Values** | `tls1.1`, `tls1.2`, `tls1.3` |

Minimum TLS version to use when connecting to the principal.

### TLS Maximum Version

| | |
|---|---|
| **CLI Flag** | `--tls-max-version` |
| **Environment Variable** | `ARGOCD_AGENT_TLS_MAX_VERSION` |
| **ConfigMap Entry** | `agent.tls.max-version` |
| **Type** | String |
| **Default** | `""` (highest available) |
| **Valid Values** | `tls1.1`, `tls1.2`, `tls1.3` |

Maximum TLS version to use when connecting to the principal.

### TLS Cipher Suites

| | |
|---|---|
| **CLI Flag** | `--tls-ciphersuites` |
| **Environment Variable** | `ARGOCD_AGENT_TLS_CIPHERSUITES` |
| **ConfigMap Entry** | `agent.tls.ciphersuites` |
| **Type** | String (comma-separated) |
| **Default** | `""` (Go defaults) |

Comma-separated list of TLS cipher suites to use. Use `--tls-ciphersuites=list` to display available options.

## Logging and Debugging

### Log Level

| | |
|---|---|
| **CLI Flag** | `--log-level` |
| **Environment Variable** | `ARGOCD_AGENT_LOG_LEVEL` |
| **ConfigMap Entry** | `agent.log.level` |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [argoproj-labs/argocd-agent](https://github.com/argoproj-labs/argocd-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
