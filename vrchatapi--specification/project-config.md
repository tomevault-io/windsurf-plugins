---
trigger: always_on
description: VRChat publishes no specification. Every path, field, type, and example here is
---

# VRChat API specification

VRChat publishes no specification. Every path, field, type, and example here is
reverse-engineered from live traffic, so a confident sentence nobody checked is
the failure this repo produces most.

`test/arazzo.yaml` drives ~500 workflows against the live API and writes the
capture to `test/.out/har/arazzo.har`. That capture is the evidence. Query it
before writing anything:

```sh
jq -r '[.log.entries[] | select(.request.url | contains("/users/")) | .response.content.text | fromjson] | .[0]' test/.out/har/arazzo.har
```

## Never document what the traffic has not shown

- Every property, type, enum value, `required` entry, and example comes from a
  captured response. Never add one from a field name, a sibling schema, or the
  client's behaviour.
- A field that is always `null` establishes nothing about its type. Write `{}`
  rather than guessing from the name. Never write a bare `nullable: true`: Ajv
  rejects it without a `type`, and the schema check fails.
- `nullable: true` means JSON `null` was captured. A field that is sometimes
  absent is not nullable.
- Close an `enum` only when the set is known. Two observed values do not make a
  set; a generated client will reject the third.
- `/config` returns decoy properties, plausible-looking names VRChat rerolls at
  random. Never document one, and keep `APIConfig` `additionalProperties: true`.

## Absent from one capture is not absent

Adding needs evidence; removing needs far more. A capture records one account in
one state, and a response changes shape with that state — `presence` carries
nine keys for an offline account and nineteen for an online one.

A property missing from a capture is a coverage gap, not a phantom. Add the
workflow that reaches the state carrying it, and judge the property once that
workflow has run. Deleting on the strength of one capture removes what the
suite merely failed to reach.

## An unevidenced schema is a missing test

`{}`, a value only ever `null`, and a property no response has carried all mean
the same thing: no workflow reaches the case that would settle it. Add the
workflow to `test/arazzo.yaml`; never close the gap by guessing.

`required` cuts both ways. A property every observed response carries but that
`required` omits is either required, or missing the workflow that shows it
absent. A property in `required` that some response omits is neither.

## Guess what to probe, verify before you write it

Documenting only what the traffic has shown is a rule about what reaches a
`description`, not about what you are allowed to think. Inference is how you
find the traffic worth capturing. Guess widely, then let a request settle it.

- An ID prefix names its resource, and a resource with its own prefix usually
  has a collection of its own. `icat` and `ivib` in a validation message
  unpacked to `GET /instanceCategories` and `GET /instanceVibes`, two routes no
  capture had ever touched.
- Send a deliberately wrong value to make the API name the format it wants.
  `categoryId: "x"` answered `categoryId must be a 'icat' ID`.
- Take the spelling from paths the description already carries. VRChat writes
  top-level collections as camelCase plurals: `/avatarStyles`, `/tokenBundles`,
  `/moderationReports`.
- A candidate path an existing route swallows answers with that route's error.
  `/instances/categories` returns the 400 from `getInstance` parsing
  `categories` as `worldId:instanceId`, which rules out every nested spelling
  in one request.
- Read a field where it is populated, not where it is empty. `languages` and
  `userIcons` are `[]` on every instance the suite creates and full on a busy
  public one.
- Probe candidates in a batch and read the status codes. A 200 establishes a
  route.

An inference no request settles stays out of the description. Say it to the
user as an open question instead.

## Check with curl, document from a workflow

One `curl` answers what a route does now, without running the suite. The session
cache holds the cookie:

```sh
curl -s -A "specification-test/1 (https://vrchat.community)" \
	-b "auth=$(cat test/.out/session)" \
	https://api.vrchat.cloud/api/1/economy/stores
```

Use it to check a single route, to confirm what a capture only implies, or to
separate an API behaviour from a tooling artifact: when curl and the suite
disagree on the same URL, the client differs, not VRChat.

A curl records one moment. Never write a description from one alone. Add the
workflow to `test/arazzo.yaml`, then document what it captures.

## A "not implemented" 404 means no route matched

VRChat answers `{"error":"The endpoint you're looking for is not implemented by
our system."}` when no route matches. A removed endpoint and a malformed path
both produce it: `/api/1//css/app.css` and a percent-encoded dot return it
byte-for-byte. This body never establishes that a route is gone. Request the
exact path directly before marking anything deprecated.

## One operation per workflow

A failing step ends its workflow, so every operation after it goes unexercised
and its coverage disappears without saying so. Give each operation its own
workflow. Chain steps only where a later one needs state an earlier one created:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vrchatapi/specification](https://github.com/vrchatapi/specification) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-09 -->
