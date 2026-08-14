---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Repository status

**v0.9.x shipped — see CHANGELOG for the current patch.** Full fiscal lifecycle covered:

- **v0.1** — `fetchByChave`, `fetchByNsu`, parser RTC v1.01.
- **v0.2** — `emitirDpsPronta` / `emitirEmLote`, dry-run, XMLDSig, XSD WASM, CPF/CNPJ DV, ViaCEP, `buildDps`.
- **v0.3** — `cancelar` (101101) + `substituir` (105102) with 5-state compensation machine (`ok` / `retry_pending` / `rolled_back` / `rollback_pending` / `rollback_failed`), pluggable `RetryStore`, `replayPendingEvents`.
- **v0.4** — safe emit flow: `emitir(params)` with `DpsCounter`. Counter only consumes after offline validations pass; transient errors go to `RetryStore` instead of throwing. Old API preserved as `emitirDpsPronta(dps)` escape hatch.
- **v0.5** — 6 `consultar*` methods against ADN `/parametrizacao`, with pluggable `ParametrosCache` (default in-memory + TTL).
- **v0.6** — `NfseClientFake` in `open-nfse/testing` subpath, structurally compatible via `NfseClientLike`.
- **v0.7** — DANFSe PDF: `gerarDanfse(nfse, options)` with strategy `'auto' | 'online' | 'local'` (default `'auto'` = ADN online + fallback to local pdfkit renderer); `fetchDanfse(chave)` online-only.
- **v0.8.0** — 429 handling: typed `TooManyRequestsError`, classified as transient, persisted in `RetryStore` with `notBefore` (from `Retry-After`) and `attempts`. Pluggable `RetryPolicy` (interface + `createDefaultRetryPolicy`); the lib wraps every configured policy via internal `makeSafePolicy` so a buggy custom policy can't mask the original fiscal error. **Fixed** a critical pre-existing bug from v0.7.2: events persisted to `RetryStore` were unsigned (replay rejected by SEFIN); legacy data rescued automatically.
- **v0.8.1–0.8.6** — XSD/serialization conformance hardening (enums, types), `buildDps` business-rule guards, and **alignment with Anexo II SEFIN_ADN v1.00-20251226**: the `infPedReg` `Id` dropped `nPedRegEvento` (now `PRE` + chave(50) + tipoEvento(6) = 59 chars, `PRE[0-9]{56}`), the `<nPedRegEvento>` element was removed from the event body, and event dedup is now `(chave, tipoEvento)`. `nPedRegEvento` no longer exists anywhere in the API — **do not re-introduce it.** See CHANGELOG for the full per-patch list.

Docs site: VitePress + TypeDoc → https://fm-s.github.io/open-nfse/. Roadmap ahead: stabilization until 1.0 — public API may still receive tweaks. Details per-version in CHANGELOG.

## Commands

```
npm test             # run vitest once
npm run test:watch   # vitest in watch
npm run test:coverage
npm run typecheck    # tsc --noEmit on the whole tree (tests included)
npm run build        # tsc -p tsconfig.build.json → emits dist/ without tests
npm run lint         # biome check
npm run lint:fix     # biome check --write (safe fixes; some rules need --unsafe)
```

`prepublishOnly` chains lint + typecheck + tests + clean + build. Don't publish without it passing.

- Single test file: `npx vitest run src/nfse/parse-xml.test.ts`
- Single test by name: `npx vitest run -t 'parses exterior tomador'`

## What this library is

TypeScript/Node client for **NFS-e Padrão Nacional** (nfse.gov.br) — the unified Brazilian service-invoice API operated by the Receita Federal, sole standard from **2026-01-01** (LC 214/2025). Talks directly to the official API; not a wrapper over a commercial gateway.

**Canonical Portuguese domain terms** — do not translate:

- **DPS** — Declaração de Prestação de Serviços (what the emitente submits)
- **NFS-e** — the authorized document returned by the Receita
- **DF-e** — Documentos Fiscais Eletrônicos (umbrella for distribution by NSU)
- **NSU** — Número Sequencial Único (cursor for incremental DF-e sync, per CPF/CNPJ)
- **DANFSe** — PDF representation of an NFS-e
- **IBS / CBS / NBS / cClassTrib** — Reforma Tributária tax codes
- **Sefin Nacional** — federal emission endpoint host; **ADN** (Ambiente de Dados Nacional) — federal distribution endpoint host

## Architecture — actual shipped shape

The API is split across **two base URLs**, not one:

| Service | Path on `Ambiente` | Endpoints used |
|---|---|---|
| **SEFIN Nacional** | `endpoints.sefin` | `POST /nfse` ✓, `GET /nfse/{chave}` ✓, events on `/nfse/{chave}/eventos` ✓, `GET/HEAD /dps/{id}` ✓ (v0.7.2). **Out of scope:** `POST /decisao-judicial/nfse` (backs the Emissor Público Web UI per Guia v1.2 §4.3, not a contribuinte API); `GET /nfse/{chave}/eventos/{tipoEvento}/{numSeqEvento}` (backlog). |
| **ADN Contribuintes** | `endpoints.adn` | `GET /DFe/{NSU}` ✓, `GET /NFSe/{ChaveAcesso}/Eventos` (not yet wrapped) |
| **ADN DANFSe** | `endpoints.danfse` | `GET /{chaveAcesso}` → PDF ✓ |
| **ADN Parâmetros Municipais** | `endpoints.parametrosMunicipais` | `GET /aliquotas/{cMun}/{cServ}/{competencia}` + 5 more ✓ |

Crucially, **SEFIN uses camelCase + int `tipoAmbiente`** while **ADN uses PascalCase + string `TipoAmbiente`** — wire-format types stay private per module and the public DTOs normalize to a single convention. Don't ever "unify" the wire formats at the HTTP layer; they're genuinely different contracts.

```
┌────────────────────────────────────────────────────────────────┐

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fm-s/open-nfse](https://github.com/Fm-s/open-nfse) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-13 -->
