---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`iobroker.mihome` is an ioBroker adapter for the **Xiaomi Mi Home / Aqara gateway** (Mi Control Hub, also `acpartner.v3`). It talks the gateway's local UDP protocol, creates one channel per device behind the gateway and mirrors the device values into ioBroker states. Writable states (gateway light, plugs, wall switches, relays, curtains, AC partner) are sent back to the gateway.

TypeScript (CommonJS output). Sources live in `src/`, the published/runnable code is the compiled `build/` (`package.json` `main` is `build/main.js`). `build/` is gitignored — always run the build before starting the adapter or the integration tests.

## Commands

```bash
npm run build                             # tsc -p tsconfig.build.json  -> build/
npm run watch                             # same in watch mode
npm run check                             # type check only (tsconfig.json, noEmit)
npm run lint                              # eslint (@iobroker/eslint-config, flat config)
npx eslint -c eslint.config.mjs --fix src # autofix + prettier formatting

npm run test:package                      # validates package.json / io-package.json / admin JSON (fast)
npm run test:integration                  # starts a real js-controller + adapter + the gateway simulator
npm run translate                         # translate-adapter -b admin/i18n/en.json
npm run release-patch                     # @alcalzone/release-script, moves README changelog into io-package news
```

There is deliberately **no `prepare` script** — `npm ci`/`npm install` does not build. Run `npm run build` yourself after a fresh checkout and before starting the adapter or the tests. Because `build/` is neither committed nor built on install, `common.nogit` is `true` in `io-package.json`: the adapter can only be installed from npm, not from GitHub. The integration test requires that **no** js-controller is running on the machine, otherwise it aborts with "JS-Controller is already running!".

## Architecture

### Layout

| Path | Content |
| --- | --- |
| `src/main.ts` | the adapter: one `MihomeAdapter extends utils.Adapter` class |
| `src/lib/Hub.ts` | `Hub` — the UDP transport, sensor registry and gateway discovery |
| `src/lib/devices.ts` | `Devices` — the catalog: model name → class + the states to create |
| `src/lib/Sensors/Sensor.ts` | abstract base class of all devices |
| `src/lib/Sensors/*.ts` | one class per device family |
| `src/lib/types.ts` | protocol and catalog types |
| `src/lib/adapter-config.d.ts` | augments `ioBroker.AdapterConfig` |
| `admin/jsonConfig.json` | the configuration dialog |
| `admin/icons/*.png` | per-device icons, referenced as `/icons/<type>.png` in `common.icon` |
| `test/lib/gateway.js` | gateway simulator used by the integration test |

`src/lib/adapter-config.d.ts` is hand-maintained and must be kept in sync with `native` in `io-package.json` **and** with `admin/jsonConfig.json` — nothing generates it. All three currently hold exactly: `bind`, `port`, `key`, `keys`, `sids`, `interval`, `heartbeatTimeout`, `restartInterval`, `mmHg`.

### Transport: `src/lib/Hub.ts`

One UDP4 socket bound to `config.port` (9898). On `listening` it joins the multicast group `224.0.0.50` (on `config.bind` if that is not `0.0.0.0`) and sends `{"cmd": "whois"}` to `224.0.0.50:4321`. Every gateway answers with its own messages; commands are sent back to the gateway IP on `config.port`.

- `sensorFactory()` looks the reported `model` up in `Devices` (by `type`) and constructs the matching class. An unknown model throws and is reported as a `warning` — the device is then simply ignored.
- Devices that report **no** model (Aqara 2-channel relay, firmware bug) get their model from `config.sids` (`sid` → `model`).
- Gateways with a sid shorter than 12 characters get leading zeros (`msg.sid` fix in `onSocketMessage`).
- **Protocol 2.0.x** is normalized to 1.0.x on the way in: `params` (list of single-attribute objects) → `data`, `read_rsp`/`write_rsp` → `*_ack`, `discovery_rsp` → `get_id_list_ack` with the sid list as `data`. `protoMajor(ip)` decides, based on the `proto_version` the gateway last reported. On the way out `sendMessage()` does the reverse: `short_id` is dropped, `data.key` is lifted to the top level and `data` becomes `params`.
- `getKey(ip)` encrypts the **last token** of that gateway with its key (`aes-128-cbc`, fixed IV) — that ciphertext is the `key` field every `write` command needs. Without a token (no heartbeat yet) it returns `null` and writes are rejected by the gateway.
- Events: `message` (every parsed packet, also drives the connection watchdog), `data` (sid, className, values), `device` (new sensor), `browse` (discovery), `debug`, `warning`, `error`. `error` makes the adapter restart the hub, so only real connection failures may use it.

### Devices: `src/lib/devices.ts` and `src/lib/Sensors/`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ioBroker/ioBroker.mihome](https://github.com/ioBroker/ioBroker.mihome) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
