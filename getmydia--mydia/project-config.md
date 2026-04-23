---
trigger: always_on
description: This is a web application written using the Phoenix web framework.
---

This is a web application written using the Phoenix web framework.

## About Mydia

Mydia is a self-hosted media management application for organizing and tracking TV shows and movies. It helps users:

- Manage their personal media libraries
- Track TV show episodes and seasons
- Organize movies and their metadata
- Monitor media files and their locations
- Automatically fetch metadata from external sources

The application is designed to be deployed on personal servers or home lab environments, giving users complete control over their media collection data.

## Remote Access Architecture (P2P)

Mydia uses a decentralized **p2p** architecture for remote access.

### Key Components

- **Core (Rust)**: The shared networking logic (Host, Swarm, DHT, mDNS) is implemented in a pure Rust crate (`native/mydia_p2p_core`). This ensures protocol parity between client and server.
- **Backend (Elixir)**: The Phoenix app wraps the core crate using a Rustler NIF (`Mydia.P2p`). It acts as a permanent node in the mesh.
- **Frontend (Flutter)**: The player app wraps the same core crate using `flutter_rust_bridge`. It connects to the backend via the p2p mesh for discovery and control.

### Connectivity

- **Discovery**: Nodes find each other via mDNS (local network) and Kademlia DHT (internet).
- **Transport**: Connections are established over TCP or QUIC, secured with Noise encryption.
- **Media**: Media streams (HLS) are served over the p2p connection (via a local proxy in the client).

## Metadata Relay Service

Mydia uses a companion service called **metadata-relay**, which is a developer-owned service that:

- **Proxies metadata requests** to external services like TVDB and TMDB
- **Protects API keys** by avoiding direct client-to-service connections
- **Reduces rate limiting issues** by centralizing API requests
- **Prevents API key leakage** in client applications

### Key Points

- The metadata-relay service is **developed together with mydia** but **deployed completely separately**
- Mydia instances connect to metadata-relay instead of hitting TVDB/TMDB directly
- This architecture allows multiple mydia instances to share a single metadata-relay deployment
- The service handles authentication and rate limiting centrally

### When Working on Metadata Integration

- **Always** use the metadata-relay endpoints instead of direct TVDB/TMDB API calls
- **Never** embed TVDB or TMDB API keys directly in the mydia application
- Configure the metadata-relay base URL via application configuration
- Handle metadata-relay service failures gracefully with appropriate error messages

## Player (Flutter)

The Flutter player has additional workflow guidance in `player/CLAUDE.md`.

### Player Android Builds

Android builds use the player's Nix flake (`player/flake.nix`) which provides Flutter, Android SDK, NDK, and Rust cross-compilation toolchains.

**Commands:**

- `./dev player android build` - Build release APK
- `./dev player android run` - Build and run on connected Android device
- `./dev player android shell` - Open nix develop shell for manual commands

**Output:** `player/build/app/outputs/flutter-apk/app-release.apk`

**Requirements:** Nix must be installed on the host system (not Docker).

**What the flake provides:**
- Flutter SDK
- Android SDK with NDK 27.0.12077973
- Rust toolchain with Android targets (aarch64, armv7, x86_64, i686)
- All necessary environment variables for Rust cross-compilation

## Project guidelines

- Use `mix precommit` alias when you are done with all changes and fix any pending issues
- Use the already included and available `:req` (`Req`) library for HTTP requests, **avoid** `:httpoison`, `:tesla`, and `:httpc`. Req is included by default and is the preferred HTTP client for Phoenix apps

### DRY Patterns (Don't Repeat Yourself)

**Extract repeated logic into private functions:**

    # BAD: duplicated validation logic
    def create_show(attrs) do
      if String.length(attrs["title"] || "") > 0 do
        # ...
      end
    end

    def update_show(show, attrs) do
      if String.length(attrs["title"] || "") > 0 do
        # ...
      end
    end

    # GOOD: extract to private function
    defp valid_title?(attrs), do: String.length(attrs["title"] || "") > 0

**Use `with` for sequential operations instead of nested case:**

    # BAD: deeply nested case statements
    case fetch_show(id) do
      {:ok, show} ->
        case fetch_seasons(show) do
          {:ok, seasons} ->
            case update_metadata(show, seasons) do
              {:ok, updated} -> {:ok, updated}
              {:error, reason} -> {:error, reason}
            end
          {:error, reason} -> {:error, reason}
        end
      {:error, reason} -> {:error, reason}
    end

    # GOOD: flat with statement
    with {:ok, show} <- fetch_show(id),
         {:ok, seasons} <- fetch_seasons(show),
         {:ok, updated} <- update_metadata(show, seasons) do
      {:ok, updated}
    end

**Centralize business logic in context modules:**

- **Never** duplicate query logic across LiveViews or controllers
- **Always** put queries and business logic in context modules (e.g., `Mydia.Media`, `Mydia.Libraries`)
- Create reusable query functions: `list_shows/1`, `get_show!/1`, `list_shows_with_seasons/1`


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [getmydia/mydia](https://github.com/getmydia/mydia) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-20 -->
