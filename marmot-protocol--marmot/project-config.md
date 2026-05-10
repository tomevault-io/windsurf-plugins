---
trigger: always_on
description: A guide for AI coding agents working on the Marmot Protocol specification. This will work for most major AI harnesses. The `CLAUDE.md` file in this repo is a symlink to this doc.
---

# AGENTS.md (aka CLAUDE.md)

A guide for AI coding agents working on the Marmot Protocol specification. This will work for most major AI harnesses. The `CLAUDE.md` file in this repo is a symlink to this doc.

## Project Overview

Marmot is a secure, decentralized group messaging protocol that combines the [MLS Protocol](https://www.rfc-editor.org/rfc/rfc9420.html) with [Nostr's](https://github.com/nostr-protocol/nostr) decentralized network. The protocol enables end-to-end encrypted group messaging without relying on centralized servers.

**Key Characteristics:**

- Protocol specification repository (not an implementation)
- Markdown-based documentation (MIPs - Marmot Implementation Proposals)
- References MLS RFCs and Nostr NIPs
- Focus on security, interoperability, and clarity

## Project Structure

```plaintext
marmot/
├── README.md              # Main project overview
├── AGENTS.md             # This file
├── 00.md - 04.md         # Marmot Implementation Proposals (MIPs)
├── threat_model.md       # Security threat analysis
├── data_flows.md         # Protocol flow diagrams
├── dependency_reqs.md    # Implementation dependencies
├── docs/mls/             # Local copies of MLS RFCs
│   ├── rfc9420.txt       # MLS Protocol spec
│   ├── rfc9750.txt       # MLS Architecture
│   └── draft-ietf-mls-extensions-08.txt
└── assets/               # Images and diagrams
```

## Key Documents

### Marmot Implementation Proposals (MIPs)

- **[MIP-00](00.md)**: Credentials & Key Packages (REQUIRED)
- **[MIP-01](01.md)**: Group Construction & Marmot Group Data Extension (REQUIRED)
- **[MIP-02](02.md)**: Welcome Events (REQUIRED)
- **[MIP-03](03.md)**: Group Messages (REQUIRED)
- **[MIP-04](04.md)**: Encrypted Media (OPTIONAL)

### Supporting Documentation

- **[threat_model.md](threat_model.md)**: Comprehensive security analysis
- **[data_flows.md](data_flows.md)**: Protocol flow diagrams and architecture
- **[dependency_reqs.md](dependency_reqs.md)**: Implementation dependencies

## Protocol Fundamentals

### Core Concepts

1. **MLS (Messaging Layer Security)**: Provides group encryption, forward secrecy, and post-compromise security
2. **Nostr**: Provides decentralized relay network and identity system
3. **Marmot Group Data Extension**: Custom MLS extension containing group metadata
4. **KeyPackages**: Public invitation cards for asynchronous group joins
5. **Double Encryption**: MLS symmetric encryption + ChaCha20-Poly1305 (key derived from MLS exporter secret) for kind: 445 application messages

### Event Kinds

- `kind: 30443`: KeyPackage events (public invitations, addressable)
- `kind: 444`: Welcome events (gift-wrapped via NIP-59)
- `kind: 445`: Group events (messages, proposals, commits)
- `kind: 10051`: KeyPackage relay list events

### Critical Security Requirements

When working on protocol specifications, ensure these are always addressed:

1. **Credential Validation**: MLS credential identity MUST match Nostr pubkey in KeyPackage events
2. **Commit/Welcome Ordering**: Commits MUST be confirmed before sending Welcome events
3. **Ephemeral Keypair Uniqueness**: Fresh keypair for EVERY kind: 445 event
4. **Unsigned Inner Events**: Inner events MUST NOT be signed (prevents leak publication)
5. **Admin Authorization**: Commits that are not self-updates or SelfRemove-only MUST verify sender is in admin_pubkeys array (self-update and SelfRemove-only Commits are allowed from any member)
6. **TLS Serialization**: Exact TLS presentation language format required

## Code Style & Conventions

### Markdown Formatting

- Use clear headings with descriptive names
- Include code examples in JSON format for Nostr events
- Reference other MIPs using `[MIP-XX](XX.md)` format
- Use tables for structured data (event fields, security properties, etc.)
- Include security notes with ✅ (protections) and ⚠️ (limitations)

### Specification Writing

- **MUST/SHOULD/MAY**: Use RFC 2119 terminology for requirements
- **CRITICAL**: Mark security-critical requirements explicitly
- **Examples**: Include complete, valid examples
- **Edge Cases**: Document race conditions, error handling, and failure modes
- **Interoperability**: Consider cross-implementation compatibility

### Security Considerations

- Always reference threat_model.md for security context
- Document observable metadata and privacy limitations
- Include countermeasures for identified threats
- Mark security-critical sections with **CRITICAL** labels

## Common Tasks

### Adding a New MIP

1. Create `XX.md` file (next sequential number)
2. Follow structure of existing MIPs:
   - Title and status (review/draft)
   - Required/optional flag
   - Overview and motivation
   - Detailed specification
   - Examples
   - Security considerations
   - References to other MIPs
3. Update README.md MIP table
4. Update threat_model.md if security implications exist

### Updating Protocol Specifications

1. Read relevant MIPs and MLS RFCs
2. Check threat_model.md for security implications
3. Update data_flows.md if protocol flows change
4. Ensure examples remain valid
5. Update cross-references between MIPs

### Security Analysis


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [marmot-protocol/marmot](https://github.com/marmot-protocol/marmot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-04 -->
