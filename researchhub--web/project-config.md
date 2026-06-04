---
trigger: always_on
description: This document provides architectural and relationship information about the ResearchHub PostgreSQL database, complementing the detailed table specifications in `database-tables.cursor-rules`.
---

# ResearchHub Database Architecture (Cursor Rule)

This document provides architectural and relationship information about the ResearchHub PostgreSQL database, complementing the detailed table specifications in `database-tables.cursor-rules`.

## System Architecture Overview

ResearchHub's database design implements a domain-driven approach centered around research content, with several key architectural patterns:

1. **Unified Document Model**: Content (papers, posts, notes) is abstracted through a unified document system
2. **Polymorphic Relationships**: Generic relations support flexible content relationships
3. **Reputation Economy**: A complete transaction system for academic reputation
4. **Hub-based Organization**: Content categorization through research fields
5. **User Identity System**: Separates user accounts from author profiles
6. **Nonprofit Integration**: Supports fundraising for nonprofit organizations through the Endaoment service

## Entity Relationship Diagram (Conceptual)

```
[USER SYSTEM]                     [CONTENT SYSTEM]                     [ORGANIZATION SYSTEM]
+----+                  +---+                 +----+
| user_user   | <----> | paper_paper      | <----> | hub_hub     |
+----+                  +---+                 +----+
       |                                |                                   |
       v                                v                                   v
+----+                  +---+                 +----+
| user_author | <----> | discussion_thread | <----> | topic_topic |
+----+                  +---+                 +----+
       |                                |                                   |
       v                                v                                   |
+----+                  +---+                      |
| user_action | <----> | reputation_score  | <----+
+----+                  +---+

[NONPROFIT SYSTEM]
+---------------+                 +---------------+
| nonprofit_org | <-------------> | purchase_fundraise |
+---------------+                 +---------------+
       |                                |
       v                                v
+------------------------+
| nonprofit_fundraise_link |
+------------------------+
```

## Domain Models

### Content Domain

The content model centers around unified documents that can represent different types of content:

1. **Papers** (`paper_paper`): Research papers, which can be:
   - Imported from external sources with DOIs
   - Uploaded directly by users
   - Created as preprints within the system

2. **Discussions** (`discussion_thread`): Conversations that can be:
   - Attached to papers
   - Created as standalone discussions
   - Function as comments on other content

3. **Posts** (`researchhub_document_researchhubpost`): User-generated content that can be:
   - Research summaries
   - Commentary
   - Blog-style posts

All content connects through the `researchhub_document_researchhubunifieddocument` table, which establishes a polymorphic pattern allowing different content types to be treated uniformly in many contexts.

### User Domain

The user model separates identity from academic profile:

1. **User Accounts** (`user_user`): Authentication and platform identity
2. **Author Profiles** (`user_author`): Academic identity and credentials
3. **Verification** (`user_userverification`): Processes for validating academic credentials

This separation enables:
- One user to manage multiple author profiles
- Author identity verification without affecting user account access
- Academic metrics separate from platform activity metrics

### Reputation Domain

The reputation system implements a complete economic model:

1. **Scoring** (`reputation_score`): Point values representing academic impact
2. **Transactions** (`reputation_distribution`): Movement of reputation between users
3. **Incentives** (`reputation_bounty`): Rewards for valuable contributions
4. **Escrow** (`reputation_escrow`): Holding system for reputation in process

### Organization Domain

Content is organized through multiple taxonomies:

1. **Hubs** (`hub_hub`): Primary research field categorization
2. **Topics** (`topic_topic`): Finer-grained subject matter classification
3. **Concepts** (`researchhub_document_unifieddocumentconcepts`): Specific concepts within content

## Key Relationships

### Content to Users

Content links to users through multiple relationship types:

1. **Authorship**: `paper_authorship` connects papers to authors
2. **Creation**: Most content has a `created_by_id` reference to `user_user`
3. **Interaction**: `discussion_vote`, `user_action` track user interactions with content

### Content Organization

Content is organized through:

1. **Hub Assignment**: `researchhub_document_researchhubunifieddocument_hubs` junction table
2. **Topic Association**: `topic_unifieddocumenttopics` relevancy-scored connections
3. **Hierarchy**: Topics belong to subfields, which belong to fields

### Polymorphic Relationships

The system uses Django's ContentType framework for polymorphic relationships:


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ResearchHub/web](https://github.com/ResearchHub/web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
