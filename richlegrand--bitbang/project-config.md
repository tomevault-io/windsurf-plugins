---
trigger: always_on
description: This document defines the canonical vocabulary used across BitBang's code, docs, and protocol descriptions. It exists because the same concepts are referred to by several names in different places (whitepaper, code comments, package names, log lines), and the resulting interpretation tax adds up over time.
---

# BitBang naming conventions

This document defines the canonical vocabulary used across BitBang's code, docs, and protocol descriptions. It exists because the same concepts are referred to by several names in different places (whitepaper, code comments, package names, log lines), and the resulting interpretation tax adds up over time.

Use the **canonical** term in new code and docs. The **discouraged synonyms** are listed so contributors recognize them in older code without picking them up for new work.

---

## The two sides of a session

Every BitBang session involves exactly two endpoints that play distinct roles. The roles are stable for the lifetime of a session; only one side is reached on a given session-establishment.

### Listener

The side that **runs first** and **waits** for a session to be initiated against it. This is the role-level term — use it whenever the *role* matters more than the specific implementation.

- Runs `bitbang serve …`, `bitbang fileshare`, or `bitbang proxy` from the CLI.
- Wraps a WSGI/ASGI app via the `bitbang-python` adapter.
- Is reachable at a URL of the form `https://<server>/<UID>#<access_code>`.
- Holds a private key; performs the listener-side of bidirectional verify.
- Implementations: `bitbangproxy` (Go binary), `bitbang-python` (Python library), Pixy / Goby firmware.

**Discouraged synonyms in new code:** "server" (collides with signaling server), "host" (collides with HTTP/WebRTC host candidates), "service" (overly generic).

### Device

A specific kind of listener — one that runs on **dedicated hardware** rather than on a general-purpose computer. ESP32-class microcontrollers, Pixy / Goby firmware, IoT sensors, embedded cameras. Distinct from a "listener" running on a workstation (`bitbang serve` on someone's laptop is a listener, not a device).

"Device" is the right term whenever the embedded/IoT character is what matters — onboarding flows, power-constrained design, firmware updates, the IoT-platform comparison story in the whitepaper. The whitepaper uses "device" pervasively for this reason.

Reserve "device" for the hardware case. When you mean "the side that accepted the connection regardless of whether it's hardware or software," use "listener" instead. Mixing them up makes the wire-level "/ws/device/<uid>" endpoint name feel arbitrary — it's actually the listener endpoint, with a name that predates the role/implementation split.

**Wire-level legacy:** `/ws/device/<uid>`, the `device_pubkey` field in offers, and `registry.DeviceConn` in the signaling server all use "device" historically. These names are stable and shouldn't churn; just be aware they mean "listener" at the role level. Future renames could clean this up but aren't pressing.

### Connector

The side that **opens** a URL or **types** a pair code and **initiates** the WebRTC handshake. This is the role-level term — use it whenever the *role* matters more than the specific implementation.

- Opens `https://bitba.ng/<UID>#<code>` in a browser.
- Runs `bitbang connect <URL>` or `bitbang connect <6-digit-code>` from the CLI.
- Calls `bitbang cp <URL>:/path /local` from the CLI.
- Sends the `request` or `pair_init` message to the signaling server.
- Performs the connector-side of bidirectional verify.
- Implementations: bootstrap.js (browser runtime), `bitbangproxy` (Go connect/cp/pair flows), future Python connector if added.

**Discouraged synonyms in new code:** "client" (used historically, still appears in some package names like `internal/client/`, but ambiguous because *every* WebSocket connection is technically a client of the signaling server), "peer" (acceptable inside WebRTC contexts but vague at the application level).

### Browser

A specific kind of connector — one that runs as **JavaScript inside a web browser** rather than as a native CLI process. Loads bootstrap.js from the signaling server, exposes the proxied app inside an iframe, runs WebRTC in the browser's native stack.

"Browser" is the right term whenever the in-browser character is what matters — UI affordances, SubtleCrypto, getStats() shape, the "no install on the viewing side" property, the `/<UID>` URL flow as opened from a clickable link. The README's user-facing copy says "browser" pervasively for this reason.

Reserve "browser" for the in-browser case. When you mean "the side that initiated the connection regardless of whether it's a browser or a CLI," use "connector" instead. The CLI's `bitbang connect` does most of the same work the browser does (offer/answer, ICE trickle, bidirectional verify, `connection_path` telemetry) — calling it "the browser" in that context obscures the CLI case.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [richlegrand/bitbang](https://github.com/richlegrand/bitbang) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
