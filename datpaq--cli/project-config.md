---
trigger: always_on
description: Printing Press CLI for Datpaq. Comprehensive multi-service API platform providing aviation, geolocation, financial, utilities, image processing,...
---


# Datpaq — Printing Press CLI

## Prerequisites: Install the CLI

This skill drives the `datpaq` binary. **You must verify the CLI is installed before invoking any command from this skill.** If it is missing, install it first:

1. Install via the Printing Press installer:
   ```bash
   npx -y @mvanhorn/printing-press install datpaq --cli-only
   ```
2. Verify: `datpaq --version`
3. Ensure `$GOPATH/bin` (or `$HOME/go/bin`) is on `$PATH`.

If the `npx` install fails before this CLI has a public-library category, install Node or use the category-specific Go fallback after publish.

If `--version` reports "command not found" after install, the install step did not put the binary on `$PATH`. Do not proceed with skill commands until verification succeeds.

Comprehensive multi-service API platform providing aviation, geolocation, financial, utilities, image processing, and data enrichment capabilities. All endpoints require authentication via API key passed as an x-api-key header or api_key query parameter.

## Command Reference

**aircraft** — FAA aircraft registry lookups by tail number, ICAO hex, or type code

- `datpaq aircraft batch-lookup` — Batch aircraft lookups by tail number
- `datpaq aircraft lookup-by-icao` — Look up aircraft by ICAO hex code
- `datpaq aircraft lookup-by-tail` — Returns aircraft ownership, manufacturer, model, type, and year from the FAA registry using an N-number (tail number).
- `datpaq aircraft lookup-by-type` — Look up aircraft type specifications

**company-enrichment** — Company data enrichment by name or domain

- `datpaq company-enrichment` — Returns structured company data including industry, size, social profiles, and contact information. Provide...

**convert-time** — Timezone-aware datetime conversion

- `datpaq convert-time` — Converts a datetime value from a source timezone to a target timezone. Accepts ISO 8601 strings, Unix epoch...

**country-codes** — ISO country code lookup and metadata

- `datpaq country-codes country-by-iso` — Get a country by ISO 2 or ISO 3 code
- `datpaq country-codes country-export` — Export country data in bulk
- `datpaq country-codes country-list` — List all countries with ISO codes and metadata
- `datpaq country-codes country-search` — Search countries by full or partial name

**current-time** — Current time by IANA timezone identifier

- `datpaq current-time` — Returns the current date/time with UTC offset and DST status for a given timezone.

**define** — Manage define

- `datpaq define` — Returns definitions, part of speech, pronunciation, and etymology for the given word.

**dns** — Manage dns

- `datpaq dns` — Supports record types A, AAAA, MX, CNAME, TXT, NS, SOA, and ALL.

**domain-lookup** — Domain availability and registration information

- `datpaq domain-lookup get` — Check domain availability and registration info
- `datpaq domain-lookup post` — Check domain availability (POST)

**email-validation** — Email address format and deliverability validation

- `datpaq email-validation email-validate-batch` — Validate multiple email addresses
- `datpaq email-validation email-validate-single` — Checks email format, domain MX records, and optional deliverability. Detects disposable/temporary address providers.

**ev-charger** — Electric vehicle charging station discovery and status

- `datpaq ev-charger ev-networks` — List all supported charging networks
- `datpaq ev-charger ev-station-by-id` — Get specific charging station by ID
- `datpaq ev-charger ev-station-status` — Get real-time station status
- `datpaq ev-charger ev-stations-by-address` — Search EV charging stations by address
- `datpaq ev-charger ev-stations-by-coords` — Find EV charging stations near coordinates

**exchange-rates-and-currency** — Manage exchange rates and currency

- `datpaq exchange-rates-and-currency exchange-rate-bulk` — Batch currency conversions
- `datpaq exchange-rates-and-currency exchange-rate-get` — Returns the current exchange rate between two currencies. Provide an amount to perform conversion. Supports fiat...

**generate** — Manage generate

- `datpaq generate` — Generate realistic mock data using a JSON request body. Recommended for full control over type, count, fields, and...

**generate-batch** — Manage generate batch

- `datpaq generate-batch` — Submit multiple sample-data generation requests in one call.

**helicopter** — Helicopter registry lookups by registration, ICAO, or serial number

- `datpaq helicopter lookup-by-icao` — Look up helicopter by ICAO hex code
- `datpaq helicopter lookup-by-registration` — Look up helicopter by registration number
- `datpaq helicopter lookup-by-serial` — Look up helicopter by manufacturer serial number
- `datpaq helicopter manufacturers` — List helicopter manufacturers

**image-processing** — Image resize, compress, convert, crop, and pipeline operations

- `datpaq image-processing image-compress` — Compress an image
- `datpaq image-processing image-convert` — Convert image to a different format
- `datpaq image-processing image-crop` — Crop an image to a region
- `datpaq image-processing image-metadata` — Extract metadata from an image

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [datpaq/cli](https://github.com/datpaq/cli) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
