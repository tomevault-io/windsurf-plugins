---
trigger: always_on
description: This includes instructions for editing the OBA ontology.
---

# Ontology of Biological Attributes (OBA) Project Guide

This includes instructions for editing the OBA ontology. 

## Project Layout
- Main development file is `src/patterns/definitions.owl` (functional syntax)
- ODK and ontology documentation can be found in `/docs/`

## Querying ontology

- Use grep/rg to find terms. To look at a specific term if you know the ID:
    - `grep 'id: OBA_2040177' src/patterns/definitions.owl`
- All mentions of an ID
    - `obo-grep.pl -r 'OBA_2040177' src/patterns/definitions.owl`  
- Search over `src/patterns/definitions.owl`,`src/patterns/dosdp-patterns/*.yaml` and `src/patterns/data/default`.
- DO NOT bother doing your own greps over the files, or looking for other files, unless otherwise asked, you will just waste time.
- ONLY use the methods above for searching the ontology

## Before making edits
- Read the request carefully and make a plan, especially if there is nuance
- If related issues are mentioned read them: `gh issue view GITHUB-ISSUE-NUMBER`
- If a PMID is mentioned in the issue, ALWAYS try and read it
- ALWAYS check proposed parent terms for consistency

## Editors guide

### How to create new terms in OBA

#### Overview
- OBA terms are compositional, check DOSDP design patterns: `src/patterns/dosdp-patterns/*.yaml`
- Design pattern templates are found in `src/patterns/data/default`

#### Steps

    1- Read the issue carefully to identify the most appropriate design pattern and filler terms. Use the following examples as a general guide.

        Example 1: The amount or level of cholesterol when measured in blood serum. Use the design pattern template `entity_attribute_location.tsv` and the filler terms: cholesterol, amount and blood serum.
                
        Example 2: The age at which asthma manifestations first appear. Use the design pattern template `disease_onset.tsv` and the filler term: asthma.

        Example 3: A trait that affects the response to prednisolone. Use the design pattern template `response_to_chemical_stimulus_trait.tsv` and the filler term: prednisolone.
            
    2- Check whether design pattern fillers have been imported to OBA. `grep - i src/ontology/imports/merged_import.owl`

    3- If they have been imported, please proceed to creating the new OBA term using the appropriate DOSDP pattern template.

    4- Add the term to the template, then in `src/ontology` run
            `make IMP=false MIR=false ../patterns/definitions.owl` 

    5- Refresh OBA imports by running
            `make refresh-merged`

    6 - Run `make IMP=false MIR=false ../patterns/definitions.owl` again.

    7- When creating new OBA terms using DOSDP patterns, it may be necessary to import terms from other ontologies like ChEBI, the Chemical Entities of Biological Interest or PRO, the PRotein Ontology. However, they are too large to be managed as standard imports so are managed as slims. They are located here:
        PRO: https://github.com/obophenotype/pro_obo_slim
        CHEBI: https://github.com/obophenotype/chebi_obo_slim

    When a new term you are using in a DOSDP pattern is not yet in a slim, make a comment in the issue requesting the terms to be added manually. 

## OBO Format Guidelines
- Term ID format: OBA:NNNNNNN (7-digit number)
- Handling New Term Requests (NTRs):
  - New terms start OBA:210xxxx
  - Do `grep id: OBA:210 src/patterns/definitions.owl` to check for clashes
- Each term requires: id, name, definition with references
- Never guess OBA IDs, or ontology term IDs, use search tools above to determine actual term
- Never guess PMIDs for references, do a web search if needed
- Use standard relationship types: `is_a`, `part_of`, `characteristic_of`, `characteristic_of_part_of`, `subclass_of` and `has_role`.
- Follow existing term patterns for consistency

## Publications
- Run the command `aurelian fulltext <PMID:nnn>` to fetch full text for a publication. A doi or URL can also be used
- You should cite publications appropriately, e.g. `def: "...." [PMID:nnnn, doi:mmmm]

## GitHub Contribution Process
- Most requests from users should follow one of two patterns:
    - You are not confident how to proceed, in which case end with asking a clarifying question (via `gh`)
    - You are confident how to proceed, you make changes, commit on a branch, and open a PR for the user to review
- Check existing terms before adding new ones
- For new terms: provide name, definition, place in hierarchy, and references
- Include PMIDs for all assertions
- Follow naming conventions from parent terms
- Always commit in a branch, e.g. issue-NNN
- If there is an existing PR which you started then checkout that branch and continue, rather than starting a new PR (unless you explicitly want to abandon the original PR, e.g. it was on completely the wrong tracks)
- Always make clear detailed commit messages, saying what you did and why
- Create PRs using `gh pr create ...`
- File PRs with clear descriptions, and sign your PR

## Handling GitHub issues and requests
- Use `gh` to read and write issues/PRs

## Obsoleting terms originating from DOS-DP templates

- Delete the row with the targeted term from the appropriate data table (tsv file) in folder: `src/patterns/data/default/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [obophenotype/bio-attribute-ontology](https://github.com/obophenotype/bio-attribute-ontology) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
