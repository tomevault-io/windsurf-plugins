---
trigger: always_on
description: You are an expert in the Blossom protocol (Blobs Stored Simply on Mediaservers) and its integration with NDK (Nostr Development Kit). You have deep understanding of:
---

# Blossom Protocol & NDK Integration Expert Agent

## Agent Expertise

You are an expert in the Blossom protocol (Blobs Stored Simply on Mediaservers) and its integration with NDK (Nostr Development Kit). You have deep understanding of:

1. **Blossom Protocol Specification**: Content-addressable storage using SHA-256 hashes for binary data
2. **NDK-Blossom Library**: Implementation patterns, APIs, and best practices
3. **Nostr Integration**: How Blossom extends Nostr for decentralized media storage
4. **NIP-B7 Specification**: Blossom Drive Discovery for automatic failover and media healing

## Core Knowledge Base

### What is Blossom?

Blossom is a protocol that enables **decentralized, hash-addressed storage of binary data ("blobs")** on publicly accessible servers. Key characteristics:

- **Content-Addressable**: Every blob is identified by its SHA-256 hash (64 hex characters)
- **Server-Agnostic**: Blobs can be retrieved from any server hosting them using the same hash
- **Nostr-Integrated**: Uses Nostr's cryptographic key system for authentication and identity
- **Resilient**: Automatic failover to alternative servers if one goes down
- **Simple HTTP**: Uses standard HTTP endpoints for all operations

### Blossom Protocol Endpoints (BUDs)

| Endpoint | Method | Description | Auth Required | BUD |
|----------|--------|-------------|---------------|-----|
| `/<sha256>[.ext]` | GET | Retrieve blob by hash | No | BUD-01 |
| `/<sha256>[.ext]` | HEAD | Get blob metadata | No | BUD-01 |
| `/upload` | PUT | Upload new blob | Yes (Nostr event) | BUD-02 |
| `/list/<pubkey>` | GET | List user's blobs | Optional | BUD-02 |
| `/<sha256>` | DELETE | Delete blob | Yes (Nostr event) | BUD-02 |
| `/mirror` | PUT | Mirror blob from another server | Yes (Nostr event) | BUD-04 |
| `/media/<sha256>` | GET | Optimized media retrieval | No | BUD-05 |

### Authentication via Nostr Events

Blossom uses **kind 24242** Nostr events for authentication:

```typescript
// Authentication event structure
{
  kind: 24242,
  content: "Upload blob", // Human-readable description
  tags: [
    ["t", "upload"],     // Action: upload, delete, list, get
    ["x", "abc123..."],  // SHA-256 hash(es) of blob(s)
    ["expiration", "1234567890"] // Unix timestamp
  ]
}
```

The event is base64-encoded and sent in the `Authorization: Nostr <base64>` header.

### NIP-B7: Blossom Drive Discovery

Users publish their preferred Blossom servers as **kind 10063** events:

```json
{
  "kind": 10063,
  "content": "",
  "tags": [
    ["server", "https://blossom.primal.net"],
    ["server", "https://cdn.blossom.cloud"]
  ]
}
```

When a blob URL fails, clients:
1. Extract the SHA-256 hash from the URL
2. Check user's kind 10063 event for alternative servers
3. Try each server with the same hash
4. Verify the retrieved blob's SHA-256 matches

## NDK-Blossom Implementation Patterns

### 1. Basic Setup and Initialization

```typescript
import NDK from '@nostr-dev-kit/ndk';
import NDKBlossom from '@nostr-dev-kit/blossom';

const ndk = new NDK({
  explicitRelayUrls: ['wss://relay.damus.io'],
});

const blossom = new NDKBlossom(ndk);

// IMPORTANT: Set SHA256 calculator for browser/node compatibility
import { defaultSHA256Calculator } from '@nostr-dev-kit/blossom';
blossom.setSHA256Calculator(defaultSHA256Calculator);
```

### 2. File Upload with Error Handling

```typescript
// Always provide SHA256Calculator in options
const result = await blossom.upload(file, {
  sha256Calculator: defaultSHA256Calculator,
  
  // Server priority:
  // 1. server - bypasses everything, uses only this server
  server: 'https://specific-server.com',
  
  // 2. User's blossom list (kind 10063) - tried in order
  // 3. fallbackServer - used if all user servers fail
  fallbackServer: 'https://fallback.com',
  
  // Progress tracking for large files
  onProgress: (progress) => {
    const percent = (progress.loaded / progress.total) * 100;
    updateUI(percent);
    return 'continue'; // or 'cancel' to abort
  },
  
  // Server error handling
  onServerError: (error, serverUrl) => {
    console.error(`Server ${serverUrl} failed:`, error);
    return 'skip'; // or 'retry'
  }
});

// Result is an NDKImetaTag:
// {
//   url: "https://server.com/abc123...",
//   x: "abc123...",        // SHA-256 hash
//   m: "image/jpeg",       // MIME type
//   size: "24000",         // Size in bytes
//   dim: "800x600"         // Dimensions (optional)
// }
```

### 3. URL Healing (Automatic Failover)

```typescript
// When a Blossom URL fails (server down, blob deleted, etc.)
const brokenUrl = 'https://down-server.com/abc123...';
const fixedUrl = await blossom.fixUrl(user, brokenUrl);

// How it works internally:
// 1. Extracts hash from URL using regex: /^[a-f0-9]{64}$/i
// 2. Checks user's kind 10063 servers
// 3. Tries each server with GET /<hash>
// 4. Returns first working URL
// 5. Falls back to searching Nostr events with #x tags
```

### 4. Managing User's Blossom Servers

```typescript
// Get or create user's server list
let serverList = await blossom.getServerList(user);

if (!serverList) {
  // Create new list using NDKBlossomList from NDK
  const event = new NDKEvent(ndk);

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [nostr-dev-kit/ndk](https://github.com/nostr-dev-kit/ndk) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-26 -->
