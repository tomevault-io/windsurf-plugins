---
trigger: always_on
description: Polaris is a decentralized, blockchain-anchored music registry with graph-based visualization. It provides a canonical, auditable registry of music creators, releases, tracks, and compositions with a focus on detailed relationship tracking and collaborative data submission.
---

# CLAUDE.md - AI Assistant Guide for Polaris Music Registry

## Project Overview

Polaris is a decentralized, blockchain-anchored music registry with graph-based visualization. It provides a canonical, auditable registry of music creators, releases, tracks, and compositions with a focus on detailed relationship tracking and collaborative data submission.

**Key Innovation**: Uses "Releases" as the primary accounting unit, with Groups (not individual artists) as the main performance entity. Relationships between Persons and Groups are carefully tracked through MEMBER_OF and GUEST_ON relationships.

### Technology Stack

- **Blockchain**: Antelope/EOS (smart contracts in C++)
- **Graph Database**: Neo4j (GQL-compatible Cypher queries)
- **Backend**: Node.js (ES modules) with Express
- **Frontend**: JavaScript InfoVis Toolkit (JIT) for hyperbolic graph visualization
- **Storage**: IPFS + S3 for event data, Redis for hot cache
- **Indexing**: Substreams (Rust-based blockchain data pipeline)
- **Governance**: Fractally DAO integration for Respect-weighted voting

## Repository Structure

```
polaris-music-registry/
├── contracts/              # Antelope C++ smart contracts
│   └── polaris.music.cpp   # Main contract (anchoring, voting, staking)
│
├── backend/
│   ├── src/
│   │   ├── api/           # Express GraphQL + REST API server
│   │   ├── graph/         # Neo4j schema and operations
│   │   ├── storage/       # Event storage (IPFS, S3, Redis)
│   │   └── indexer/       # Event processor (blockchain → graph)
│   └── test/
│       ├── api/           # API endpoint tests
│       ├── graph/         # Graph database tests
│       ├── storage/       # Storage layer tests
│       ├── e2e/           # End-to-end workflow tests
│       └── performance/   # Load and performance tests
│
├── frontend/
│   ├── src/
│   │   └── visualization/ # JIT-based graph visualization
│   └── public/
│
├── substreams/            # Blockchain event indexing
│   ├── src/              # Rust modules
│   └── proto/            # Protobuf definitions
│
├── tools/
│   ├── import/           # Data import from Discogs, etc.
│   └── migration/        # Database migration scripts
│
├── docs/                 # Detailed implementation specs
│   ├── 01-smart-contract.md
│   ├── 02-graph-database-schema.md
│   ├── 03-event-storage.md
│   └── ... (comprehensive documentation)
│
└── README.md            # User-facing documentation
```

## Core Concepts

### Domain Model

**CRITICAL**: Understand the distinction between these entities:

1. **Person**: Individual musician, producer, engineer, songwriter, etc.
   - Can be a MEMBER_OF multiple Groups over time
   - Can be a GUEST_ON tracks where they're not a group member

2. **Group**: Band, orchestra, ensemble, solo project (collection of Persons)
   - Has MEMBER_OF relationships with clear date ranges and roles
   - PERFORMED_ON tracks as a unit
   - Even solo artists are modeled as "groups of one"

3. **Song**: Musical composition (the written work)
   - Has WROTE relationships to Persons (songwriters)
   - Can have multiple Track recordings

4. **Track**: Specific recording/performance of a Song
   - RECORDING_OF a Song
   - PERFORMED_ON by Group(s)
   - Can have GUEST_ON relationships for non-members
   - Can SAMPLE other Tracks

5. **Release**: Album, EP, Single, Live Performance
   - Contains Tracks via IN_RELEASE relationship
   - IN_MASTER for canonical grouping of re-releases
   - RELEASED by Label

### Members vs. Guests - CRITICAL DISTINCTION

**Members** (`MEMBER_OF` relationship):
- Core group members credited on the album
- Perform on multiple tracks as part of the group
- Listed in liner notes as official band members
- Visualized in RGraph around Group nodes

**Guests** (`GUEST_ON` relationship):
- Individual musicians appearing outside group context
- Session musicians, featured artists, engineers
- Perform on specific tracks or contribute to release
- Visualized as non-colored edges to specific tracks/releases

**Rule of Thumb**: If someone performs on >2/3 of tracks and liner notes don't specify, they're likely members. If unclear, default to guest.

### Event-Driven Architecture

All data changes flow through canonical events:

```
User Submission → Event Creation → Off-chain Storage (IPFS+S3)
                                 ↓
                   Blockchain Anchoring (hash only)
                                 ↓
                   Event Processor → Neo4j Graph Update
                                 ↓
                   Community Voting → Finalization → Rewards
```

**Event Types** (defined in smart contract):
- `21` - CREATE_RELEASE_BUNDLE (full release with groups, tracks)
- `30` - ADD_CLAIM (add data to entity)
- `31` - EDIT_CLAIM (modify existing data)
- `40` - VOTE (vote on submission)
- `41` - LIKE (like a node, track path)
- `50` - FINALIZE (distribute rewards)
- `60` - MERGE_NODE (deduplicate entities)

## Development Workflows

### Adding New Features

1. **Read Documentation First**: Check `docs/` for existing specs

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [PolarisMusic/polaris-music](https://github.com/PolarisMusic/polaris-music) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
