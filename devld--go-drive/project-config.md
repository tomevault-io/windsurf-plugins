---
trigger: always_on
description: This file guides AI agents and human developers who add or modify storage adapters under `script-drives/`. The goal is to make a remote storage service behave like a go-drive directory tree by adding JavaScript files only, without recompiling go-drive.
---

# go-drive JavaScript Drive Adapter Guide

This file guides AI agents and human developers who add or modify storage adapters under `script-drives/`. The goal is to make a remote storage service behave like a go-drive directory tree by adding JavaScript files only, without recompiling go-drive.

## 1. Decide whether a script Drive is appropriate

### Good candidates

Prefer a script Drive when the service meets most of these conditions:

- It provides a stable HTTP/HTTPS REST API.
- Files and directories can be represented by a path, type, size, and modification time.
- Listing, uploading, downloading, directory creation, and deletion are available over HTTP.
- Authentication uses an API key, bearer token, HMAC signature, or OAuth 2.0.
- It does not require a Node.js package, native vendor SDK, dynamic library, or operating-system command.
- Each API operation can finish synchronously, or an asynchronous operation can be polled until completion.
- Large files can be streamed, uploaded in parts, or uploaded directly from the browser.

Typical candidates include file APIs such as Dropbox, object-storage APIs such as Qiniu, self-hosted HTTP file services, and cloud drives that have a complete REST API but no built-in go-drive implementation.

### Technically possible, but usually not worthwhile

- WebDAV is HTTP-based, but the runtime has no DOM or XML parser. Use the built-in WebDAV Drive unless responses are exceptionally small and stable.
- S3-compatible storage can be signed with `encUtils`, but the built-in S3 Drive handles regions, multipart uploads, and compatibility differences more reliably.
- A service exposed only through a vendor JavaScript SDK is viable only if the SDK can be rewritten as ES5 without Node.js or DOM dependencies. Calling the REST API directly is usually better.
- Long-polling asynchronous APIs can work, but occupy a VM while polling. Check `ctx.Err()` and apply a timeout.

### Poor candidates

Implement these as Go Drives, or use an existing built-in Drive:

- Samba/SMB/CIFS, SFTP, and FTP require raw TCP, session negotiation, connection reuse, or binary protocols. The script runtime has no socket API. SMB also requires negotiation, signing, encryption, and stateful handles; `http()` cannot substitute for it.
- Local filesystems, FUSE, block devices, and tape systems require operating-system or device access.
- SDKs that require native libraries, external commands, Node.js `require`, `Buffer`, streams, or npm packages.
- Services that require WebSocket, HTTP/2-specific flow control, client certificates, or a custom transport stack without an equivalent ordinary HTTP API.
- Workloads requiring heavy CPU processing, complex compression/encryption, or large in-memory buffers. The ES5 interpreter is not designed for them.
- Services that cannot reliably list a hierarchy, read file contents, or expose stable paths.

Rule of thumb: use a script Drive when the core task is “construct HTTP requests and map JSON to Entry objects.” Use Go when the core task is “implement a transport protocol, integrate with the operating system, or reuse a native SDK.”

## 2. Sources of truth to read before editing

Check these sources in order. Do not rely only on old adapter examples:

1. `docs/scripts/env/drive.d.ts` — Drive lifecycle, interfaces, and Drive-specific APIs.
2. `docs/scripts/global.d.ts` — global HTTP, IO, error, encoding, path, and form APIs.
3. `drive/script/helper.js` — required methods, method binding, and the actual behavior of `$` shared properties.
4. `drive/script/index.go` and `drive/script/utils.go` — Go/JavaScript value conversion and resource ownership.
5. `script-drives/dropbox.js` — OAuth, pagination, streaming uploads, and temporary download URLs.
6. `script-drives/qiniu.js` and `qiniu-uploader.js` — HMAC signing, object storage, and direct browser uploads.
7. `docs/drive-uploaders/types.d.ts` — required when implementing direct browser uploads.

Type declarations assist development; the Go bridge is the final source of truth. If the declarations and implementation disagree, correct the declaration or documentation instead of inventing an API.

## 3. Deliverables and file conventions

The server-side adapter is:

```text
script-drives/<name>.js
```

The optional browser-side direct uploader is:

```text
script-drives/<name>-uploader.js
```

Use a stable, short, lowercase identifier for `<name>`. Files with the same base name form one extension. A server script must begin with consecutive `//` metadata lines:

```js
// Example Cloud
// Example Cloud REST API adapter.
//
// Create an API token with file read/write permissions.

/// <reference path="../docs/scripts/env/drive.d.ts"/>
```

- The first line is the display name shown in the UI.
- Following `//` lines, up to the empty comment line, are the Markdown description.
- The `reference` directive provides editor completion only; it does not change runtime behavior.
- After saving a script, create or reload the Drive from the administration UI.

## 4. Runtime constraints

### JavaScript version


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [devld/go-drive](https://github.com/devld/go-drive) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-09 -->
