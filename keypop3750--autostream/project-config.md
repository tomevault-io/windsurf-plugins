---
trigger: always_on
description: Provides secondary stream option with smart resolution targeting:
---

# AutoStream V3 - AI Coding Instructions

AutoStream is a **high-performance Stremio addon** that intelligently aggregates and selects the best streaming sources with **universal debrid provider support** and **click-time resolution**. The system follows a **layered service architecture** with **comprehensive security protections** and **platform-specific optimization**.

## 🏗️ Architecture Overview

- **Server Layer** (`server.js`): Main HTTP server with defensive coding, memory monitoring, universal debrid resolution, and device-aware request handling
- **Services Layer** (`services/`): Stream aggregation (`sources.js`), universal debrid integration (`debrid.js`), reliability tracking (`penaltyReliability.js`), enhanced metadata with ID validation (`enhanced_meta.js`)
- **Core Layer** (`core/`): Universal debrid providers (`debridProviders.js`), platform-specific scoring (`scoring_v6.js`), content formatting (`format.js`), series caching (`series-cache.js`)
- **UI Layer** (`ui/`): Configuration interface with client-side state management for all 8 debrid providers
- **Utils Layer** (`utils/`): TTL caches (`cache.js`), HTTP utilities (`http.js`), ID correction systems (`id-correction.js`)

## 🔑 Critical Security Architecture

### 1. Universal Debrid Provider System
**NEW CRITICAL PATTERN**: Support 8 debrid providers equally, not AllDebrid-centric:
```js
// core/debridProviders.js - Universal provider configuration
const DEBRID_PROVIDERS = {
  realdebrid: { key: 'realdebrid', name: 'RealDebrid', shortName: 'RD' },
  alldebrid: { key: 'alldebrid', name: 'AllDebrid', shortName: 'AD' },
  premiumize: { key: 'premiumize', name: 'Premiumize', shortName: 'PM' },
  torbox: { key: 'torbox', name: 'TorBox', shortName: 'TB' },
  // ... 8 total providers
};

// Server detects and validates ANY provider
const workingProviders = [];
for (const [key, token] of Object.entries(providerConfig)) {
  const isWorking = await validateDebridKey(key, token);
  if (isWorking) workingProviders.push({ key, token, provider: getProvider(key) });
}
```

### 2. Security-First Torrent Handling
**CRITICAL SECURITY**: No magnet URLs served when debrid is configured:
```js
// Step 3: Convert ALL torrents to debrid URLs when ANY provider configured
if (hasDebridConfigured && selectedStreams.length > 0) {
  for (const s of selectedStreams) {
    if ((s.autostreamOrigin === 'torrentio' || s.autostreamOrigin === 'tpb') && s.infoHash) {
      s.url = buildPlayUrl({...}, { provider: effectiveDebridProvider, token: effectiveDebridToken });
      
      // SECURITY: Remove streams that fail conversion rather than serve raw magnets
      if (!s.url || /^magnet:/i.test(s.url)) {
        s._invalid = true; // Filtered out entirely
      }
    }
  }
}
```
**Result**: Zero P2P activity when debrid configured - ISPs only see HTTPS requests to debrid services.

### 3. Environment Variable Security Protection
**CRITICAL**: Prevent credential leakage across all providers:
```js
// services/debrid.js - Security check on startup
const dangerousEnvVars = [
  'AD_KEY', 'ALLDEBRID_KEY', 'RD_KEY', 'REALDEBRID_KEY', 
  'PM_KEY', 'PREMIUMIZE_KEY', 'TB_KEY', 'TORBOX_KEY',
  // ... all 8 providers protected
];
foundVars.forEach(varName => {
  delete process.env[varName];
  console.error(`🚨 Removed dangerous environment variable: ${varName}`);
});
```

## 🔧 Essential Development Patterns

### 1. Graceful Module Loading with Fallbacks
**Always provide fallback functions** to prevent crashes when optional modules are missing:
```js
const scoringMod = (() => {
  try { return require('./core/scoring_v6'); }
  catch (e1) { return { filterAndScoreStreams: (streams) => streams.slice(0,2) }; }
})();
```

### 2. Universal Debrid Rate Limiting
**ALL debrid providers use unified rate limiting**:
```js
// services/debrid.js - Universal rate limiter for all providers
class DebridRateLimiter {
  constructor() {
    this.maxRequestsPerMinute = 30; // Conservative limit for debrid APIs
    this.maxCacheSize = 200; // Memory leak prevention
  }
  async checkRateLimit(apiKey) { /* Universal rate limiting logic */ }
}

// services/debrid.js - Universal circuit breaker
class DebridCircuitBreaker {
  async checkCircuit(apiKey) {
    if (failures.count >= this.maxFailures) {
      throw new Error('Debrid API circuit breaker is open. Service temporarily unavailable.');
    }
  }
}
```

### 3. Platform-Specific Scoring System (V6)
**CRITICAL PATTERN**: Device-aware scoring for optimal compatibility:
```js
// core/scoring_v6.js - Device detection and optimization
function detectDeviceType(req) {
  const userAgent = req.headers['user-agent'] || '';
  if (/\b(smart[-\s]?tv|tizen|webos|roku)\b/i.test(userAgent)) return 'tv';
  if (/\b(android|iphone|mobile)\b/i.test(userAgent)) return 'mobile';
  return 'web';
}

// TV scoring prioritizes compatibility over quality
function getTVQualityScore(title, factors) {
  if (/\b(x265|hevc)\b/.test(title)) score -= 60; // Heavy codec penalty for TV
  if (/\b(x264|h\.?264)\b/.test(title)) score += 40; // Compatibility bonus
}
```

### 4. Memory Management with Size Limits
**ALL Map/Cache structures MUST have size limits**:
```js
class DebridRateLimiter {
  cleanup() {
    if (this.requests.size > this.maxCacheSize) {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [keypop3750/AutoStream](https://github.com/keypop3750/AutoStream) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
