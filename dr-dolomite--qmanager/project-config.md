---
trigger: always_on
description: **Users**: hobbyist power users + field technicians managing Quectel modems on OpenWRT. Technically literate, not developers. Sessions range from quick checks to focused configuration.
---

## Design Context

**Users**: hobbyist power users + field technicians managing Quectel modems on OpenWRT. Technically literate, not developers. Sessions range from quick checks to focused configuration.

**Brand**: Modern, Approachable, Smart — premium tool that respects user intelligence without requiring modem-engineer knowledge.

**Aesthetic**: Vercel/Linear polish meets Grafana/UniFi density. Light + dark first-class (OKLCH). Euclid Circular B (primary), Manrope (secondary). Radius `0.65rem`. Avoid terminal/legacy/consumer styling.

### Status Badge Pattern
All status badges: `variant="outline"` + semantic color classes + `size-3` lucide icons. Never solid variants.

| State | Classes | Icon |
| ----- | ------- | ---- |
| Success | `bg-success/15 text-success hover:bg-success/20 border-success/30` | `CheckCircle2Icon` |
| Warning | `bg-warning/15 text-warning hover:bg-warning/20 border-warning/30` | `TriangleAlertIcon` |
| Destructive | `bg-destructive/15 text-destructive hover:bg-destructive/20 border-destructive/30` | `XCircleIcon` / `AlertCircleIcon` |
| Info | `bg-info/15 text-info hover:bg-info/20 border-info/30` | Context-specific |
| Muted | `bg-muted/50 text-muted-foreground border-muted-foreground/30` | `MinusCircleIcon` |

Reusable `ServiceStatusBadge` at `components/local-network/service-status-badge.tsx`. Use muted for deliberately inactive states; destructive for failure/error states.

### Design Principles
1. **Data clarity first** — metrics scannable at a glance.
2. **Progressive disclosure** — essentials upfront, advanced controls accessible.
3. **Confidence through feedback** — every action shows loading/success/error.
4. **Consistent** — shadcn/ui + design tokens uniformly, no one-off styles.
5. **Responsive + resilient** — graceful loading/empty/error states, never blank.

### UI Component Conventions
- **CardHeader**: plain `CardTitle` + `CardDescription`, no icons (icons go in badges / action areas).
- **Primary actions**: default variant (not outline). Use `SaveButton` for save actions.
- **Step progress**: `Loader2Icon` spinner + dot indicators. Reserve fill bars for data viz (signal strength, quality meters) only.

## Release Notes (`RELEASE_NOTE.md`)

Sections: `## ✨ New Features`, `## ✅ Improvements`, `## 📥 Installation`, `## 💙 Thank You`. Short user-facing bullets; no internal function names. Headline features first; fixes/polish under Improvements. Include the one-line fresh install command + Software Update upgrade path.

## CGI Endpoint Reference (Additions)

| Feature | CGI Script | Hook | Types | Reboot? |
|---|---|---|---|---|
| Video Optimizer | `network/video_optimizer.sh` | `use-video-optimizer.ts` + `use-cdn-hostlist.ts` | `video-optimizer.ts` | No |
| Traffic Masquerade | `network/video_optimizer.sh` | `use-traffic-masquerade.ts` | `video-optimizer.ts` | No |
| NetBird VPN | `vpn/netbird.sh` | `use-netbird.ts` | inline | Yes (uninstall) |
| Config Backup | `system/config-backup/{collect,apply,apply_status,apply_cancel}.sh` | `use-config-backup.ts` + `use-config-restore.ts` | `config-backup.ts` | Deferred (dialog + banner for IMEI/profile) |

## Feature-Specific Notes

### DPI Settings (Video Optimizer + Traffic Masquerade)
- Routes: `/local-network/video-optimizer` (settings + CDN hostlist), `/local-network/traffic-masquerade`. Old `/local-network/dpi-masking` redirects.
- Binary: `nfqws` from zapret, installed to `/usr/bin/nfqws` on demand by `qmanager_dpi_install` (arch-detect → fetch `openwrt-embedded.tar.gz`). State files: `/tmp/qmanager_dpi_install.{json,pid}`.
- **Single shared nfqws on queue 200** — VO and masquerade are mutually exclusive modes of ONE process: single PID (`/var/run/nfqws.pid`). Backend enforces mutex in `save`/`save_masquerade`; init.d checks masquerade first, then VO.
- **nft rules are persistent**, shipped as `/etc/nftables.d/12-mangle-qmanager-dpi.nft` (chain `mangle_postrouting_qmanager_dpi`, `oifname "rmnet*"`, `queue num 200 bypass`). fw4 sources the file on every load/reload, so rules survive `fw4 reload` (VPN toggles, port-forward edits, mwan3 ipset refreshes, etc. — all of which used to silently wipe the runtime-injected rules). The `bypass` flag means rules are safe to leave permanent even when nfqws is not running. The init.d script never touches nftables — it only manages the daemon. `dpi_helper.sh` no longer has `dpi_insert_rules` / `dpi_remove_rules`. Uninstall removes the .nft file and drops the live chain.
- Modes: VO = SNI split (`split2`) + QUIC desync, filtered by `--hostlist`. Masquerade = fake TLS ClientHello with spoofed SNI (default `speedtest.net`), all traffic.
- Hostlist: `/etc/qmanager/video_domains.txt` (active) + `video_domains_default.txt` (immutable). Hostlist CGI supports GET `?section=hostlist`, POST `save_hostlist`, POST `restore_hostlist`.
- GET handlers gate live stats on UCI `enabled` to avoid cross-mode contamination. Kernel check: `dpi_check_kmod()` reads `/proc/config.gz` for `CONFIG_NETFILTER_NETLINK_QUEUE=y`.
- Boot persistence: enabling either mode → init.d `enable`; disabling → `disable` only if BOTH are off. Uninstall always `disable`s.

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [dr-dolomite/QManager](https://github.com/dr-dolomite/QManager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-11 -->
