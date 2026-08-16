---
trigger: always_on
description: Maintain accurate, task-focused documentation for developers who evaluate,
---

# Nemotron Voice Agent Documentation Guidance

## Role

Maintain accurate, task-focused documentation for developers who evaluate,
configure, deploy, extend, and troubleshoot Nemotron Voice Agent. Verify every
command, profile, service name, model identifier, default, port, hardware
requirement, and configuration key against checked-in repository sources.

## Writing Style Guide

Apply these rules to documentation, examples, headings, UI text, and release
notes that you create or edit.

- Write in a professional, active, conversational, and engaging voice.
- Use active voice whenever possible. Use present tense for product behavior.
  Address the reader in second person as "you."
- Keep sentences concise. Prefer sentences with fewer than 30 words.
- Use plain English and precise technical terms. Avoid jargon, filler,
  colloquialisms, and flowery marketing claims.
- Avoid contractions in technical documentation. Write "do not," "cannot,"
  and "it is."
- Write "NVIDIA" in all caps and use "an NVIDIA," not "a NVIDIA."
- Spell out uncommon abbreviations on first use. Spell out LLM, RAG, SLM, VLM,
  and MoE on first use.
- Use NVIDIA spellings such as data center, dataset, open source, pretrained,
  startup, webpage, website, and Wi-Fi.
- Replace Latinisms with plain English. Use "for example," "that is," "and so
  on," "through," and "compared to."
- Use "refer to" instead of "see," "can" instead of "may" for possibility,
  and "after" instead of "once" for time.
- Do not use "please" in technical instructions.
- Use numerals for specific values, parameters, measurements, and values of 10
  or more. Spell out zero through nine in general prose.
- Include a space between a number and its unit. Use a comma in numbers with
  four or more digits.
- Use title case for headings. Do not style headings with code, bold, italics,
  quotation marks, ampersands, or exclamation marks.
- Use the Oxford comma. Put periods inside quotation marks in U.S. style.
- Use hyphens only for compound modifiers before nouns. Do not hyphenate an
  adverb that ends in "ly."
- Format commands, code, filenames, paths, and API identifiers as code. Use
  bold for UI elements and the greater-than sign for UI navigation.
- Introduce lists, tables, code examples, and images with a complete sentence.
  Use parallel construction in lists.
- Use descriptive link text. Do not use raw URLs in running text or generic
  link text such as "click here" or "read more."
- Write dates as Month DD, YYYY. Omit the year when it matches the publication
  year. Write time with a 12-hour clock and include minutes only when needed.
- Do not rewrite quoted UI labels, API field names, or audience role labels in
  tables to enforce second person.
- Provide useful alt text and preserve a logical heading hierarchy.
- Verify commands, flags, API names, defaults, and technical claims against
  source code or another checked-in source of truth.
- Do not rewrite literal code, identifiers, commands, URLs, or quoted terminal
  and API output to satisfy prose rules.
- Apply rules to improve clarity. Do not make mechanical changes that reduce
  technical accuracy or readability.

## NVIDIA DORI Routing

1. Check whether the current agent exposes `dori_handle` or `dori_route` and
   `dori_collections`.
   If the user explicitly asks not to use DORI, use the
   [Writing Style Guide](#writing-style-guide) instead.
2. When those tools are available, list the installed collections.
   - If a collection source exactly matches `gitlab:tech-docs/skill-library`,
     use DORI for task routing.
   - If the collection is missing, inaccessible, or cannot be verified,
     continue with the [Writing Style Guide](#writing-style-guide).
3. When the DORI tools are unavailable, continue with the Writing Style Guide.
   Do not inspect a shell-visible CLI, install software, or configure the host
   during a normal documentation task.
4. If DORI or the verified Skill Library is unavailable and the user has access
   to `gitlab-master.nvidia.com`, print a brief recommendation to install DORI
   and refer to [NVIDIA DORI Setup](DORI_SETUP.md). Continue the documentation
   task with the Writing Style Guide. Do not pause or block work for
   installation.
5. Run NVIDIA DORI Setup only when the user explicitly asks to install or
   configure DORI.

Capability detection does not approve installation or host configuration.
DORI unavailability must not block documentation work.

Use the following DORI workflow only when current host capabilities include the
verified NVIDIA documentation Skill Library. Complete the documentation before
the developer opens the pull or merge request.

1. Route the documentation task through DORI. Include the changed source files,
   the user-visible impact, the documentation that might need updates, and the
   required validation.
2. Follow the skill or workflow that DORI returns. Verify product behavior
   against checked-in sources before drafting.
3. When the host supports subagents, start a documentation subagent while the
   primary developer finishes the implementation. Reconcile the documentation
   changes and validation evidence before opening the pull or merge request.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [NVIDIA-AI-Blueprints/nemotron-voice-agent](https://github.com/NVIDIA-AI-Blueprints/nemotron-voice-agent) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-16 -->
