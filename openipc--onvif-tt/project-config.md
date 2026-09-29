---
trigger: always_on
description: This file is a cold-start brief for AI agents collaborating on this
---

# Working on `onvif-tt` with Claude (or any AI agent)

This file is a cold-start brief for AI agents collaborating on this
repo. It explains what the project is, how its pieces fit together,
and the conventions that — if you get them wrong — produce subtle
bugs that look right but aren't.

## What this is

`onvif-tt` is a Linux-native, headless ONVIF conformance test tool —
an open-source alternative to the closed (members-only, Windows-only)
ONVIF Device Test Tool. It:

1. Parses the public ONVIF Test Specification HTML corpus into a
   structured catalog (`corpus/parsed.json`, 1,121 test cases).
2. Lets contributors register Python implementations of individual
   spec IDs via `@register("DEVICE-1-1-2")` decorators.
3. Runs the registered tests against a Device Under Test via pytest,
   emitting JUnit XML for CI and a structured JSON report whose schema
   is published in `docs/schemas/`.

## Repo layout

```
onvif-tt/
├── corpus/
│   ├── html/                 # 22 ONVIF Test Specification files (verbatim, redistributable)
│   └── parsed.json           # generated cache; deterministic-parse-tested
├── src/onvif_tt/
│   ├── specs/                # lxml DocBook parser + dataclasses
│   ├── registry.py           # @register decorator + REGISTRY dict + xfail_on matcher
│   ├── runtime/
│   │   ├── dut.py            # ONVIFCamera wrapper, lazy service binding,
│   │   │                     # PullPointHandle, NotifyHandle, SOAP trace
│   │   ├── features.py       # cached GetServices + GetDeviceInformation
│   │   ├── discovery.py      # multicast WS-Discovery Probe helper
│   │   └── soap_trace.py     # zeep plugin that captures envelopes
│   ├── runner/
│   │   ├── dispatch.py       # pytest parametrise — one node per registry entry
│   │   └── plugin.py         # CLI options + JSON reporter (xdist-safe)
│   ├── cases/                # one .py per profile area
│   │   ├── base.py           # Device + capabilities + GetServices flavours
│   │   ├── auth.py           # LOCAL-AUTH-* (no AUTH-* in catalog)
│   │   ├── discovery.py      # WS-Discovery
│   │   ├── ipconfig.py       # LOCAL-NETWORK-* read-only (catalog IPCONFIG-* are writes)
│   │   ├── media.py          # Media v10 + Media2 (adaptive) + consistency tests
│   │   ├── event.py          # GetEventProperties + PullPoint + Basic Notification
│   │   ├── ptz.py            # GetNodes + Move/Stop write ops
│   │   └── imaging.py        # GetImagingSettings + Move/Stop write ops
│   └── cli.py                # argparse: `onvif-tt list|show|corpus|run`
├── tests/                    # Tests for the tool itself (parser + registry)
├── docs/
│   ├── ai-readme.md          # How to drive the tool from an LLM tool-use loop
│   ├── adding-a-test.md      # Contributor guide
│   └── schemas/              # JSON Schema for results.json + corpus/parsed.json
└── scripts/                  # Shell helpers (subscription-cleanup verifier, …)
```

## Conventions that matter

These are mistakes that bit us during development. If you reproduce
them, the tests will pass in spurious ways.

### 1. Look up real test IDs via the catalog before `@register`-ing

The spec ID `DEVICE-1-1-2` is "ALL CAPABILITIES", not "GetDeviceInformation"
(that's `DEVICE-3-1-9`). The names are not as obvious as they look.

```bash
onvif-tt show DEVICE-3-1-9    # prints the actual procedure
onvif-tt list --id-glob "MEDIA2-2-*" --missing   # what's left to implement
```

If you invent an ID that isn't in the catalog, the CI check
"every registered spec ID exists in the catalog" will fail.

### 2. `python-onvif-zeep` takes **positional** WSDL parameters

```python
dut.devicemgmt.GetServices(False)                       # ✅
dut.devicemgmt.GetServices(IncludeCapability=False)     # ❌ raises
dut.devicemgmt.GetCapabilities("All")                   # ✅
dut.devicemgmt.GetCapabilities(Category="All")          # ❌
```

For operations with structured arguments, build with `create_type`:

```python
req = dut.media.create_type("GetStreamUri")
req.StreamSetup = {"Stream": "RTP-Unicast", "Transport": {"Protocol": "RTSP"}}
req.ProfileToken = profile.token
resp = dut.media.GetStreamUri(req)
```

### 3. ONVIF "void" responses come back as Python `None`

Operations like `Move`, `Stop`, `AbsoluteMove`, `RelativeMove`,
`Unsubscribe`, `SetSynchronizationPoint`, … have empty response bodies
in the WSDL — zeep returns `None`. Never assert `resp is not None` on
these; the assertion is "no SOAP Fault was raised".

```python
dut.imaging.Move(req)        # ✅ just the call
resp = dut.imaging.Move(req)
assert resp is not None      # ❌ will FAIL spuriously on a perfectly conformant device
```

### 4. PullPoint subscriptions need **two** WSDL bindings

The PullPoint subscription URL is queried via different bindings for
different operations:

* `PullPointSubscriptionBinding` → `PullMessages`, `SetSynchronizationPoint`
* `SubscriptionManagerBinding` → `Renew`, `Unsubscribe`

`PullPointHandle` (and `NotifyHandle` for Basic Notification) in
`runtime/dut.py` already wraps both. Use them; don't roll your own.

```python
with dut.create_pullpoint("PT60S") as pp:
    pp.pull_messages(timeout="PT3S", limit=5)
    # auto-unsubscribe on __exit__
```

### 5. `LOCAL-*` prefix for tool-author IDs


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [OpenIPC/onvif-tt](https://github.com/OpenIPC/onvif-tt) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-29 -->
