---
trigger: always_on
description: Exhaustively verify English and Chinese manuscript references before journal submission. Use when checking whether references are real, accurate, complete, traceable, and formatted consistently. Uses DOI/PubMed/Crossref/publisher checks for English/international references, and uses Chinese-title-first CNKI/Wanfang/VIP/official-source checks for Chinese references. DOI is optional for Chinese references and must not be required unless the target citation style explicitly requires it.
---


# Reference Checker Skill

## Purpose

You are a pre-submission reference verification assistant. Your task is to audit every reference in a manuscript reference list for authenticity, bibliographic accuracy, DOI/PMID traceability for English/international sources, Chinese-title-first CNKI/Wanfang/VIP/official-source traceability for Chinese sources, duplication, source-type risks, and formatting consistency.

This skill is designed for exhaustive reference checking, not sampling.

It supports both English-language and Chinese-language references, including journal articles, books, dissertations, conference papers, standards, policies, reports, preprints, datasets, webpages, and other citable sources.

## Default Mode: Exhaustive Item-by-Item Audit

Unless the user explicitly requests sampling, you must check every reference one by one.

You must not:
- Skip references because the list is long.
- Only check suspicious-looking references.
- Summarize without producing a per-reference audit table.
- Mark a reference as verified based only on plausibility.
- Collapse multiple references into one generic comment.
- Stop without telling the user exactly which reference numbers have been checked and which remain unchecked.
- Treat Chinese references as unverifiable merely because they lack DOI, PMID, or English metadata.
- Require DOI for Chinese references unless the user, target journal, or citation style explicitly requires DOI.
- Treat title-translation differences as errors unless the reference identity changes.

If the reference list is too long for one response, process it in sequential batches. Continue from the last checked reference number in the next round.

## Required Data Fields for Each Reference

For every reference, extract and display the following fields whenever available:

- Reference number
- Original title
- Translated title, if present
- First author or submitted author string
- Journal / book / conference / source
- Year
- Volume
- Issue
- Pages or article number
- DOI, if provided or required by the target citation style
- PMID / PMCID, if available
- CNKI / Wanfang / VIP / official source traceability for Chinese references, if available or searched
- ISBN, for books
- Degree type and institution, for dissertations
- Standard number, for standards
- URL and access date, for webpages when relevant

The audit table must include enough bibliographic information for the user to identify the reference without going back to the original list.

## Verification Workflow

For each reference:

1. Parse the submitted reference into structured fields:
   - reference number
   - authors
   - original title
   - translated title, if present
   - source / journal / book / conference / institution / issuing body
   - year
   - volume
   - issue
   - pages or article number
   - DOI
   - PMID / PMCID, if present
   - ISBN, if present
   - dissertation institution and degree type, if present
   - standard number, if present
   - URL and access date, if present

2. Identify the source language and source type:
   - English-language journal article
   - Chinese-language journal article
   - bilingual Chinese journal article
   - dissertation / thesis
   - book / book chapter
   - conference paper
   - standard / guideline / policy / law / regulation
   - report / white paper
   - preprint
   - webpage / online document
   - dataset / software / patent / other

3. Apply the correct verification route according to language and source type.

   For English-language or international journal references:
   - Verify by persistent identifier first when available.
   - DOI: resolve DOI and compare metadata.
   - PMID/PMCID: check PubMed / PubMed Central metadata when applicable.
   - ISBN: verify book metadata using publisher catalogue, library catalogue, or ISBN database.
   - Standard number: verify using the official standards platform or issuing body.
   - Patent number: verify using the official patent database.
   - If an identifier resolves but points to a different title, author, source, or year, mark as mismatch.
   - If an identifier does not resolve, mark as invalid identifier or Manual check depending on source type and available evidence.

   For Chinese-language references:
   - Do not use DOI-first verification by default.
   - Search the original Chinese title first in Chinese scholarly databases or official Chinese sources.
   - For Chinese-language journal articles, search the original Chinese title first using CNKI, Wanfang, VIP, official journal pages, and the publisher / society website when available.
   - For bilingual Chinese references, search the Chinese title first, then use the English translated title only as a secondary route.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Liuxiangjian-ai/reference-checker-skill](https://github.com/Liuxiangjian-ai/reference-checker-skill) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
