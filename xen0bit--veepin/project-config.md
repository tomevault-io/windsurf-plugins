---
trigger: always_on
description: generates one flag per spec — name from `Key` (or `Flag`, for the handful
---

# veepin — working notes for coding agents

A from-scratch userspace VPN in pure Go. Sixteen production protocols, **client
and server for every one**, each verified in Docker against a real third-party
implementation *and* against itself.

That sentence is the whole thesis, and it is what most decisions here follow
from. Read it as three constraints:

- **From scratch.** No third-party protocol libraries. Dependencies are
  `golang.org/x/{crypto,net,sys}` and nothing else. `x/crypto` is there because
  WireGuard mandates ChaCha20-Poly1305 and BLAKE2s; `x/net` for QUIC.
- **Both roles.** A protocol is not "added" until `veepin connect <p>` and
  `veepin serve <p>` both work. Half a protocol is not a milestone.
- **Verified against a real peer.** A veepin↔veepin test proves the two halves
  agree with each other, which is not the same as being right. See
  [The interop matrix earns its keep](#the-interop-matrix-earns-its-keep).

## Orientation

```
cmd/veepin/          the CLI: connect, serve, probe
client/              the registry, and the Session/Result/Server contracts
dataplane/           TUN, address pool, packet pump, routing, shaping — protocol-agnostic
internal/cryptoutil/ the primitives — protocol-agnostic
<proto>/             the public facade for one protocol: Config, Opt* consts, Dial, NewServer
internal/<proto>/    that protocol's implementation
tests/interop/       Docker cells: one compose file per direction, per protocol
internal/livingreadme/  the interop matrix, which drives both the README tables and the CI shards
nm/                  a separate Go module: the NetworkManager plugin
```

Two shared packages carry the PPP protocols: `internal/ppp` (LCP, MS-CHAPv2,
IPCP, both roles) and `internal/mschap`. Two carry IPsec: `internal/ikev2/esp`
(the RFC 4303 data path, used by five protocols now) and `internal/ikev1` (which
serves both L2TP/IPsec and Cisco IPsec).

`toy` / `internal/toy` is a deliberately insecure worked example with a written
spec. It is the right thing to read first when adding a protocol, and it must
never carry traffic.

## Hard rules

These are contracts, not conventions. Breaking one is silent at compile time and
wrong at runtime.

- **`Dial` installs no routes and no addresses.** It returns a `client.Result`
  the caller applies. See `client/client.go`.
- **`client.Result.Gateway` is the server's OUTER address** — the one dialled on
  the underlying network, never an address inside the tunnel. It exists so the
  caller can pin a host route and stop the tunnel's own packets recursing into
  it. Getting this wrong is silent: the handshake succeeds, the interface comes
  up, and every packet leaves by the wrong door. `client.Result.Validate`
  catches the common mistake.
- **`NewServer` opens the TUN and validates, but binds nothing.** Sockets are
  bound in `ListenAndServe`, so the caller can configure host networking first.
- **Parsers return subslices of their input.** The inbound data path is
  allocation-free by design; a parser that copies costs one allocation per
  packet. `datapath_test.go` in each package pins this.
- **`internal/` is where implementations live.** The `<proto>/` package is the
  supported surface and should be thin.

## The mechanical guards

CI fails loudly and by name if you skip a step. Knowing these up front saves a
round trip:

| Guard | What it requires |
|---|---|
| `docs_test.go` — `TestPackageDocNamesEveryProtocol` | `doc.go`'s package comment names every registered package |
| `docs_test.go` — `TestREADMECountsProtocolsCorrectly` | **every** occurrence of "*N* production protocols" and "*Nth* registered protocol" in the README agrees with the registry — spelled out ("sixteen", "seventeenth") |
| `fuzztargets_test.go` — `TestFuzzTargetsAreAllListed` | every `Fuzz*` in the tree is in the `TARGETS` heredoc in `.github/workflows/ci.yml`, and `expected=N` matches the count |
| `cmd/veepin/main_test.go` | every registered protocol has a `connect` case |
| `cmd/veepin/flags_test.go` — `TestTheFlagSetIsTheSpecTable` | every registered protocol declares `RegisterClientOpts`/`RegisterServerOpts` for each role it claims, and every spec in them produces a flag that reaches its own key. Two specs cannot claim one flag spelling |
| `cmd/veepin/flags_test.go` | every bound flag reaches the option map (it perturbs each one and requires the map to change); every emitted key has a matching `Opt*` const |
| `cmd/veepin/flags_test.go` — `TestRequiredClientOptsAreTheOnesTheParseRejects` | an option whose absence the parse rejects with "is required" is marked `Required: true` — **and** that the full option map built from the specs parses at all, without which the check is vacuous for that protocol |
| `cmd/veepin/flags_test.go` — `TestSecretFlagsAgreeAcrossBothTables` | a key in both a protocol's client and server tables carries the same `Secret` flag in each |
| `autherr_test.go` — `TestEveryProtocolJudgingACredentialReportsErrAuth` | every facade declaring a `Secret` client option references `client.ErrAuth`/`client.WrapAuth`, or is named in `noCredentialJudged` with the reason its `Dial` judges no credential |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xen0bit/veepin](https://github.com/xen0bit/veepin) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-23 -->
