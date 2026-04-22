---
trigger: always_on
description: Technitium-DNS-Companion is a tool for keeping 2+ Technitium DNS servers in sync. It is designed to be a simple and easy to use, responsive web application that can be used to sync multiple Technitium DNS servers.
---

# Technitium-DNS-Companion

## Summary

Technitium-DNS-Companion is a tool for keeping 2+ Technitium DNS servers in sync. It is designed to be a simple and easy to use, responsive web application that can be used to sync multiple Technitium DNS servers.

## Inquiries/Verbosity

High verbosity is appreciated when suggesting changes or configurations.

Please ask questions of me. I would rather you be more verbose than to make assumptions and changes without understanding my intentions and consulting me first.

## Performance Optimization

**Performance Features:** Backend optimizations including caching, deduplication, and throttling.

See `docs/performance/BACKEND_PERFORMANCE_QUICK_START.md` for implementation details.

## Important Notes

- NEVER put private admin tokens or sensitive information in public repositories or share them publicly.
- NEVER commit real admin tokens or sensitive information to version control.

## Technitium DNS

Project page: https://github.com/TechnitiumSoftware/DnsServer/tree/master
HTTP API documentation: https://github.com/TechnitiumSoftware/DnsServer/blob/master/APIDOCS.md
Advanced Blocking app config: https://github.com/TechnitiumSoftware/DnsServer/blob/master/Apps/AdvancedBlockingApp/dnsApp.config
Docker environment variables: https://github.com/TechnitiumSoftware/DnsServer/blob/master/DockerEnvironmentVariables.md

## Example Environment

This project is designed to work with multiple Technitium DNS servers in various configurations:

- **Supported Versions**: Technitium DNS v13.6.0+
- **Deployment**: Docker or standalone installations
- **Clustering**: Full support for Technitium DNS cluster configurations
  - Automatically detects Primary/Secondary node roles
  - Primary nodes allow write operations
  - Secondary nodes are read-only replicas
- **Network**: Works with static IPs, DHCP, VPN (Tailscale/WireGuard), or public IPs
- **SSL/TLS**: Supports both Let's Encrypt and self-signed certificates
- **Authentication**: Uses Technitium DNS admin tokens for API access

## Technitium-DNS-Companion Cluster Support

**Implementation Status**: ✅ Active (November 8, 2025)

**Backend**:

- `/api/nodes` endpoint returns `isPrimary: true/false` for each node
- Primary node automatically detected by parsing `clusterNodes` array from Technitium DNS API
- Cluster state includes: `type` (Primary/Secondary/Standalone), `domain`, `health`

**Frontend**:

- Helper hooks available: `usePrimaryNode()`, `useIsClusterEnabled()`, `useClusterNodes()`
- Write pages (DNS Filtering, DHCP, Zones Management) auto-select Primary node
- `<ClusterInfoBanner>` component displays restriction notice on write pages
- Read-only views (Dashboard, Logs, Sync) can still view all nodes

**Key Files**:

- `apps/backend/src/technitium/technitium.service.ts` - Primary detection logic
- `apps/frontend/src/hooks/usePrimaryNode.ts` - Helper hooks
- `apps/frontend/src/components/common/ClusterInfoBanner.tsx` - Info banner component
- `docs/features/clustering/CLUSTER_PRIMARY_WRITE_RESTRICTION.md` - Complete implementation guide

## Tools and Technologies

- Frontend: React or Vue.js, but I'm open to other suggestions
- Backend: Node.js with Express, but I'm open to other suggestions
- Technitium DNS API for interacting with Technitium DNS servers

## Production Deployment

**Deployment Options**:

- **Docker Compose** (Recommended): Use `docker-compose.yml` for production deployment
- **Frontend**: React + Vite SPA served by backend or nginx
- **Backend**: NestJS REST API with HTTPS support
- **Configuration**: Environment variables for node credentials (see `.env.example`)
  - Interactive UI access: Technitium-backed session auth (v1.4+: always enabled)
  - Background jobs: `TECHNITIUM_BACKGROUND_TOKEN` (least-privilege)
  - Removed in v1.4: `TECHNITIUM_CLUSTER_TOKEN`
  - Legacy only (Technitium DNS < v14): per-node `TECHNITIUM_<NODE>_TOKEN`
  - Frontend uses `VITE_API_URL` to connect to backend API

## Project Structure

This is a **monorepo** with the following structure:

```
technitium-dns-companion/
├── apps/
│   ├── backend/          # NestJS REST API server
│   │   ├── src/
│   │   │   ├── technitium/   # Technitium DNS service & controllers
│   │   │   └── main.ts
│   │   └── package.json
│   └── frontend/         # React + Vite SPA
│       ├── src/
│       │   ├── components/
│       │   ├── pages/
│       │   ├── context/
│       │   └── types/
│       └── package.json
├── docs/                 # All project documentation
└── configs/              # Sample configuration files
```

**Tech Stack**:

- **Backend**: NestJS (TypeScript), Axios for HTTP requests
- **Frontend**: React 18, TypeScript, Vite, TailwindCSS
- **State Management**: React Context API
- **Communication**: REST API (backend serves frontend in production)

## Project Documentation

**IMPORTANT**: Comprehensive documentation is organized in the `docs/` folder.

- **Start here**: Read `docs/README.md` for complete documentation structure and navigation
- **Architecture**: See `docs/architecture.md` for system design overview
- **Features**: Feature documentation in `docs/features/`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [Fail-Safe/Technitium-DNS-Companion](https://github.com/Fail-Safe/Technitium-DNS-Companion) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-22 -->
