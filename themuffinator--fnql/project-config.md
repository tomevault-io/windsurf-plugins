---
trigger: always_on
description: FnQL is Fappin' Quake Live: a modernized Quake Live engine derived from
---

# AGENTS.md

## Mission

FnQL is Fappin' Quake Live: a modernized Quake Live engine derived from
FnQuake3 and guided by the reconstructed Quake Live source in QLSRP. The target
is retail Quake Live compatibility under a legitimate Steam installation. This
repository is engine-only; do not reconstruct or ship game code as part of FnQL.

Every change should protect these project constraints:

1. Retail Quake Live Steam compatibility is the compatibility target.
2. Game-code reconstruction is out of scope. Treat `qagame`, `cgame`, and UI
   code as ABI/reference boundaries unless the user explicitly asks otherwise.
3. QL behavior wins over Quake III behavior when QLSRP or retail evidence shows
   a real engine difference.
4. FnQ3 modernization work remains valuable: keep the renderer, audio, platform,
   packaging, tooling, and test improvements unless they conflict with QL.
5. Speed, determinism, and cross-platform viability matter, but not at the cost
   of compatibility-sensitive behavior.
6. Robustness and cross-platform support must be considered in every
   implementation, including narrow fixes. Prefer resilient error handling,
   explicit fallback behavior, and platform-conscious paths over assumptions
   that only hold on the current development machine.
7. The target engine should load retail Quake Live and interoperate in both
   directions: FnQL clients should join retail-operated protocol-91 servers,
   and retail Quake Live clients should join FnQL-hosted servers. Use the
   legitimate Steam session-ticket path when available. Without Steam, retain
   the retail-shaped fallback and report the remote server's authorization
   decision honestly; never synthesize authentication success.
8. Changes must not regress existing behavior in any aspect. Preserve working
   compatibility, features, performance, determinism, diagnostics, build
   configurations, release behavior, and supported-platform paths. If retail
   Quake Live evidence requires intentionally replacing incompatible behavior,
   document the evidence, scope the change narrowly, and add a regression gate
   for both the corrected behavior and unaffected paths.
9. FnQL is a 32-bit-only project. Build, test, package, and launch only x86
   artifacts (Win32 on Windows and the corresponding 32-bit target elsewhere).
   Do not use x86_64 artifacts as validation and do not ship them.

## Reference Repositories

- `E:\Repositories\FnQuake3`: imported engine baseline. Remote:
  `https://github.com/themuffinator/FnQ3.git`. Import was taken from the local
  working tree on 2026-06-23 at commit
  `91c28d77878302ae67119fc3a29643cc20ce8489`; that source worktree had local
  uncommitted changes at import time.
- `E:\Repositories\QuakeLive-SRP`: QLSRP / Quake Live Source Reconstruction
  Project. Remote: `https://github.com/themuffinator/QL-SRP.git`. Initial
  reference point was commit `94bdd7acdce0c90bf890416e23e704795eac716e`;
  that reference worktree also had local uncommitted changes when this project
  was initialized.
- Retail Quake Live install: use the user's legitimate Steam installation as
  the runtime compatibility target, normally
  `C:\Program Files (x86)\Steam\steamapps\common\Quake Live`.

## Reconstruction Workflow

- Start with static comparison before changing behavior. Compare FnQL against
  QLSRP `src/code/`, the QLSRP reference corpus, and retail-observed behavior.
- Treat QLSRP as behavioral, protocol, file-format, and ABI evidence rather
  than as a source of implementation text. Anything derived from
  `../QuakeLive-SRP/` must be independently rewritten for FnQL: do not perform
  mechanical line-by-line ports, cosmetic renames, or preserve a QLSRP
  implementation structure merely because it already exists there.
- Design each QLSRP-informed rewrite around FnQL's current architecture. Prefer
  small typed interfaces, explicit ownership and bounds, deterministic parsing,
  resilient failure paths, and modern C++ where the surrounding subsystem and
  ABI permit it. Keep C-compatible boundaries where retail modules, legacy
  renderers, or platform APIs require them.
- A rewrite is not complete merely because it matches QLSRP. It must aim to
  gain or preserve compatibility with the legitimate retail Quake Live Steam
  runtime, and it must avoid regressing already-compatible retail behavior.
- Record the retail/QLSRP observation that motivates compatibility-sensitive
  constants and branches, then validate the independently written behavior with
  focused tests, fixture inspection, or a documented retail probe. When QLSRP
  and observed retail behavior disagree, retail behavior is authoritative.
- Keep observed facts separate from inferences in notes, commits, reviews, and
  implementation comments.
- Prefer small compatibility slices: filesystem/search path, Steam install
  discovery, protocol, module ABI, renderer data formats, audio behavior, and
  platform glue should be migrated independently.
- Preserve legacy Quake III license headers and upstream provenance comments.
  Rebrand project-owned code, docs, build outputs, packages, and helper names.
- If a QL feature depends on live online services, keep it explicit and
  default-off until there is a documented open replacement path.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [themuffinator/FnQL](https://github.com/themuffinator/FnQL) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-19 -->
