---
trigger: always_on
description: Handles JSON bodies, retries on 429 with `retry-after`, maps statuses to
---

# CLAUDE.md

Notes for Claude (and future contributors) working on `dns-update`.

## What this crate is

`dns-update` is an async Rust library for dynamic DNS updates. It exposes a
single enum `DnsUpdater` that fronts many backend providers behind a uniform
`create` / `update` / `delete` API for record types `A`, `AAAA`, `CNAME`, `NS`,
`MX`, `TXT`, `SRV`, `TLSA`, `CAA`. It targets RFC 2136 (with TSIG) and a
growing list of cloud / registrar DNS APIs. The stated goal in `README.md` is
to eventually cover as many providers as Go's
[lego](https://go-acme.github.io/lego/dns/) library.

## Layout

- `src/lib.rs`: public surface. Defines `Error`, `DnsRecord*`, `CAARecord`,
  `MXRecord`, `SRVRecord`, `TLSARecord`, `KeyValue`, `TsigAlgorithm`,
  `Algorithm`, `IntoFqdn`, and the `DnsUpdater` enum that dispatches to a
  provider.
- `src/update.rs`: the `impl DnsUpdater` factories (`new_*`) and the
  `create` / `update` / `delete` match arms that fan out to each provider.
- `src/providers/`: one file per provider. Each file is self-contained and
  exposes a `pub struct <Name>Provider` plus `pub(crate) async fn new`,
  `create`, `update`, `delete` methods called from `update.rs`.
- `src/tests/`: one `*_tests.rs` file per provider, registered in
  `src/tests/mod.rs`.
- `src/http.rs`: shared `HttpClientBuilder` / `HttpClient` over `reqwest`.
  Handles JSON bodies, retries on 429 with `retry-after`, maps statuses to
  `Error::Unauthorized` / `Error::NotFound` / `Error::BadRequest` /
  `Error::Api`.
- `src/utils.rs`: `strip_origin_from_name`, `txt_chunks*`, `IntoFqdn` impls,
  `Display` for record types, helpers like `CAARecord::decompose`. Reuse
  these.
- `src/crypto.rs`, `src/jwt.rs`: HMAC, SHA, RSA-via-`ring`/`aws-lc-rs`, and
  Google-style JWT helpers (used by route53, ovh, google_cloud_dns).
- `src/bind.rs`: BIND zone-file rendering. Not a provider.

## Adding a new provider (the recipe)

For a vanilla token-based REST provider, follow `digitalocean.rs` or
`desec.rs` as the template. Steps:

1. Create `src/providers/<name>.rs` defining:
   - `#[derive(Clone)] pub struct <Name>Provider { client: HttpClientBuilder, ... }`
   - `pub(crate) fn new(...) -> crate::Result<Self>` (or `-> Self` if no
     fallible setup). Inject auth headers via
     `HttpClientBuilder::default().with_header(...).with_timeout(timeout)`.
   - `pub(crate) async fn create / update / delete` with the same signatures
     used elsewhere:
     ```rust
     async fn create(&self, name: impl IntoFqdn<'_>, record: DnsRecord, ttl: u32, origin: impl IntoFqdn<'_>) -> crate::Result<()>
     async fn update(&self, name: impl IntoFqdn<'_>, record: DnsRecord, ttl: u32, origin: impl IntoFqdn<'_>) -> crate::Result<()>
     async fn delete(&self, name: impl IntoFqdn<'_>, origin: impl IntoFqdn<'_>, record_type: DnsRecordType) -> crate::Result<()>
     ```
   - A test-only `with_endpoint` setter (gated `#[cfg(test)]`) so mockito can
     point the provider at a local URL.
2. Register it in `src/providers/mod.rs` (`pub mod <name>;`).
3. Wire it into `src/lib.rs`: add a variant on `DnsUpdater` and import the
   provider type.
4. Wire it into `src/update.rs`:
   - Add a `new_<name>(...)` constructor.
   - Add match arms in `create` / `update` / `delete`.
5. Write `src/tests/<name>_tests.rs` and register the module in
   `src/tests/mod.rs`. Use `mockito::Server::new_async()` for unit tests; the
   existing `cloudflare_tests.rs` is the canonical example. Also include a
   `#[ignore = "..."]` integration test that hits the real API behind env
   vars, again like cloudflare.

### Conventions

- **No comments in source code.** Identifiers carry the *what*; rationale
  belongs in commit / PR messages.
- **Never use em dashes (`—`).** Use `:`, `;`, parens, comma, or sentence
  breaks.
- **Naming**: subdomain vs name vs fqdn distinction matters. Use
  `name.into_name()` and `origin.into_name()` from `IntoFqdn`. To get the
  bare subdomain relative to a zone use
  `utils::strip_origin_from_name(name, origin, Some(""))` (or `None` for the
  `@` default).
- **Zone discovery**: when the API exposes zones by name, walk up the
  origin one label at a time until a zone matches (see Cloudflare's
  `obtain_zone_id`). For providers that always require an explicit zone,
  trust the caller's `origin`.
- **Record-ID resolution**: providers that need an ID for update / delete
  usually do a list-records lookup filtered by name + type. Reuse the
  Cloudflare and DigitalOcean patterns. For update specifically, resolve
  the record ID before issuing PATCH / PUT (see issue #52 / commit 707b82c).
- **TXT quoting / chunking**: some APIs want raw text, others want quoted
  with `\"`-escaping, others want chunks of <=255 bytes. Reuse
  `utils::txt_chunks_to_text` / `utils::txt_chunks`. Be deliberate about
  which form you send; check the provider's docs and look at lego's
  `presentRecord` for a hint.
- **CAA**: convert via `CAARecord::decompose()` to `(flags, tag, value)`.
  Many APIs accept that flat form; others want the BIND-style string given
  by `CAARecord::Display`.
- **TLSA**: `TLSARecord::Display` produces `"<usage> <selector> <matching>
  <hex>"`. Many providers can take that directly. Some don't support TLSA at

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [stalwartlabs/dns-update](https://github.com/stalwartlabs/dns-update) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
