---
trigger: always_on
description: Ollama Proxy is a Dockerized Nginx reverse proxy that provides authenticated access to a local Ollama AI service through Cloudflare Tunnel. The proxy includes CORS support, authentication via Bearer tokens, and secure internet exposure.
---

# Ollama Proxy - GitHub Copilot Instructions

Ollama Proxy is a Dockerized Nginx reverse proxy that provides authenticated access to a local Ollama AI service through Cloudflare Tunnel. The proxy includes CORS support, authentication via Bearer tokens, and secure internet exposure.

**ALWAYS reference these instructions first and fallback to search or bash commands only when you encounter unexpected information that does not match the info here.**

## Working Effectively

### Prerequisites and Dependencies
- **Docker and Docker Compose**: Required for building and running the proxy
- **Cloudflare Account**: Needed to obtain a Cloudflare Tunnel token (see `docs/cloudflare.md`)
- **Ollama Service**: Must be running on the host machine at port `11434`

### Quick Start Commands
1. **Clone and setup environment**:
   ```bash
   git clone https://github.com/kesor/ollama-proxy.git
   cd ollama-proxy
   cp .env.secret-example .env.secret
   # Edit .env.secret with your actual tokens
   ```

2. **Build the Docker image**:
   ```bash
   docker compose build
   ```
   **IMPORTANT**: The default Dockerfile has an SSL certificate issue when downloading cloudflared. **NEVER CANCEL** the build process - it will fail within 10-15 seconds due to SSL cert issues.

   **WORKAROUND**: If the build fails with SSL certificate errors, modify the curl command in the Dockerfile temporarily by adding the `-k` flag:
   ```bash
   # Change line ~19 in Dockerfile from:
   curl -# -L --output cloudflared.deb https://...
   # To:
   curl -k -# -L --output cloudflared.deb https://...
   ```

### Expected Build Behavior
**IMPORTANT**: The default `docker compose build` **WILL FAIL** due to SSL certificate issues. This is expected behavior.

**Working Build Process**:
1. First build attempt will fail with SSL errors (expected)
2. Modify Dockerfile to add `-k` flag to curl commands
3. Rebuild successfully

3. **Run the service**:
   ```bash
   docker compose up -d
   ```

4. **Check logs**:
   ```bash
   docker compose logs
   ```

### Build Process Details
- **Docker build time**: 2-3 seconds (with layer caching), 5-10 seconds (fresh build)
- **NEVER CANCEL**: While builds are generally fast, always set timeout to 120+ seconds for Docker commands
- **Common failure**: SSL certificate issues when downloading cloudflared from GitHub releases

### Known Build Issues and Solutions
1. **SSL Certificate Error**: The original Dockerfile fails with SSL certificate validation errors:
   ```
   curl: (60) SSL certificate problem: self-signed certificate in certificate chain
   ```
   **Solution**: Add `-k` flag to curl commands in Dockerfile lines 19 and 21:
   ```dockerfile
   curl -k -# -L --output cloudflared.deb https://github.com/...
   ```

2. **Environment File Missing**: Docker Compose fails if `.env.secret` doesn't exist:
   ```
   env file .env.secret not found
   ```
   **Solution**: Always create `.env.secret` from `.env.secret-example` before running

3. **Environment Variable Caching**: Docker Compose may cache environment values between runs:
   ```
   # After changing .env.secret, always restart completely
   docker compose down
   docker compose up -d
   ```

### Environment Configuration
- **Required file**: `.env.secret` (copy from `.env.secret-example`)
- **Required variables**:
  ```bash
  CLOUDFLARE_TUNNEL_TOKEN="your_actual_tunnel_token"
  OLLAMA_SECRET_API_KEY="sk-your_custom_secret_key"
  ```
- **Security**: The `.env.secret` file is in `.gitignore` and should never be committed

### Running and Testing
1. **Start the service**:
   ```bash
   docker compose up -d
   ```

2. **Test basic connectivity** (requires valid tokens):
   ```bash
   curl https://your-domain.com/api/version \
     -H "Authorization: Bearer your_secret_api_key"
   ```

3. **Test authentication** (should return 401):
   ```bash
   curl https://your-domain.com/api/version \
     -H "Authorization: Bearer wrong_key"
   ```

4. **Test full API functionality**:
   ```bash
   curl -i https://your-domain.com/v1/chat/completions \
     -H "Content-Type: application/json" \
     -H "Authorization: Bearer your_secret_api_key" \
     -d '{
       "model":"llama3.2",
       "messages":[
         {"role":"system","content":"You are a helpful assistant."},
         {"role":"user","content":"Test prompt."}
       ],
       "temperature":1,
       "max_tokens":10,
       "stream":false
     }'
   ```

## Validation Scenarios

**ALWAYS test these scenarios after making changes:**

1. **Authentication Test**:
   - Valid token should return 502 (if no Ollama backend) or 200 (if Ollama is running)
   - Invalid token should return 401 Unauthorized
   - Missing token should return 401 Unauthorized

2. **CORS Test**:
   - OPTIONS requests should work without authentication
   - Proper CORS headers should be present in responses

3. **Container Health**:
   - Check logs for nginx startup messages
   - Verify cloudflared connection (will show token errors with dummy tokens)
   - Ensure no nginx configuration errors

4. **Template Processing**:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [kesor/ollama-proxy](https://github.com/kesor/ollama-proxy) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
