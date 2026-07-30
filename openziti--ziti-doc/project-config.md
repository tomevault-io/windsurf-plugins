---
trigger: always_on
description: The following conventions apply to multiple areas of the configuration files for routers and
---


# Conventions

The following conventions apply to multiple areas of the configuration files for routers and
controllers.

## Addressing

Listening and dialing addresses in OpenZiti are in the format of `<protocol>:<ip-or-host>:<port>`
format.

For servers that are listening `<ip-or-host>` should be the address of an interface to listen on
or "0.0.0.0" for all IPv4 interfaces or "::" for all IPv6 interfaces. `<port>` should be a valid
port number that the server should listen on.

For clients dialing a server the `<ip-or-host>` should be an IP address or hostname that resolves to
the target server. `<port>` should be the port the server is listening on.

For clients and server, the `<protocol>` section is the protocol used to host or initiate the
connection. It may be one of the following values, however `tls` is suggested for most scenarios.

- `tls`
- `tcp`
- `udp`
- `dtls`
- `ws`
- `wss`
- `transwarp`
- `transwarptls`

## Environment Variables

All values in the configuration file support environment variable replacement. The environment
variables are sourced from the scope of the executing process (i.e. controller, router). The
syntax `${VARIABLE}` is used.

Example:

```text
db: ${ZITI_DATA}/db/ctrl.db
```

## Identity

OpenZiti uses a common framework for loading, storing, and processing certificate and private key
configuration. Identity sections all have a similar format. The use of the defined certificates is
up to the implementing application. So see their configuration sections for details on which values
are utilized for what. This documentation provides an overview useful to understand the "default"
assumptions. The `identity` section may need to be formatted as YAML or JSON, depending on the
implementing application. The `ziti` CLI and bundled sub-commands,
e.g. `ziti controller`, `ziti router`, expect a configuration file formatted as YAML.

- `cert` - (required) A string in the format of `<engine>:<value>` that defines a x509 client
  certificate
- `key` - (required) A string in the format of `<engine>:<value>` that defines a private key used
  for `cert`
  and `server_cert`
  if `server_key` is not defined
- `server_cert` -(optional) A string in the format of `<engine>:<value>` that defines a x509
  certificate, if not defined `cert` is used
- `server_key` - (optional) A string in the format of `<engine>:<value>` that defines a private key
  for `server_cert`, if not defined `key` is used if `server_cert` is defined
- `ca` - (optional) A string in the format of `<engine>:<value>` that defines x509 certificate chain
  used to define trusted CAs
- `alt_server_certs` - (optional) An array of objects with `server_cert` and `server_key` values
  used to add additional server certificates and keys not managed by OpenZiti (i.e. from public CAs
  like Let's Encrypt). The server name indication (SNI) of incoming requests is compared with all
  DNS subject alternative names (SAN) of the server certificates in `identity.server_cert`
  and `identity.alt_server_certs[].server_cert`. The first-matched server certificate is presented
  to the TLS client. You must use distinct DNS SANs to avoid ambiguity during server certificate
  selection.

The `<engine>:<value>` format is used to define multiple different source types. If the `<engine>:`
part is omitted, it is assumed to be `file:`. The following engines are supported:

- `file` - indicates that `<value>` is the path to a file
- `pem` - indicates that `<value>` is an inline PEM string

Additional PKCS#11 engines such as `siometrics.so` and `authenta.so` may be used if the library are
present on the host machine. This allows for access to hardware backed private keys.

Example Identity Section (Client & Server use same key):

```text
identity:
  cert: "file:ctrl-client.cert.pem"
  server_cert: "pem:-----BEGIN CERTIFICATE-----\nMIIEtzCCAp+gAwIBAgICEA0wDQYJKoZIhvcNAQELBQAwgYsxCzAJBgNVBAYTAlVT..."
  key: ctrl.key.pem
  ca: ca-chain.cert.pem
  alt_server_certs:
    - server_cert: lets_encrypt.cert.pem
      server_key: lets_encrypt.key.pem

```

## Channel

Channel sections control different ways in which connections behave. It is controlled by the
[channel](https://github.com/openziti/channel) library. Sections that invoke the channel library
support the following options section.

- `options` - a set of optional connections options
    - `maxQueuedConnects` - (optional) the maximum number of connections to be accepted but awaiting
      initial messaging
    - `maxOutstandingConnects` - (optional) the maximum number of connection accepted and waiting
      for hello messaging to complete
    - `connectTimeoutMs` - (optional) the maximum number of milliseconds to wait for hello messaging
      to complete
        - `writeTimeout` - (optional)  the maximum amount of time to wait when writing data to a
          connection

## Heartbeats

Channels will use heartbeats to determine both latency and health. Heartbeats can be configured with
the following properties.

- `sendInterval` - (optional, 10s) How often to send heartbeats
- `checkInterval` - (optional, 1s) How often to check if heartbeat thresholds have been met
- `closeUnresponsiveTimeout` - (optional, 30s) How to long to wait after the last heartbeat response

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [openziti/ziti-doc](https://github.com/openziti/ziti-doc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
