---
trigger: always_on
description: This project contains crawlers that import source data, such as sanctions lists and other KYC/AML screening data, into the FollowTheMoney entities. It puts an emphasis on data cleaning. Much of the input data is semi-structured information published by government bodies - often rife with inconsistencies, manual data entry errors, etc. Our goal is to bring strict interpretation to these source datasets.
---

This project contains crawlers that import source data, such as sanctions lists and other KYC/AML screening data, into the FollowTheMoney entities. It puts an emphasis on data cleaning. Much of the input data is semi-structured information published by government bodies - often rife with inconsistencies, manual data entry errors, etc. Our goal is to bring strict interpretation to these source datasets.

### Repo layout

* `zavod` contains an ETL framework for crawlers, including definitions for metadata (`zavod.meta`), entity structure (`zavod.entity.Entity`) and crawler context (`zavod.context.Context`).
    * To discover which schemata and properties are available, use the `ftm ref` command group — the authoritative, always-current view of the model:
        * `ftm ref schemata` — list all schemata (add `--matchable` to filter)
        * `ftm ref schema Person` — one schema with all its (inherited) properties
        * `ftm ref types` / `ftm ref type country` — list property types, or detail one with its allowed values
        * `ftm ref prop Person:nationality` — one property's type and allowed values
      Output is a formatted table in a real terminal, but JSON whenever the output is captured — so when you run these, you'll always get JSON. Pipe to `jq` to extract fields, e.g. `ftm ref schema Person | jq -r '.properties[] | "\(.name) [\(.type)]"'` (`.properties` is a list, not keyed by name).
    * Data cleaning functions from `rigour` are documented at: https://rigour.followthemoney.tech/
    * Write code for all zavod functions in `zavod/zavod/tests`.
    * Run tests using `cd zavod && pytest zavod/tests/`.
    * Run `cd zavod && mypy --strict --exclude zavod/tests zavod/` after each change to zavod.
* `datasets` contains crawlers. Each crawler is defined using a `.yml` file (eg. `datasets/us/ofac/us_ofac_sdn.yml`), and a code file (often `crawler.py`, but defined using the `entry_point` key of the dataset `.yml`). The dataset has a `name`, which is based on the `.yml` file name stem (e.g. `us_ofac_sdn`).
    * To run a crawler: `zavod crawl <file_path>` in the project root. Running crawl several times might re-use the same data fetched in the initial run (`context.fetch_resource`).
    * When a crawler encounters uncertainty in any of the data it is parsing, it should crash or produce an error instead of emitting ambiguous data.
    * Crawlers use `lookups` to override specific values for entity properties of a particular type. For ambiguous data, individual cases can be clarified by adding lookups.
    * After running a crawler, output data is written to `data/datasets/<dataset_name>/`. The file `issues.log` contains line-based JSON of any warnings or errors produced by the crawler. Often the source data fetched by `context.fetch_resource` is also available in that folder.
    * To see a dataset's state in production — whether the latest run failed and since when, links to its archived artifacts, the current issues, and assertion drift — run `python -m contrib.maintenance.diagnose <dataset_name>` from the repo root.
    * Crawlers commonly do `from zavod import helpers as h`. The relevant code is in `zavod/zavod/helpers`. Use this pattern over direct imports.
* `ui` contains a NextJS user interface for reviewing and verifying information from crawlers. The contained table structures need to match those in `zavod.stateful`.
* `zavod/docs` documents the zavod framework, as well as best practices, especially with regards to semantic issues like Politically Exposed Persons

## Navigating the knowledge base

Use search (grep/glob/find) to find the most relevant starting document. Once you have opened a document, follow its cross-reference links to gather related context as needed — the links are curated and encode domain relationships that keyword search misses. Avoid following links more than two hops from your starting document. If something remains unclear, fall back to search or ask.

## Content guidelines

* The documentation for how we want crawlers to be written in `zavod/docs` is authoritative.
* When best practices are identified or changed, it should be documented in `zavod/docs`, and conflicting documentation should be updated.
* Refer to the [documentation styleguide for the broader FollowTheMoney ecosystem](https://github.com/opensanctions/followthemoney/blob/main/docs/styleguide.md).
* Do not assume that existing crawlers are implemented according to current best practices. Only use patterns in existing crawlers to update the documentation when prompted to, and in that case, identify specific patterns and propose specific aspects as separate concepts.
* Claude skills should reference and search relevant documentation in `zavod/docs`.
  * When writing skills, only the skill-specific procedures and examples should be in the skill directory.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [opensanctions/opensanctions](https://github.com/opensanctions/opensanctions) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-23 -->
