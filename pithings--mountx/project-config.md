---
trigger: always_on
description: Keep important information about project in AGENTS.md. For more detailed info, progressively document them in .agents/*.md and reference from this file.
---

Keep important information about project in AGENTS.md. For more detailed info, progressively document them in .agents/*.md and reference from this file.

# mountx

Mount a JavaScript filesystem: one driver interface (a subset of `node:fs/promises`), multiple transports (FUSE first, then NFSv3). User-facing docs live in `docs/` (<https://mountx.vercel.app>); `README.md` is the landing page for npm and GitHub and links there.

Conventions: pure JS/TS, zero runtime deps, pure-JS-first. Single package with subpath exports. Small conventional commits to `main`, tests green (`pnpm test`) before each commit.

There is exactly one piece of native code, and it exists for one reason: unprivileged FUSE mounting needs `fusermount3` to hand `/dev/fuse` back over `SCM_RIGHTS`, and Node cannot `recvmsg` a descriptor. See `native/` and the invariants below.

## Code map

Core (`src/`):

- `types.ts` — `FsDriver` (a subset of `node:fs/promises`; `node:fs/promises` itself is assignable to it), `FsCapabilities`, the typed-only `mountx.*` extension namespace, `StatsLike`/`DirentLike`/`FileHandleLike`.
- `errors.ts` — `ERRNO_CODES` (Linux values), `fsError()` producing errors byte-identical to `node:fs`'s, `errnoOf()` for transports.
- `path.ts` — absolute POSIX path helpers; `..` clamps at the root.
- `harness.ts` — `createLoopback(driver)`: normalizes paths, fills missing methods with `ENOSYS`, resolves capabilities. What driver authors test against.
- `lock.ts` — `PathLock`, the writer lock over the path map, shared by the FUSE and NFS sessions (`RENAME` takes it; `READ`/`WRITE` run outside it).
- `auto.ts` — exported as `mountx/auto`: `probeTransports()` and a `mount()` that picks FUSE where FUSE works (root _or_ `fusermount3`) and NFS otherwise, which is what macOS gets — root or not, since NFS needs none there. Both transports arrive via `await import()`, and the probe reaches only `nfs/probe.ts` and `fuse/fusermount.ts`, so choosing one never loads the other's codec. The result is the transport's own mount object with a `transport` discriminant defined on it — tagged, not wrapped. No fallback after a failure, and no probe when a transport is named: both are deliberate, see the module docs.
- `drivers/memory.ts`, `drivers/node-fs.ts` — the two v1 drivers; the passthrough resolves every path component itself so nothing escapes its root.
- `drivers/handle.ts` — the parts of a `FileHandleLike` that are identical in every driver holding its own bytes, and shared by `memory` and `unstorage`: `parseOpenFlags()` (both `node:fs` flag namespaces), `validateRange()`/`validatePosition()` (the `ERR_OUT_OF_RANGE` rejections `node:fs` makes), and `resizeBytes()` (geometric growth, `EFBIG` by asking the engine rather than assuming a cap). Pure; touches no filesystem.
- `drivers/unstorage.ts` — exported as `mountx/drivers/unstorage`: an `FsDriver` over an [unstorage](https://unstorage.unjs.io) `Storage`, which makes every unstorage driver mountable. `unstorage` is an **optional peer dependency imported for types only** — the built `dist/drivers/unstorage.mjs` has no import of it, so zero-runtime-deps holds. Path `/a/b` is the key `a:b` (unstorage's own `normalizeKey` already maps `/` onto `:`); directories are key prefixes, and an empty one exists only in this process because a marker key would be a file to every other consumer of the store. Random access is buffered per path — one shared `OpenFile` per open path, written back on `fsync` and last `close` — because `Storage` has no partial read or write. Permissions and timestamps are an in-memory overlay seeded from `getMeta`. `:`/`?`/trailing-`$` in a name answer `EINVAL` rather than corrupting quietly (`normalizeKey` eats all three). No symlinks, hardlinks or `statfs`, and `rename` is copy-then-delete, all declared.

FUSE (`src/fuse/`, exported as `mountx/fuse`):

- `constants.ts` — opcodes and `FUSE_*`/`FOPEN_*`/`FATTR_*`/`DT_*`, transcribed from the kernel's `include/uapi/linux/fuse.h` (tag v6.12, protocol 7.41).
- `protocol.ts` — every struct encoded **and** decoded, opcode dispatch table (`OPCODES`), message framing, errno-on-the-wire helpers, dirent packing (`DirentPacker`).
- `init.ts` — `negotiateInit(kernelInit, preferences)`, pure.
- `flags.ts` — the two `open(2)` flag namespaces, pure: `driverOpenFlags()` turns the kernel's `O_*` into the host's for the hand-off to a driver (the identity on Linux, where the wire _is_ the host, so unnamed bits survive), and `reopenFlags()` drops the one-shot creation flags a `handles: false` re-open must not repeat. The translation exists because Tier-0 tests drive a real session on whatever host runs `pnpm test`, and macOS's `O_TRUNC` is Linux's `O_APPEND`.
- `session.ts` — `FuseSession(driver, options)`: `INIT` handshake, opcode switch, file-handle table, readdir paging, `SETATTR` bitmask → driver calls, notify encoders.
- `inodes.ts` — `InodeTable`: nodeid ↔ path ↔ `(dev, ino)`, lookup refcounting, subtree remap on rename, orphans. Entirely synchronous.
- `notify.ts` — `notify_inval_inode`/`notify_inval_entry` codecs.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [pithings/mountx](https://github.com/pithings/mountx) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-28 -->
