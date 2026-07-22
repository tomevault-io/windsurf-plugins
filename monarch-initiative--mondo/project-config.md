---
trigger: always_on
description: This includes instructions for editing the npso ontology.
---

# MONDO Ontology Project Guide

This includes instructions for editing the npso ontology. 

## Project Layout
- Main development file is `src/ontology/mondo-edit.obo`
- individual terms checked out in `terms`

## Querying ontology

- To look at a specific term if you know the ID:
    - `obo-grep.pl --noheader -r 'id: MONDO:0004177' src/ontology/mondo-edit.obo`
- All mentions of an ID
    - `obo-grep.pl --noheader -r 'MONDO:0004177' src/ontology/mondo-edit.obo`
- Regexes; all mentions of hand or foot:
    - `obo-grep.pl --noheader -r '(hand|foot)' src/ontology/mondo-edit.obo`
- Note that `obo-grep.pl` is in your PATH, no need to search for it    
- Only search over `src/ontology/mondo-edit.obo`
- DO NOT bother doing your own greps over the file, or looking for other files, unless otherwise asked, you will just waste time.
- ONLY use the methods above for searching the ontology

## Before making edits
- Read the request carefully and make a plan, especially if there is nuance
- if a PMID is mentioned in the issue, ALWAYS try and read it
- Use this command to fetch full text: `aurelian fulltext PMID:NNNNNN` (using the actual ID)
- This also works for other URLs for scientific papers and for DOIs, if there are no access restrictions
- ALWAYS check proposed parent terms for consistency

## Edits
- IMPORTANT: Do not edit the edit file directly, it's large
- Add edits should be made in the `terms/` folder
- check out a term into `terms/`: `obo-checkout.pl src/ontology/mondo-edit.obo MONDO:1234567 [OTHER IDS]`
- This will create a single stanza obo files `terms/npso_1234567.obo` which you can easily edit
     - (note the colon is replaced with an underscore)
- You can go ahead and edit the smallers files in the `terms/` folder
- After edits, check back in: `obo-checkin.pl src/ontology/mondo-edit.obo MONDO:1234567 [OTHER IDS]`
- if you like you can edit multiple terms in one batch, e.g. `terms/my_batch.obo`
     - `obo-checkout.pl src/ontology/mondo-edit.obo terms/my_batch.obo`
- checking in will update the edit file and remove the file from `terms/`
- Commits are then made on src/ontology/mondo-edit.obo as appropriate
- Note that `obo-checkin.pl` and `obo-checkin.pl` are in your PATH, no need to search for it    

## Specific Guidelines for MONDO Curation
- When processing a request to add a more specific parent (is_a, subclass) to a term, do not remove existing parents unless _explicitly instructed_ to do so.
- When removing a subclass (is_a) axiom, add a corresponding "excludedSubClassOf" annotation which includes the source of the removed axiom, and the ORCID of the curator, if available. Example:

```
relationship: excluded_subClassOf MONDO:0002129 {source="https://orcid.org/0000-0001-5208-3432"} ! bone cancer
```
- Before finalising an edit for a session, we need to run `sh run.sh make NORM && mv NORM mondo-edit.obo` to normalise the serialisation.

## OBO Format Guidelines
- Term ID format: MONDO:NNNNNNN (7-digit number)
- Handling New Term Requests (NTRs):
  - New terms start  MONDO:777xxxx
  - Do do `grep id: MONDO:777 src/ontology/mondo-edit.obo` to check for clashes
- Each term requires: id, name, namespace, definition with references
- NEVER guess MONDO IDs, use search tools above to determine actual term
- NEVER guess PMIDs for references, do a web search if needed
- Use standard relationship types: is_a, part_of, has_part, etc.
- Follow existing term patterns for consistency

## Publications
- Run the command `aurelian fulltext <PMID:nnn>` to fetch full text for a publication. A doi or URL can also be used
- You should cite publications appropriately, e.g. `def: "...." [PMID:nnnn, doi:mmmm]

## Syntax Checking

- To validate syntax: `robot convert --catalog src/ontology/catalog-v001.xml -i src/ontology/mondo-edit.obo -f obo -o mondo-edit.TMP.obo`
- Use `-vvv` for a full stack trace if there are errors.

## Mondo Guidelines

## Mappings

Mappings are an integral part of Mondo. You should never create mappings unless requested by a user, if they do, make sure metadata is correct

Mondo has an idiosyncratic way of adding metadata to mappings. Indicating the kind of mapping is important, this is dome by overloading
the source qualifier and using an ID like MONDO:equivalentTo. Also the original source can be included:

```
[Term]
id: MONDO:0800447
name: bleeding disorder, platelet-type, 13, susceptibility to
subset: predisposition
synonym: "BDPLT13" EXACT ABBREVIATION [MONDO:Lexical, OMIM:614009]
synonym: "bleeding disorder, platelet-type, 13, susceptibility to" EXACT [MONDO:Lexical, OMIM:614009]
synonym: "bleeding disorder, susceptibility to, due to defective platelet thromboxane A2 receptor" EXACT [OMIM:614009]
synonym: "susceptibility to platelet-type bleeding disorder 13" EXACT []
xref: MEDGEN:481244 {source="MONDO:equivalentTo", source="MONDO:MEDGEN"}
xref: OMIM:614009 {source="Orphanet:220443", source="MONDO:equivalentTo", source="Orphanet:220443/e"}
xref: UMLS:C3279614 {source="MONDO:equivalentTo", source="MONDO:MEDGEN", source="MEDGEN:481244"}
is_a: MONDO:0020573 {source="OMIM:614009"} ! inherited disease susceptibility
relationship: has_material_basis_in_germline_mutation_in http://identifiers.org/hgnc/11608 {source="MONDO:mim2gene_medgen"} ! TBXA2R

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [monarch-initiative/mondo](https://github.com/monarch-initiative/mondo) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-22 -->
