---
trigger: always_on
description: This is a **Minecraft SMP Server Administration Panel** built with **Next.js 15**, **TypeScript**, **Prisma**, and **TailwindCSS**. It provides a web interface for managing Minecraft servers via tmux sessions, file management, user authentication with 2FA, and activity logging.
---

# GitHub Copilot Instructions for SMP Shard Admin Panel

## 🎯 Project Overview
This is a **Minecraft SMP Server Administration Panel** built with **Next.js 15**, **TypeScript**, **Prisma**, and **TailwindCSS**. It provides a web interface for managing Minecraft servers via tmux sessions, file management, user authentication with 2FA, and activity logging.

---

## 🚨 CRITICAL CONVENTIONS

### 1. API Path Convention
**ALWAYS use the `/apanel44` base path for ALL routes and API calls:**

```typescript
// ✅ CORRECT
fetch('/apanel44/api/files')
f.../apanel44/dashboard"  // Next.js will add it, causing double prefix
```

**Key Rules:**
- API paths: `/apanel44/api/*`
- Page routes: Next.js handles the prefix automatically from `basePath` in `next.config.ts`
- Never hardcode `/apanel44` in `href` attributes in Link components
- Never put pages in `/pages/apanel44/` directory

### 2. Trailing Slash Requirement
All URLs MUST end with a trailing slash (configured in `next.config.ts`):
- `/apanel44/dashboard/` ✅
- `/apanel44/dashboard` ❌ (will redirect)

---

## 📁 Project Structure

```
smp-shard-admin-panel/
├── .github/
│   └── copilot-instructions.md    # This file
├── src/
│   ├── pages/              # Next.js pages (NOT in /pages/apanel44/)
│   │   ├── api/           # API routes (automatically prefixed with /apanel44/api)
│   │   │   ├── auth/      # NextAuth.js routes
│   │   │   ├── files/     # File management APIs
│   │   │   ├── server/    # Server control APIs (console, status)
│   │   │   ├── monitoring/ # Metrics and monitoring
│   │   │   └── permissions/ # User permission management
│   │   ├── index.tsx      # Landing page → /apanel44/
│   │   ├── login.tsx      # Login page → /apanel44/login/
│   │   └── dashboard.tsx  # Main dashboard → /apanel44/dashboard/
│   ├── components/        # Reusable React components
│   ├── lib/              # Utility functions and helpers
│   │   ├── prisma.ts     # Prisma client singleton
│   │   ├── middleware.ts  # Auth middleware (withAdmin, withSuperAdmin)
│   │   ├── console.ts    # Tmux/server console utilities
│   │   ├── fileUtils.ts  # File management utilities
│   │   └── permissions.ts # Permission definitions
│   ├── styles/           # Global styles and Tailwind config
│   └── types/            # TypeScript type definitions
├── prisma/
│   └── schema.prisma     # Database schema
└── scripts/              # Shell scripts for server management
```

---

## 🔐 Security Patterns

### Authentication & Authorization
1. **Middleware Protection:**
   ```typescript
   import { withAdmin, withSuperAdmin } from '@/lib/middleware';
   
   // Admin or Super Admin required
   export default withAdmin(handler);
   
   // Super Admin only
   export default withSuperAdmin(handler);
   ```

2. **Role Hierarchy:**
   - `SUPER_ADMIN`: Full access to everything
   - `ADMIN`: Access to most features except user management
   - `USER`: Read-only access (if implemented)

3. **2FA Support:**
   - Email OTP via SMTP
   - TOTP (Google Authenticator) support
   - Configured per user in database

### Input Sanitization
**ALWAYS sanitize user inputs** for file operations and command execution:

```typescript
import { sanitizeFilename, sanitizePath, sanitizeCommand } from '@/lib/fileUtils';
import { sanitizeSessionName } from '@/lib/console';

// File operations
const safeName = sanitizeFilename(userInput);
const safePath = sanitizePath(userInput);

// Command execution (tmux)
const safeCommand = sanitizeCommand(userInput);
const safeSession = sanitizeSessionName(sessionName);
```

### Path Traversal Prevention
```typescript
import { validateAndResolvePath, PLUGINS_DIR } from '@/lib/fileUtils';

// Validate paths ALWAYS stay within allowed directory
const fullPath = await validateAndResolvePath(userPath, PLUGINS_DIR);
```

### Command Whitelisting
```typescript
import { ADMIN_ALLOWED_COMMANDS } from '@/lib/console-constants';

// Only allow pre-approved Minecraft commands
if (!ADMIN_ALLOWED_COMMANDS.includes(baseCommand)) {
  throw new Error('Command not allowed');
}
```

---

## 🏗️ Architecture Patterns

### API Route Structure
```typescript
import { NextApiResponse } from 'next';
import { withAdmin, AuthenticatedRequest } from '@/lib/middleware';
import { logActivity } from '@/lib/activity';

async function handler(req: AuthenticatedRequest, res: NextApiResponse) {
  if (req.method === 'GET') {
    try {
      // Business logic here
      
      // Log activity
      await logActivity({
        userId: req.user.id,
        actionType: 'action_name',
        resource: 'resource_name',
        details: { /* JSON details */ },
        req,
      });
      
      return res.status(200).json({ data: result });
    } catch (error) {
      console.error('Error:', error);
      return res.status(500).json({ error: 'Internal server error' });
    }
  }
  
  return res.status(405).json({ error: 'Method not allowed' });
}

export default withAdmin(handler);
```

### File Upload Handling
```typescript
import { IncomingForm } from 'formidable';

export const config = {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/Salty-Dragon) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
