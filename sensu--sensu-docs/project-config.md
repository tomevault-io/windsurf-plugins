---
trigger: always_on
description: Read this reference to learn how Sensu's agent runs as a lightweight client on infrastructure components you want to monitor and creates observability events.
---


[Example Sensu agent configuration file](../../../files/agent.yml) (download)

The Sensu agent is a lightweight client that runs on the infrastructure components you want to monitor.
Agents register with the Sensu backend as [monitoring entities][3] with `type: "agent"`.
Agent entities are responsible for creating [check and metrics events][7] to send to the [backend event pipeline][2].

The Sensu agent is available for Linux, macOS, and Windows.
For Windows operating systems, the Sensu agent uses `cmd.exe` for the execution environment.
For all other operating systems, the Sensu agent uses the Bourne shell (sh).

Read the [installation guide][1] to install the agent.

## Agent authentication

The Sensu agent authenticates to the Sensu backend via [WebSocket][45] transport by either built-in basic authentication (username and password) or mutual transport layer security (mTLS) authentication.

### Username and password authentication

The default mechanism for agent authentication is [built-in basic authentication][59] with username and password.
The Sensu agent uses username and password authentication unless mTLS authentication has been explicitly configured.

For username and password authentication, sensu-agent joins the username and password with a colon and encodes them as a Base64 value.
Sensu provides the encoded string as the value of the `Authorization` HTTP header &mdash; for example, `Authorization: Basic YWdlbnQ6UEBzc3cwcmQh` &mdash; to authenticate to the Sensu backend.

When using username and password authentication, sensu-agent also sends the following HTTP headers in requests to the backend:

- `Sensu-User`: the username in plaintext
- `Sensu-AgentName`: the agent's configured name in plaintext
- `Sensu-Subscriptions`: the agent's subscriptions in a comma-separated plaintext list
- `Sensu-Namespace`: the agent's configured namespace in plaintext

### mTLS authentication

When mTLS is configured for both the Sensu agent and backend, the agent uses mTLS authentication instead of the default username and password authentication.

Sensu backends that are configured for mTLS authentication will no longer accept agent authentication via username and password.
Agents that are configured to use mTLS authentication cannot authenticate with the backend unless the backend is configured for mTLS.

To configure the agent and backend for mTLS authentication:

- In the backend configuration, specify valid certificate and key files as values for the `agent-auth-cert-file` and `agent-auth-key-file` parameters (e.g. `backend-1.pem` and `backend-1-key.pem`, respectively).
- In the agent configuration, specify valid certificate and key files as values for the `cert-file` and `key-file` parameters (e.g. `agent.pem` and `agent-key.pem`, respectively).

{{% notice note %}}
**NOTE**: For detailed information about the certificates and keys required for mTLS authentication, read [Generate certificates for your Sensu installation](../../../operations/deploy-sensu/generate-certificates/).
For information about using the certificates and keys to secure your configuration, read [Secure Sensu](../../../operations/deploy-sensu/secure-sensu/). 
{{% /notice %}}

The agent and backend will compare the provided certificates with the trusted CA certificate either in the system trust store or specified explicitly as the `agent-auth-trusted-ca-file` in the backend configuration and `trusted-ca-file` in the agent configuration.

When using mTLS authentication, sensu-agent sends the following HTTP headers in requests to the backend:

- `Sensu-AgentName`: the agent's configured name in plaintext
- `Sensu-Subscriptions`: the agent's subscriptions in a comma-separated, plaintext list
- `Sensu-Namespace`: the agent's configured namespace in plaintext

If the Sensu agent is configured for mTLS authentication, it will not send the `Authorization` HTTP header.

#### Certificate revocation check

The Sensu backend checks certificate revocation list (CRL) and Online Certificate Status Protocol (OCSP) endpoints for agent mTLS, etcd client, and etcd peer connections whose remote sides present X.509 certificates that provide CRL and OCSP revocation information.

## Communication between the agent and backend

The Sensu agent uses [WebSocket][45] (ws) protocol to send and receive JSON messages with the Sensu backend.
For optimal network throughput, agents will attempt to negotiate the use of [Protobuf][47] serialization when communicating with a Sensu backend that supports it.
This communication is via clear text by default.

Follow [Secure Sensu][46] to configure the backend and agent for WebSocket Secure (wss) encrypted communication.

{{% notice note %}}
**NOTE**: For information about your agent transport status, use the [/health API](../../../api/other/health/#get-health-data-for-your-agent-transport).
{{% /notice %}}

## Create observability events using service checks

Sensu uses the [publish/subscribe pattern of communication][15], which allows automated registration and deregistration of ephemeral systems.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [sensu/sensu-docs](https://github.com/sensu/sensu-docs) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
