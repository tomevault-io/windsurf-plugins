---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

`ostrio:files` is an Atmosphere package for Meteor 3.2 or newer (`ostrio:cookies` needs `fetch@0.1.6`, first shipped in 3.2) that uploads files from the client to the server and serves them back. It is published from this repo root through `package.js`, not through npm. `package.json` holds only dev tooling. Runtime npm deps go in `Npm.depends` inside `package.js`, and both the `onUse` and `onTest` blocks must declare them.

## Commands

```shell
meteor npm install
npm run lint                 # ESLint 9 flat config (eslint.config.mjs)
npm run typecheck            # tsc --noEmit -p . && tsd (index.test-d.ts)
npm run test:mocha           # meteor test-packages ./ --driver-package meteortesting:mocha --once
npm run test:mocha:watch     # same, re-runs on change
npm run test:mocha -- --port 3456              # use another port when 3000 is busy
MOCHA_GREP="_checkAccess" npm run test:mocha   # run tests whose name matches (works with meteortesting:mocha)
meteor test-packages ./ --driver-package meteortesting:mocha --once --release 3.5.2   # pin a Meteor release
```

Tests are server-only mocha (`chai`, `sinon`). `tests/server.js` is the entry point and imports these files. Add a new test file there.

- `core.test.js`: `FilesCollectionCore`, `link()`, `formatFileURL`
- `cursor.test.js`: `FileCursor` and `FilesCursor`, including the server-side sync method errors
- `helpers.test.js`: `lib.js` helpers
- `server.test.js`: server API, `_checkAccess`, `loadAsync`, `writeAsync`, `serve()`, `WriteStream`
- `security.test.js`: upload ownership, path and file identity checks, size limits, Range, HTTP errors, idempotent EOF

Run all three of lint, typecheck, and tests before finishing. `.versions` is not updated by `test-packages`. When dependencies change, regenerate it from a throwaway app (`meteor create --bare /tmp/x`, add `ostrio:files` and `meteortesting:mocha` with `METEOR_PACKAGE_DIRS` pointing at this repo's parent, copy the resolved versions, drop the `ostrio:files` line). `meteor publish` also rewrites it.

## Architecture

The package entry points differ by side. `server.js` and `client.js` each export a `FilesCollection` that extends `FilesCollectionCore` (`core.js`). The core holds the isomorphic parts: `find`/`findOneAsync`, `link()`, `_dataToSchema`, `_getExt`, and a schema built on top of a regular `Mongo.Collection` (`this.collection`).

Upload flow (client → server):
1. Client `insert()`/`insertAsync()` creates an `UploadInstance` (`upload.js`). It reads the file in `chunkSize` slices, optionally through a Web Worker (`worker.min.js`, also inlined as a blob in `client.js`). The `FileUpload` object returned to user code wraps the instance with reactive `progress`/`state` and `pause`/`continue`/`abort`.
2. There are two transports, picked by `transport: 'ddp' | 'http'`:
   - DDP: Meteor methods named in `_methodNames` (`_FilesCollectionStart_<name>`, `_Write_`, `_Abort_`, `_Remove_`).
   - HTTP: POST to `${downloadRoute}/${collectionName}/__upload` with `x-start`, `x-eof`, `x-chunkid`, `x-fileid`, and `x-mtok` headers, handled in the `WebApp` middleware in `server.js`.
3. Both transports converge in `_startUpload` (which calls `_prepareUpload`) and `_writeUpload` (server). An in-progress upload lives in `_preCollection` (TTL index, `continueUploadTTL`) and in `_currentUploads[fileId]`, a `WriteStream` (`write-stream.js`) that writes chunks at offset `(chunkId-1)*chunkSize` into one file handle. On EOF, `_finishUpload` inserts the final document into `this.collection` and emits `afterUpload`.

Download flow: the same `WebApp` middleware matches `${downloadRoute}/${collectionName}/${_id}/${version}/${name}` (or the `public` variant). Then `_checkAccess` runs (the `protected` option), then `download()`, then `serve()`, which handles Range/206/416, `responseHeaders`, and `Content-Disposition`, and streams with `fs.createReadStream`. Hooks along this path: `interceptRequest`, `interceptDownload`, `downloadCallback`.

Auth on HTTP routes: by default the client sets an `x_mtok` cookie (via `ostrio:cookies`) to `Meteor.connection._lastSessionId`. The server maps it to a userId through `Meteor.server.sessions` (`_getUserId`, `_getUserDefault`). That lookup is in-process memory, so multi-instance deployments need sticky sessions. `config.getUser` replaces this lookup.

Storage is the filesystem only. Files live under `storagePath` (default `assets/app/uploads/<collectionName>`). Each document's `versions.<name>.path` points at a file on disk. Integrations with 3rd-party storage (S3, GridFS, and others) are recipes in `docs/` that move files in `onAfterUpload` and serve them through `interceptDownload`.

`cursor.js`: `FileCursor` wraps one document, `FilesCursor` wraps a Mongo cursor. On the server only the `*Async` methods work, because Meteor 3 server cursors are async. The sync methods throw there.

`lib.js`: shared `helpers` (type checks, `clone`, `sanitize`), `fixJSONParse`/`fixJSONStringify` (Date round-trip for `meta` over HTTP), and `formatFileURL`.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [veliovgroup/Meteor-Files](https://github.com/veliovgroup/Meteor-Files) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-07 -->
