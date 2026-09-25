---
trigger: always_on
description: Pure-Python SMB3 server for quicksand host-guest directory mounting.
---

# quicksand-smb

Pure-Python SMB3 server for quicksand host-guest directory mounting.

## Overview

An SMB3 file server with two transports. On macOS and Linux it runs inetd-style, reading/writing on stdin/stdout — QEMU's `guestfwd=cmd:` spawns one instance per guest TCP connection, and no TCP port is opened on the host, eliminating the attack surface that Samba's smbd had. On Windows, `quicksand-core` serves it in-process via `serve_socket` on a persistent loopback-only (127.0.0.1) TCP listener, which avoids requiring Administrator rights.

## Acceptance Criteria

- [x] `mount -t cifs //10.0.2.100/SHARE /mnt -o sec=none,vers=3.0` succeeds from Ubuntu guest
- [x] `ls`, `cat`, `stat` work on mounted files (read path)
- [x] `echo "data" > file`, `mkdir`, `rm`, `mv` work (write path)
- [x] No TCP ports opened on host during operation (macOS/Linux — Windows uses a loopback-only 127.0.0.1 listener)
- [x] Path traversal attacks blocked (symlink escapes, `../` escapes)
- [x] Dynamic mounts work (add shares after VM boot via config file reload)
- [x] Read-only shares enforce read-only (writes fail with permission denied)
- [x] Multiple concurrent shares work
- [x] Binary files transfer correctly (no newline translation)
- [ ] Large file I/O works (files > 1MB) — tested at protocol level, not yet in VM
- [ ] Alpine guest mount validation

## Key Protocol Details

### NTLMSSP Authentication (sec=none still requires it)

The Linux kernel CIFS client **always** performs a full NTLMSSP exchange, even with `sec=none`:

1. **NEGOTIATE**: Server sends SPNEGO `negTokenInit` offering NTLMSSP OID
2. **SESSION_SETUP round 1**: Client sends raw NTLMSSP_NEGOTIATE → Server sends **raw NTLMSSP_CHALLENGE** (NOT SPNEGO-wrapped)
3. **SESSION_SETUP round 2**: Client sends raw NTLMSSP_AUTH → Server returns STATUS_SUCCESS

**Critical**: The SMB2/3 kernel code uses `RawNTLMSSP` — it does NOT unwrap SPNEGO from SESSION_SETUP responses. SPNEGO is only used in the NEGOTIATE response. Wrapping the NTLMSSP_CHALLENGE in SPNEGO causes `error(22): Invalid argument`.

### SecurityMode MUST be non-zero

Per MS-SMB2 Section 3.3.5.4, the NEGOTIATE response **MUST** have `SMB2_NEGOTIATE_SIGNING_ENABLED (0x0001)` set. Setting SecurityMode=0 causes the client to silently disconnect after NEGOTIATE.

### IPC$ Share Required

The CIFS client always connects to `IPC$` before the real share (for IOCTL validation). The server must accept `IPC$` as a valid share with `ShareType=PIPE (0x02)`.

### Compound Related Operations

The CIFS client sends compound requests (CREATE+QUERY_INFO+CLOSE) with `SMB2_FLAGS_RELATED_OPERATIONS (0x04)`. In related operations, the FileId in QUERY_INFO/CLOSE is `0xFFFFFFFF...` meaning "use the FileId from the previous CREATE response". The server must track the last FileId and substitute it.

### FILE_ALL_INFORMATION Struct Alignment

The composite `FileAllInformation` response requires exact field sizes:
- `FileBasicInformation`: **40 bytes** (4 timestamps + attributes + 4-byte reserved)
- `FileStandardInformation`: **24 bytes** (2 sizes + nlinks + 2 bools + 2-byte reserved)

Missing the reserved padding fields causes `get root inode failed`.

### FSCTL_VALIDATE_NEGOTIATE_INFO

The IOCTL response body is **48 bytes** (not 44 — includes a Reserved2 field). The OutputOffset must account for the full 48-byte body. Off-by-4 causes `buffer length N smaller than minimum size 28`.

### QUERY_DIRECTORY Info Classes and Reserved Fields

The Linux kernel CIFS client uses `FILE_ID_FULL_DIRECTORY_INFORMATION` (info class 38) for readdir — NOT class 3 or 37 as you might expect. Every directory info struct has reserved/padding fields that must be included or the FileName offset shifts and filenames appear truncated.

Critical struct sizes (fixed header before FileName):
- `FILE_DIRECTORY_INFORMATION` (1): **64 bytes**
- `FILE_BOTH_DIRECTORY_INFORMATION` (3): **94 bytes** (includes ShortNameLength(1) + Reserved(1) + ShortName(24))
- `FILE_ID_BOTH_DIRECTORY_INFORMATION` (37): **104 bytes** (adds Reserved2(2) + FileId(8))
- `FILE_ID_FULL_DIRECTORY_INFORMATION` (38): **80 bytes** (includes EaSize(4) + **Reserved(4)** + FileId(8))

The Reserved(4) field in class 38 between EaSize and FileId is easy to miss — omitting it shifts FileName by 4 bytes (2 UTF-16LE chars), causing filenames like `README.md` to appear as `ADME.md`.

### Dynamic Mount Config Reload

The CIFS client reuses existing TCP connections for new mounts. The server process reads the config file once at startup, so `reload_config()` must re-read the config file on each TREE_CONNECT to pick up dynamically added shares.

## Mandatory Verification Process

When adding or modifying any SMB struct (directory info, file info, IOCTL, etc.):

1. **Field-by-field spec check**: Open the MS-FSCC or MS-SMB2 spec section for the struct. List every field and its size. Compare against the `struct.pack` format string. Count total bytes with `struct.calcsize()`. Reserved/padding fields with value 0 are still required on the wire.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [microsoft/quicksand](https://github.com/microsoft/quicksand) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-25 -->
