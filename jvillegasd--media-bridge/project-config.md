---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Commands

```bash
# Development build with watch mode (rebuilds on file changes)
npm run dev

# Production build (minified, no sourcemaps)
npm run build

# TypeScript type checking only (no emit)
npm run type-check
```

There are no tests in this project. After building, load the `dist/` directory as an unpacked extension in Chrome (`chrome://extensions/` → Developer mode → Load unpacked).

## Output Size Limit

All HLS, M3U8, and DASH downloads are processed by the muxer running inside the browser. Output files are limited to approximately **2 GB** — files beyond this will exhaust browser memory during the merge stage.

## Cloud Upload

`src/core/cloud/` contains the full provider abstraction:

- `base-cloud-provider.ts` — abstract `BaseCloudProvider` with `id: CloudProvider` and `upload(blob, filename, onProgress?, signal?): Promise<string>`
- `google-auth.ts` — `GoogleAuth` static class; OAuth via `chrome.identity.launchWebAuthFlow()` with user-provided client ID (stored in `chrome.storage.local` under `google_client_id`). No `oauth2` manifest key needed.
- `google-drive.ts` — `GoogleDriveClient extends BaseCloudProvider`; simple multipart for files ≤ 5 MB, resumable chunked upload for larger files
- `s3-client.ts` — `S3Client extends BaseCloudProvider`; SigV4-signed PUT for files < 10 MB, multipart for ≥ 10 MB. Persists in-flight multipart `uploadId` to `chrome.storage.local` (`s3_pending_uploads`) for crash-resilient cleanup.
- `upload-manager.ts` — `Map<CloudProvider, BaseCloudProvider>` registry; routes `uploadBlob()` through `client.upload()`; `isConfigured()` checks `providers.size > 0`

### Upload Flow

Upload is **manual only**. Completed downloads show an "Upload to cloud" action in both the History page (Options → History → item menu) and the popup Downloads tab. Clicking it opens a file picker (`showOpenFilePicker`); the user selects the local video file, which is stored in IDB (since `chrome.runtime.sendMessage` uses JSON serialization that destroys `ArrayBuffer`), then an `UPLOAD_REQUEST` message is sent to the service worker. If both providers are configured, a provider picker dialog appears. There is no automatic upload on download completion.

Upload progress is tracked via `DownloadStage.UPLOADING` and displayed with a water-fill cloud icon in history. Uploads can be cancelled via an `AbortController`; cancellation prompts a confirmation dialog. On cancel, `AbortError` (DOMException) is expected and handled silently — not logged as an error.

### Crash Resilience

- **S3 multipart**: In-flight `(key, uploadId)` pairs are persisted to `chrome.storage.local`. On service worker startup, `S3Client.cleanupOrphanedUploads()` aborts any orphaned uploads and clears the list.
- **Stuck uploads**: Downloads in `UPLOADING` stage after a crash are restored to `COMPLETED` by `cleanupStaleUploads()` in `init()`.

### Google Drive Setup

Users must create their own OAuth credentials (free). The options page shows step-by-step instructions:
1. Enable Google Drive API in Google Cloud Console
2. Create OAuth 2.0 Client ID (type: **Web application**)
3. Add the extension's redirect URI (`chrome.identity.getRedirectURL()`) as an authorized redirect URI
4. Paste the client ID into the options page
5. Clicking "Sign in with Google" auto-saves settings with `enabled: true`

The `chromiumapp.org` redirect works for unpacked extensions — Chrome intercepts it internally without needing the Chrome Web Store.

### Adding a New Provider

Create a class extending `BaseCloudProvider`, add its key to the `CloudProvider` union in `shared/messages.ts`, instantiate and register it in the `UploadManager` constructor — no other code needs to change.

### S3 Bucket CORS Requirement

The bucket must have a CORS policy whitelisting the extension origin. The options page S3 section generates the correct JSON dynamically (using `chrome.runtime.id`) and provides a "Copy CORS Config" button. Paste it into **S3 → bucket → Permissions → Cross-origin resource sharing (CORS) → Edit**. Without it the browser will block upload requests from the `chrome-extension://` origin.

### S3 Secret Key Encryption

The S3 secret access key can be encrypted at rest with AES-GCM via `SecureStorage`. The user sets a passphrase in the options page; the key is stored as an `EncryptedBlob` in `chrome.storage.local`. On upload, `resolveS3Secret()` in the service worker decrypts it using the passphrase from session storage.

## Architecture

Media Bridge is a Manifest V3 Chrome extension. It has five distinct execution contexts that communicate via `chrome.runtime.sendMessage`:

1. **Service Worker** (`src/service-worker.ts` → `dist/background.js`): The central orchestrator. Handles all download lifecycle management, routes messages from popup/content scripts, maintains download state, and keeps itself alive during long operations using `chrome.runtime.getPlatformInfo()` heartbeats. Intercepts `.m3u8` and `.mpd` network requests via `chrome.webRequest.onCompleted`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jvillegasd/media-bridge](https://github.com/jvillegasd/media-bridge) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
