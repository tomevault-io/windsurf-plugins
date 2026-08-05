---
trigger: always_on
description: Article search is the system that turns a user's reading intent into a precise, ordered set of articles from that user's library. It exists to make a large feed archive feel navigable: users should be able to ask for text, state, quality, recency, tags, sources, events, and ranking in one expression, and receive results that match the meaning of that expression.
---

# Article Search Architecture

Article search is the system that turns a user's reading intent into a precise, ordered set of articles from that user's library. It exists to make a large feed archive feel navigable: users should be able to ask for text, state, quality, recency, tags, sources, events, and ranking in one expression, and receive results that match the meaning of that expression.

This document describes what search is supposed to do. It is an architectural specification, not an implementation guide.

## Architectural Objective

Search must provide a predictable retrieval contract for articles.

A search request represents:

- The user whose library is being searched.
- The scope of the library being considered.
- The article states and metadata that constrain eligibility.
- The text intent that narrows relevance.
- The ordering model that decides what should appear first.
- The desired response shape: matching article identifiers or a matching count.

The search system is responsible for combining those dimensions into one coherent answer.

## Core Principles

Search is user-scoped. A result may only come from the requesting user's article library unless a behavior is explicitly defined as a cross-feed or global-user view. User ownership is the first boundary of every search.

Search is restrictive by composition. Each explicit filter narrows the eligible article set. Text, tags, dates, reading state, source scope, scores, event state, and language must work together rather than replacing one another, except where precedence is part of the query language.

Search is intent-preserving. A compact query string is not just text; it may contain structured intent. The system must distinguish between human text to search for and fielded constraints that describe how articles should be selected or ranked.

Search is stable enough to reason about. Equivalent requests should produce equivalent result sets and ordering, assuming the article library has not changed.

Search is optimized around article identity. The primary answer is the ordered set of matching article IDs. Full article rendering, enrichment, and presentation are responsibilities outside the search contract.

## Search Scope

Every search operates inside a library scope. The broadest normal scope is all feeds belonging to the user. Narrower scopes may select one feed, one category, or a derived subset such as tagged articles or grouped representatives.

Scope controls where results may come from. It does not decide relevance by itself. Relevance is determined after scope is established by applying text intent, state filters, metadata filters, quality thresholds, and ordering.

Some views intentionally loosen normal feed/category scope when the concept being requested is not source-local. For example, a hot-article view is about the article's global attention state inside the user's library, not about one selected feed.

## Query Intent

The search expression has two meanings at once:

- Free text expresses article content the user wants to find.
- Fielded tokens express constraints or ranking preferences.

Quoted text represents phrase intent. Unquoted text represents term intent. Fielded tokens represent structured intent and should not accidentally become normal text search terms.

When a structured token and an external request parameter describe the same concept, the search expression is the more specific user intent and takes precedence. This allows saved views and UI defaults to be refined from the search box without changing the surrounding view.

Legacy vocabulary remains part of the architecture when it represents the same user concept. For example, "starred" and "favorite" are one conceptual state even if older clients use older language.

## Article Eligibility

An article is eligible only when it satisfies the active constraints.

The fundamental eligibility dimensions are:

- Ownership: the article belongs to the requesting user.
- Source scope: the article is in the selected feed/category scope unless the requested concept overrides source locality.
- Canonical visibility: duplicate or non-canonical articles are excluded according to the product's canonical article rules.
- Reading state: unread, read, favorite, clicked, seen, hot, or all.
- Text relevance: title and article text match the requested term or phrase intent.
- Metadata: tag, title, author, language, date, first-seen age, event state, event size, interest-island applicability, or grouping concept.
- Quality gates: advertisement, sentiment, and quality thresholds are all satisfied.

Eligibility is binary. Ranking must not resurrect articles that failed eligibility.

## Time Concepts

Search supports human time intent. Users may ask for concrete dates, relative periods, recent days, or named days. These expressions describe publication-time ranges unless another product concept explicitly says otherwise.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pietheinstrengholt/rssmonster](https://github.com/pietheinstrengholt/rssmonster) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
