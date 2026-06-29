---
trigger: always_on
description: >
---


# TML — Tool Manifest Language

TML is a compact, line-oriented format for describing REST API tools so AI agents
can discover and call them with minimal token overhead. This skill teaches Claude
how to read existing TML, write new TML from scratch or from an OpenAPI spec,
compile TML to the .min bootstrap format, and help agents use both formats.

---

## Quick Reference

| Task | What to do |
|------|-----------|
| Read a .min file | Parse header then one tool per line, pipe-delimited |
| Read a .tml file | Parse blocks separated by `---` |
| Write TML from scratch | Use the block grammar below |
| Convert OpenAPI to TML | Extract operationId, parameters, responses → TML block |
| Compile TML to .min | Minify each block to a single pipe-delimited line |
| Help agent call a tool | Extract method + path + params from .min line, build HTTP request |
| Create /.well-known/tools.min | Write a valid .min file served at that path |

---

## Part 1 — The .min Format (Agent Bootstrap)

### Structure

Every `.min` file has one header line followed by one tool per line:

```
#oatr1|{api_name}|{base_url}
{id}|{METHOD} {path}|{params}|{output}|{tags}
```

### Header line

```
#oatr1|Open-Meteo Weather|https://api.open-meteo.com/v1
```

Fields (pipe-separated):
- `#oatr1` — version marker, always this value for now
- API name — human-readable display name
- Base URL — prepended to relative paths in tool lines

### Tool lines

```
weather.current|GET /forecast|latitude:n!,longitude:n!,current_weather:b=t,timezone:s=auto|{current_weather:{temp,windspeed,code}}|weather current now
```

Five fields:

| Field | Description | Example |
|-------|-------------|---------|
| id | dot.notation identifier | `weather.current` |
| method + path | HTTP verb + endpoint | `GET /forecast` |
| params | comma-separated param definitions | `q:s!,limit:i=10` |
| output | compact response shape | `{id,name,status}` |
| tags | space-separated intent tags | `weather current temperature` |

### Parameter syntax

```
{name}:{type}{modifier}{=default}
```

**Types:**

| Code | Full type | Example |
|------|-----------|---------|
| `s` | string | `q:s!` |
| `n` | number/float | `latitude:n!` |
| `i` | integer | `limit:i=10` |
| `b` | boolean | `active:b=true` |
| `a` | array | `ids:a?` |
| `o` | object | `fields:o!` |

**Modifiers:**

| Symbol | Meaning | Example |
|--------|---------|---------|
| `!` | required | `name:s!` |
| `?` | optional | `cursor:s?` |
| `=value` | default value | `limit:i=10` |

No modifier means optional with no default.

### Complete .min example

```
#oatr1|Stripe Payments|https://api.stripe.com
charges.create|POST /v1/charges|amount:i!,currency:s!,customer:s?,description:s?,capture:b=true|{id,amount,status,paid}|stripe charge payment create
charges.list|GET /v1/charges|limit:i=10,customer:s?,starting_after:s?|{data:[{id,amount,status}],has_more}|stripe charges list
customers.create|POST /v1/customers|email:s?,name:s?,description:s?|{id,email,name,created}|stripe customer create new
refunds.create|POST /v1/refunds|charge:s!,amount:i?,reason:s?|{id,amount,status,charge}|stripe refund create
```

---

## Part 2 — The TML Format (Source Layer)

TML is the human-editable source that compiles to .min. Use it when:
- You need full parameter descriptions
- Auth details must be explicit
- The tool definition will be maintained by humans
- You're creating the source for a registry

### Block grammar

```
@{tool_id}
desc: {intent description — written for AI disambiguation}
{METHOD} {full_url_or_path}
auth: {auth_type}[{detail}]
in:
  {param}:{type}{modifier}{=default}  "{description}"
out: {output_shape}
tags: {space-separated tags}
---
```

Every block ends with `---`. Multiple blocks in one file.

### TML types (longer names than .min)

| TML type | .min code | Meaning |
|----------|-----------|---------|
| `str` | `s` | string |
| `num` | `n` | float/decimal |
| `int` | `i` | integer |
| `bool` | `b` | boolean |
| `arr` | `a` | array |
| `obj` | `o` | object |

Modifiers are identical: `!` required, `?` optional, `=value` default.

### Auth values

| Value | When to use |
|-------|-------------|
| `none` | No auth required |
| `bearer` | Authorization: Bearer {token} |
| `apikey[header:X-Api-Key]` | API key in named header |
| `apikey[query:api_key]` | API key as query parameter |
| `oauth2[scope1,scope2]` | OAuth2 with listed scopes |
| `basic` | HTTP Basic authentication |

### Complete TML example

```
@weather.current
desc: Get current weather conditions for a location. Use when user asks
      about current temperature, wind, or conditions for a city right now.
      Do not use for forecasts or future weather.
GET https://api.open-meteo.com/v1/forecast
auth: none
in:
  latitude:num!              "Decimal latitude eg. 28.08 for Melbourne FL"
  longitude:num!             "Decimal longitude eg. -80.61 for Melbourne FL"
  current_weather:bool=true  "Must be true to get current_weather block"
  temperature_unit:str=celsius  "celsius or fahrenheit"
  timezone:str=auto          "eg. America/New_York or auto"
out: {latitude,longitude,current_weather:{temperature,windspeed,winddirection,weathercode,time}}
tags: weather current temperature wind conditions now
---
@weather.forecast_hourly
desc: Get hourly weather forecast up to 16 days. Use when user asks about

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [maf404/tml](https://github.com/maf404/tml) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
