---
trigger: always_on
description: RustDeskPro-WSS is a production-ready Docker Compose setup for running RustDesk Server Pro (hbbr/hbbs) behind Caddy reverse proxy with automatic HTTPS, secure file permissions, and hardened firewall configuration.
---

# RustDeskPro-WSS

RustDeskPro-WSS is a production-ready Docker Compose setup for running RustDesk Server Pro (hbbr/hbbs) behind Caddy reverse proxy with automatic HTTPS, secure file permissions, and hardened firewall configuration.

Always reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.

## Working Effectively

### Prerequisites and Environment Setup
- Ensure Docker Engine and Docker Compose Plugin are installed:
  ```bash
  docker --version
  docker compose version
  ```
- The setup requires Linux with host networking support (network_mode: host)
- Root privileges are required for installation (creates system users and directories)

### Essential Commands and Timing
- **NEVER CANCEL**: All Docker operations complete quickly (under 10 seconds typically)
- Set timeout to 60+ seconds for Docker operations as a safety buffer

#### Bootstrap the Repository
```bash
# Clone and enter directory
git clone https://github.com/tommyvange/RustDeskPro-WSS.git
cd RustDeskPro-WSS

# Create environment configuration
cp .env.example .env
${EDITOR:-nano} .env
```

#### Validate Configuration
```bash
# Test Docker Compose syntax (instant)
docker compose config

# Test Caddyfile syntax (requires caddy image, ~1-2 seconds)
docker run --rm -v "$(pwd)/Caddyfile:/etc/caddy/Caddyfile" caddy:latest caddy validate --config /etc/caddy/Caddyfile
```

#### Install and Deploy
```bash
# Make installer executable
chmod +x install.sh

# Run automated installation (requires root, ~10-15 seconds total)
# NEVER CANCEL: Process completes quickly but requires root for user/directory creation
sudo ./install.sh
```

#### Pull Images Manually (if needed)
```bash
# Pull all required Docker images (~7-10 seconds)
# NEVER CANCEL: Set timeout to 60+ seconds
docker compose pull
```

#### Container Management
```bash
# Start services (~1 second)
docker compose up -d --force-recreate

# Check container status (instant)
docker compose ps

# View logs (instant)
docker compose logs
docker compose logs -f  # follow logs

# Stop services (~2 seconds)
docker compose down
```

## Validation

### Essential Validation Steps
Always perform these validation steps after making changes:

1. **Configuration Validation**:
   ```bash
   # Validate Docker Compose configuration
   docker compose config
   
   # Validate environment variables are set
   grep -v '^#' .env | grep -v '^$'
   ```

2. **Container Health Check**:
   ```bash
   # Start containers and verify they're running
   docker compose up -d
   docker compose ps
   
   # Check for any container errors
   docker compose logs --no-color | grep -i error || echo "No errors found"
   ```

3. **Port and Network Validation**:
   ```bash
   # Verify required ports are available (21114, 21115, 21116, 21117, 21118, 21119)
   ss -tuln | grep -E ':(21114|21115|21116|21117|21118|21119|80|443)'
   
   # Test Caddy configuration syntax
   docker compose exec caddy caddy validate --config /etc/caddy/Caddyfile
   ```

4. **File Permissions Validation**:
   ```bash
   # Verify directory permissions are correct
   ls -la /srv/caddy/ /srv/rustdesk/ 2>/dev/null || echo "Directories not created yet"
   ```

### Manual Validation Scenarios
After deployment, test these scenarios:

1. **HTTP/HTTPS Connectivity**: Visit `https://yourdomain.com` (should show RustDesk console interface)
2. **WebSocket Endpoints**: Test `/ws/id` and `/ws/relay` endpoints are accessible
3. **Container Logs**: Verify no critical errors in `docker compose logs`

### You Cannot Test Interactively
- The RustDesk Pro server requires a valid license key for full functionality
- WebSocket connections require RustDesk client applications
- Some features only work with actual RustDesk clients connecting

## Repository Structure and Key Files

### Essential Files
```
.
├── README.md              # Complete setup documentation
├── install.sh             # Automated installation script (main entry point)
├── compose.yml            # Docker services definition
├── Caddyfile              # Reverse proxy configuration template
├── .env.example           # Environment configuration template
└── LICENSE                # GPLv3 license
```

### Key Configuration Points
- **compose.yml**: Defines 3 services (hbbr, hbbs, caddy) with host networking
- **Caddyfile**: Template with EXAMPLE.COM placeholder and optional CORS block
- **.env**: Contains domains, file paths, CORS settings, and user IDs

## Common Workflows

### Environment Configuration (.env file)
Required variables that must be set:
```bash
DOMAINS=your.domain.com              # Comma-separated domain list
FILE_LOCATION_CADDY=/srv/caddy       # Caddy data/config path
FILE_LOCATION_RUSTDESK=/srv/rustdesk # RustDesk data path
RUSTDESK_CORS=true                   # Enable/disable CORS restrictions
# UIDs/GIDs automatically populated by install.sh
```

### Caddyfile Processing
The install script automatically:
1. Replaces `EXAMPLE.COM` with your domains (comma → space conversion)
2. Removes CORS block if `RUSTDESK_CORS=false`

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [tommyvange/RustDeskPro-WSS](https://github.com/tommyvange/RustDeskPro-WSS) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-06 -->
