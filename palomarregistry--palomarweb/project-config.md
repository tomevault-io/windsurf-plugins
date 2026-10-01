---
trigger: always_on
description: - This repository is commonly developed on NixOS. If Playwright's bundled
---

# Agent notes

- This repository is commonly developed on NixOS. If Playwright's bundled
  Chromium fails to launch because a shared library such as
  `libglib-2.0.so.0` is unavailable, do not leave the browser suite unrun. Use
  the Nixpkgs browser instead:

  ```sh
  nix shell nixpkgs#chromium -c bash -lc \
    'export PLAYWRIGHT_CHROMIUM_EXECUTABLE_PATH=$(command -v chromium); npm run test:browser'
  ```

- `npm test` reads `schema-v3.json` and `tests/fixtures/recent.json` out of a
  PalomarDatabase checkout, from
  `PALOMAR_DATABASE_CHECKOUT` or a sibling `../PalomarDatabase/`. An
  unavailable contract is a failure rather than a skip, deliberately: the point
  of the tests is that this repository's validators agree with the Database
  outputs they read, and a version of those tests which quietly does nothing
  agrees with everything. Without a checkout you get hard failures and no hint
  why, so this is the first thing to check.

- `recent.json` has a strict document envelope and independently validated
  landing rows. Browser validation is limited to fields needed to render and
  link safely; PalomarDatabase owns policy such as classification cardinality,
  uniqueness, and exact additive shape. An unusable row is omitted with private
  diagnostics and a visible count, while deployment health still rejects any
  omission so producer drift is caught without taking down valid siblings.

- The subject surfaces—`subjects/<kind>/<code>.json`, its `<year>.json`, and its
  `<day>/<page>.json`—are the same closed contract, and are the same document
  family as browsing: PalomarDatabase writes both from
  `day_pages.write_collection`, so `security.mjs` reads their year and page
  levels with one shared validator and one shared schema constant. A subject row
  carries the classification and the registration instant on top of the index
  row, and both are checked: a row whose classification omits the code being
  asked for is a result under a heading it has nothing to do with, which is the
  one failure a well-formed row can still be. `check-published.mjs` validates
  the front page of every code the newest rows carry, bounded by the
  classification vocabulary rather than by the registry.

- The three browse surfaces—`browse/index.json`, `browse/<year>.json`, and
  `browse/<day>/<page>.json`—are also exact closed producer/consumer contracts.
  PalomarDatabase owns their head, year, page, count, and summary-row shapes;
  change those producer-first. A head and a year each publish the path of the
  level below as a template, `year_path` and `page_path`, for readers who have
  the document and not the grammar; a subject's are its own code's, because
  both collections derive them from the directory being written. This consumer
  has the grammar and requires the templates to equal it exactly. It never
  expands one to build a request: a template read as instructions is a path the
  data origin chooses. CI downloads one live head/year/page chain into
  the named producer-contract fixture test, then the predeploy check traverses
  every row the producer advertises. The traversal reconciles those surfaces;
  it cannot independently prove that their common producer omitted nothing.

- Entry records have one contract: `schema_version: 3` in `schema-v3.json`.
  Superseded drafts have no validator, preservation fallback, public schema
  download, or legacy presentation. The schema-v3 review-language cutover was
  deployed producer-first with its rewritten public data; do not infer an
  endorsement from a legacy positive review value. The consumer contract must
  be proved by `check-published.mjs --data`, which
  traverses every advertised browse page and per-ID version index and validates
  every advertised active entry before Pages artifact upload. A recent-only
  sample is not enough.
  `recent.json`, versions, and browse/subject projections use their
  schema-v2 protocols. Source availability and independent render/evidence
  metadata retain their own versioned contracts.

- Challenge render metadata is versioned independently inside each immutable,
  content-addressed render bundle. A version widening deploys the Web consumer
  first; PalomarSubmission may emit the new metadata version only after that
  consumer is live, because the previous consumer rejects unknown versions.
  This is different from replacing the shape of a closed projection at one
  version, which remains producer-first. `check-published.mjs --data` must read
  and validate every available render metadata document in its advertised
  entry traversal; a missing historical render retains the pinned-source
  fallback, but malformed metadata fails the deployment.

- `source-availability.json` is normalized by PalomarDatabase's executable
  source-availability contract and consumed under the same per-endpoint
  freshness rules here. A known answer is authoritative only from five minutes
  in the future through eighteen hours old, inclusively; malformed or older
  observations become unknown without hiding valid siblings. Contract changes
  are deployed producer-first and the cross-repository unit test is mandatory.
  With a canonical Database checkout the test invokes its executable contract;

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PalomarRegistry/PalomarWeb](https://github.com/PalomarRegistry/PalomarWeb) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-30 -->
