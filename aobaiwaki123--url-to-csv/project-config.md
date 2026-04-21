---
trigger: always_on
description: Shared utility functions and helper modules
---


# Shared Utilities & Helper Functions

The [header-utils.js](mdc:header-utils.js) provides common utility functions for HTTP header processing and other shared functionality across the image URL collection toolkit.

## Utility Module Overview
- **Purpose**: Centralized helper functions for HTTP operations and data processing
- **Architecture**: Pure JavaScript module with no external dependencies
- **Target**: Shared functionality between Net2Sheet, CSV Checker, and URL to CSV tools

## HTTP Header Processing

### Header Parsing Utilities
```javascript
// Parse HTTP headers from response objects
const parseHeaders = (headerString) => {
  const headers = new Map();
  if (!headerString) return headers;
  
  headerString.split('\r\n').forEach(line => {
    const colonIndex = line.indexOf(':');
    if (colonIndex > 0) {
      const name = line.substring(0, colonIndex).trim().toLowerCase();
      const value = line.substring(colonIndex + 1).trim();
      headers.set(name, value);
    }
  });
  
  return headers;
};

// Extract content type from headers
const getContentType = (headers) => {
  const contentType = headers.get('content-type') || '';
  return contentType.split(';')[0].trim().toLowerCase();
};

// Check if response contains image content
const isImageContentType = (contentType) => {
  const imageTypes = new Set([
    'image/png', 'image/jpeg', 'image/jpg', 'image/gif',
    'image/webp', 'image/svg+xml', 'image/avif',
    'image/bmp', 'image/x-icon', 'image/vnd.microsoft.icon'
  ]);
  return imageTypes.has(contentType);
};
```

### Request Enhancement
```javascript
// Add standard headers for web scraping
const enhanceRequestHeaders = (baseHeaders = {}) => {
  return {
    'User-Agent': 'Mozilla/5.0 (compatible; ImageURLCollector/1.0)',
    'Accept': 'text/html,application/xhtml+xml,application/xml;q=0.9,image/*;q=0.8,*/*;q=0.7',
    'Accept-Language': 'ja,en-US;q=0.9,en;q=0.8',
    'Accept-Encoding': 'gzip, deflate, br',
    'DNT': '1',
    'Upgrade-Insecure-Requests': '1',
    ...baseHeaders
  };
};

// Timeout wrapper for fetch requests
const fetchWithTimeout = async (url, options = {}, timeoutMs = 10000) => {
  const controller = new AbortController();
  const timeoutId = setTimeout(() => controller.abort(), timeoutMs);
  
  try {
    const response = await fetch(url, {
      ...options,
      signal: controller.signal,
      headers: enhanceRequestHeaders(options.headers)
    });
    clearTimeout(timeoutId);
    return response;
  } catch (error) {
    clearTimeout(timeoutId);
    if (error.name === 'AbortError') {
      throw new Error(`Request timeout after ${timeoutMs}ms`);
    }
    throw error;
  }
};
```

## URL Processing Utilities

### URL Validation & Normalization
```javascript
// Comprehensive URL validation
const validateUrl = (urlString) => {
  try {
    const url = new URL(urlString);
    
    // Check for supported protocols
    if (!['http:', 'https:'].includes(url.protocol)) {
      return { valid: false, error: 'Unsupported protocol' };
    }
    
    // Check for suspicious patterns
    if (url.hostname.includes('localhost') || url.hostname.startsWith('127.')) {
      return { valid: false, error: 'Local URLs not supported' };
    }
    
    return { valid: true, url };
  } catch (error) {
    return { valid: false, error: error.message };
  }
};

// Convert relative URLs to absolute
const resolveUrl = (relativeUrl, baseUrl) => {
  try {
    return new URL(relativeUrl, baseUrl).href;
  } catch {
    return null;
  }
};

// Extract domain information
const getDomainInfo = (url) => {
  try {
    const urlObj = new URL(url);
    return {
      protocol: urlObj.protocol,
      hostname: urlObj.hostname,
      port: urlObj.port || (urlObj.protocol === 'https:' ? '443' : '80'),
      domain: urlObj.hostname.replace(/^www\./, ''),
      subdomain: urlObj.hostname.split('.').slice(0, -2).join('.')
    };
  } catch {
    return null;
  }
};
```

## Data Processing Helpers

### Array & Object Utilities
```javascript
// Remove duplicates while preserving order
const uniqueBy = (array, keyFunction) => {
  const seen = new Set();
  return array.filter(item => {
    const key = keyFunction(item);
    if (seen.has(key)) return false;
    seen.add(key);
    return true;
  });
};

// Chunk array into smaller batches
const chunkArray = (array, chunkSize) => {
  const chunks = [];
  for (let i = 0; i < array.length; i += chunkSize) {
    chunks.push(array.slice(i, i + chunkSize));
  }
  return chunks;
};

// Deep clone with JSON fallback
const deepClone = (obj) => {
  try {
    return structuredClone(obj);
  } catch {
    // Fallback for older browsers
    return JSON.parse(JSON.stringify(obj));
  }
};
```

### String Processing
```javascript
// Safe filename generation from URLs
const sanitizeFilename = (filename) => {
  return filename
    .replace(/[<>:"/\\|?*\x00-\x1f]/g, '_') // Remove invalid chars
    .replace(/^\.+/, '_') // No leading dots
    .replace(/\.+$/, '') // No trailing dots
    .substring(0, 255); // Limit length
};

// Extract file extension with validation
const getFileExtension = (filename) => {
  const lastDot = filename.lastIndexOf('.');
  if (lastDot === -1 || lastDot === filename.length - 1) return '';
  return filename.substring(lastDot).toLowerCase();
};


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AobaIwaki123) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
