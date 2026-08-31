---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is an Express.js server built with TypeScript for ffmpeg processing. It uses modern tooling and follows strict TypeScript best practices from Total TypeScript guidelines.

## Commands

### Development

- `pnpm dev` - Start development server with hot reload (nodemon + tsx)
- `pnpm build` - Lint and compile TypeScript to JavaScript (output: `dist/`)
- `pnpm start` - Run production build from `dist/index.js`
- `pnpm lint` - Run ESLint

### Setup

```bash
nvm use                              # Switch to Node v24.11.1
corepack enable
corepack prepare pnpm@10.1.0 --activate
pnpm install
cp .env.example .env                 # Create environment file
```

### Docker

```bash
docker build -t ffmpeg-server:latest .
docker run -p 5675:5675 -e PORT=5675 ffmpeg-server:latest
```

### CI/CD

```bash
# Create and push a git tag to trigger Docker image build
git tag v1.0.0
git push origin v1.0.0

# Then create a GitHub Release from the tag (required for deployment)
# Go to GitHub → Releases → Draft new release → Select tag → Publish
```

## Architecture

### Module System

- **ES Modules only** (`"type": "module"` in package.json)
- Use `import/export` syntax, not `require()`
- TypeScript config uses `NodeNext` module resolution

### TypeScript Configuration

Following Total TypeScript best practices:

- **Strict mode**: `strict`, `noUncheckedIndexedAccess`, `noImplicitOverride`
- **Explicit imports**: `verbatimModuleSyntax` requires `import type` for type-only imports
- **No implicit any**: All types must be explicit or inferred
- **Source**: `src/` → **Output**: `dist/`

### Code Quality Rules

ESLint enforces:

- `no-console: error` - Use `// eslint-disable-next-line no-console` only for intentional logging
- Unused variables must be prefixed with `_` (e.g., `_req`, `_unused`)
- `prefer-const` for immutable bindings
- No debugger statements

### Git Hooks (Husky)

**Pre-commit** (`.husky/pre-commit`):

- Runs `pnpm lint` on all files
- Runs Gitleaks on staged files (install: `brew install gitleaks`)
- Will warn if Gitleaks not installed but won't fail

**Pre-push** (`.husky/pre-push`):

- Runs `pnpm build` to ensure TypeScript compiles
- Fails if linting or compilation fails

### Environment Configuration

- Default port: `5675` (configurable via `PORT` env var)
- Environment variables loaded via `dotenv` from `.env` file
- `.env` files are gitignored
- **Supabase Storage**: Required env vars:
  - `SUPABASE_URL` - Supabase project URL
  - `SUPABASE_SERVICE_ROLE_KEY` - Service role key for server-side operations
  - `SUPABASE_BUCKET` - Storage bucket name for FFmpeg outputs (default: `ffmpeg-outputs`)
  - `MAX_OUTPUT_FILE_SIZE_BYTES` - Max output file size before upload (default: `104857600`)
- **Anthropic API**: Required for `/execute-llmpeg` endpoint:
  - `ANTHROPIC_API_KEY` - Anthropic API key for Claude Sonnet 4

### Express Server Structure

- Server entry point: `src/index.ts`
- Middleware: CORS enabled, JSON body parsing, request ID generation
- Current endpoints:
  - `GET /health` - Health check with timestamp and FFmpeg version verification
  - `POST /execute-ffmpeg` - Execute FFmpeg commands with queue management
    - Request body: `{ "command": "ffmpeg -i input.mp4 output.mp4" }`
    - Command MUST start with `ffmpeg ` (validation enforced by Zod)
    - Input files can be HTTP/HTTPS URLs (automatically downloaded)
    - Returns: `{ success: true, stdout, stderr, exitCode, outputs: [{ filename, path, url, size, contentType }] }`
  - `POST /execute-ffprobe` - Inspect media files with FFprobe
    - Request body: `{ "command": "ffprobe -v quiet -print_format json -show_streams input.mp4" }`
    - Command MUST start with `ffprobe ` (validation enforced by Zod)
    - Input files can be HTTP/HTTPS URLs (automatically downloaded)
    - Returns: `{ success: true, stdout, stderr, exitCode }` (no outputs — read-only)
    - 1-minute timeout per command
  - `POST /execute-llmpeg` - Natural language FFmpeg command generation and execution
    - Request body: `{ "task": "concatenate videos", "inputs": [{ "name": "video1", "url": "https://..." }, ...] }`
    - Uses Claude Sonnet 4 to convert natural language to FFmpeg commands
    - Automatically downloads input files and executes generated command
    - Returns: Same format as `/execute-ffmpeg`

### FFmpeg Processing Architecture

- **Queue-based execution** using `p-queue` to limit concurrent FFmpeg processes
- **Concurrency limit**: Dynamically calculated as `Math.min(Math.max(Math.floor(cpuCount / 2), 2), 8)`
- **Timeout protection**: Default 5-minute timeout per FFmpeg process
- **Command validation**: Requires full FFmpeg command starting with `ffmpeg ` (e.g., `ffmpeg -i input.mp4 output.mp4`)
- **Argument parsing**: Uses `shell-quote` library to safely parse command arguments after stripping `ffmpeg ` prefix
- **Security**: Rejects shell operators (`>`, `|`, `&&`, etc.) in FFmpeg arguments
- **Validation**: Uses Zod for request body validation with `.refine()` to ensure command starts with `ffmpeg `

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BamBam-Vid/ffmpeg-server](https://github.com/BamBam-Vid/ffmpeg-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-31 -->
