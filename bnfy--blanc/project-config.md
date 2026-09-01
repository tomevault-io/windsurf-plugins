---
trigger: always_on
description: This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.
---

# AGENTS.md

This file provides guidance to Codex (Codex.ai/code) when working with code in this repository.

## What this is

Blanc — a minimal Electron browser shell: custom-drawn "Island" chrome (a single floating command pill replacing the traditional tab strip + toolbar, per the Bowser Design System's Island Chrome handoff) with ad/tracker blocking wired in at the network layer, independent of Chrome's extension store and Manifest V3's `declarativeNetRequest` limits. Plus favorites, history, downloads, settings, slash commands, a Quick Switcher, private tabs, an explicit permission policy, and packaging config.

The app was renamed from "Bowser" to Blanc in July 2026 — its former name was inseparable from Nintendo's Mario villain, which didn't fit a serious indie browser's ambitions. `build.appId` in `package.json` deliberately stayed `me.bnfy.bowser` through the rename (only `name`/`productName` changed) to preserve macOS Gatekeeper/notarization identity and the auto-update chain for existing installs — don't "clean up" that mismatch, it's intentional. The design system's project name ("Bowser Design System", referenced throughout this file) is a separate Codex Design project the user maintains and wasn't renamed alongside the app; leave those references as-is.

**Current public baseline (Aug 31, 2026): v1.11.0.** It was published from `e3ab5b6` after the complete macOS arm64 signing/notarization, Windows exact-publisher/timestamp Authenticode, Linux/Windows packaged-payload and hardened-fuse, authenticated checksum-manifest, SBOM, provenance, and logged-out download gates passed. It adds the complete Mahjong start-page layout and supporting game UX, local crash diagnostics and recovery choices, dedicated certificate-failure UX, the revised Blanc mark, a verified precompiled blocker seed, packaged MIT/compliance material, privacy-bounded optional product-use events, Electron 44.1.0, and Windows updater-verification fixes. The owner installed the private signed Windows candidate and confirmed it worked. Manual Linux runtime testing was explicitly waived because the available Parallels Desktop setup is unreliable, with the remaining OS-specific rendering/input/runtime risk accepted after the automated Linux gates passed; the authenticated public AppImage subsequently passed a fresh Ubuntu download/digest/attestation/launch/render check with Blanc chrome, overlay, new-tab targets, and a rendered `v1.11.0` marker. The real public v1.10.0 → v1.11.0 updater handoffs passed on macOS and Windows through in-app discovery/download and the real **Restart Now** action; macOS additionally passed strict post-update signature and Gatekeeper checks. All three v1.11.0 platform follow-ups are complete. Its 48-hour launch soak remains open, but may be intentionally superseded by the next approved release. The owner expects more releases before official launch week, so do not recapture launch demo/gallery media for v1.11.0; refresh release-bound media once, from the final selected launch release, after its required evidence is complete. Evidence lives in `docs/release-incidents/2026-08-31-v1.11.0.md`; the complete v1.10.0 record remains in `docs/release-incidents/2026-08-29-v1.10.0.md`. A directly launched installer is not an updater-handoff test — the handoff must begin inside the old packaged Blanc, discover the public updater metadata, download the matching artifact, and invoke Restart Now.

**Open-source baseline (Aug 30, 2026): MIT is the deliberate direction.**
Blanc is open source, not merely source-available. Commit `b49261a` adopted the
MIT License for Bananify Creative-owned software, documentation, and media and
aligned the repository, FAQ, and Terms; the canonical site was deployed from
that exact commit. Do not reintroduce `UNLICENSED`, "source-available, not open
source", or language forbidding modification or redistribution unless the owner
explicitly reverses this decision. Open source is part of Blanc's trust,
adoption, auditability, and grant strategy. Patron should be framed as support
for Blanc and the best official experience, not as a source-code restriction.

The grant is intentionally bounded. The Blanc and Bananify Creative names,
logos, and other identity assets listed in `ASSET-LICENSE.md` remain reserved;
MIT does not grant trademark rights. Upstream files and third-party material
retain their own terms, recorded in `THIRD-PARTY-NOTICES.md` and the packaged
`src/THIRD_PARTY_NOTICES.txt`, including EasyList/EasyPrivacy, 1Password, Inter,
JetBrains Mono, and Lucide. Never describe the whole repository as blanket MIT
or remove those carve-outs. Public v1.11.0 is the first packaged release under
this licensing baseline. It carries `LICENSE`, both notice files,
`ASSET-LICENSE.md`, both OFL texts, and the pinned EasyList/EasyPrivacy sources
inside `app.asar`, as required by `package.json` and the packaged compliance
gate.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bnfy/blanc](https://github.com/bnfy/blanc) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
