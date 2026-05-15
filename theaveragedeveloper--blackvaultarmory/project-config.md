---
trigger: always_on
description: This is a self-hosted firearms inventory and range tracking app built with Next.js 15,
---

# BlackVault V1 Release — Claude Code Task Brief

This is a self-hosted firearms inventory and range tracking app built with Next.js 15,
Prisma, SQLite, and Tailwind. It runs via Docker Compose. The dev server is at
http://localhost:3000. All source is in `src/`, schema is in `prisma/schema.prisma`.

## How to run

```bash
npm run dev          # start dev server
npx prisma studio    # inspect the database
```

## Your job

Complete all tasks below in order. After each task, verify the change works before
moving to the next. Where the task says "test on mobile width", resize the browser
to 390px wide to check.

---

## TASK 1 — Remove encryption (plain text serial numbers)

**Goal:** Serial numbers are currently stored as `enc:...` blobs and never decrypted
before display. V1 ships with no encryption. Make serial numbers plain text everywhere.

**Files to change:**

### 1a. `src/lib/crypto.ts`
Replace the entire file with a clean passthrough implementation that:
- Makes `encryptField(value)` return `value` unchanged
- Makes `decryptField(value)` return the value unchanged if it doesn't start with `enc:`
- If the value DOES start with `enc:`, attempt to decrypt it using `VAULT_ENCRYPTION_KEY`
  from the environment (for backward compat with any existing data) — the existing
  decryption logic is correct, keep it in `decryptField` only
- Remove all imports of `createCipheriv` and `randomBytes` since encryption is gone

### 1b. `src/app/api/firearms/route.ts`
- The `decryptField` import and call on `serialNumber` in GET is already in place from
  a previous partial fix — verify it's correct
- In the POST handler, the `serialNumber` is written as plain text already (no encrypt
  call) — verify this is the case and no `encryptField` call exists

### 1c. `src/app/vault/[id]/page.tsx`
- Find the line rendering `firearm.serialNumber` directly (around line 192)
- Wrap it: `{decryptField(firearm.serialNumber) ?? "—"}`
- Add `import { decryptField } from "@/lib/crypto"` at the top with the other imports

### 1d. `src/app/api/exports/full-armory/route.ts`
- Find `resolvedSerial` (around line 381) where `firearm.serialNumber` is used directly
- Wrap with `decryptField()`: `decryptField(firearm.serialNumber) ?? firearm.serialNumber ?? ""`
- Add `import { decryptField } from "@/lib/crypto"` at the top

### 1e. Write a one-time migration script: `scripts/decrypt-serials.ts`
This script should:
- Connect to the Prisma DB
- Find all firearms where `serialNumber` starts with `enc:`
- Decrypt each one using `decryptField` from `@/lib/crypto`
- Update the record with the plain text value
- Log each update: `Decrypted serial for firearm: [name]`
- Be safe to run multiple times (idempotent)

Add this to `package.json` scripts:
```json
"decrypt-serials": "npx ts-node --project tsconfig.json scripts/decrypt-serials.ts"
```

**Test:** Start dev server, go to /vault — serial numbers should show plain text, not `enc:...`

---

## TASK 2 — Fix 503 errors on mobile/LAN (SQLite concurrency)

**Goal:** When navigating quickly on mobile/LAN, Next.js RSC prefetching fires multiple
concurrent Prisma queries against SQLite, causing 503s on Accessories, Builds, and
Vault detail pages.

### 2a. `docker-compose.yml` and `docker-compose.dev.yml`
- Both already have `connection_limit=1` appended to `DATABASE_URL` from a previous fix
- Verify both files have: `DATABASE_URL=file:/app/data/vault.db?connection_limit=1`

### 2b. `src/app/accessories/page.tsx`
The `getAccessories()` function is called at the top level with no error boundary.
Wrap the entire `export default async function AccessoriesPage()` body in a try/catch:
- On error, return a simple error UI:
```tsx
<div className="flex flex-col items-center justify-center min-h-[60vh] gap-4">
  <p className="text-vault-text-muted text-sm">Failed to load accessories.</p>
  <a href="/accessories" className="text-[#00C2FF] text-sm hover:underline">Tap to retry</a>
</div>
```

### 2c. `src/app/builds/page.tsx`
Same pattern — wrap the page body in try/catch with the same retry UI.

### 2d. `src/app/vault/[id]/page.tsx`
The `getFirearm()` call already has a `notFound()` guard but no error boundary.
Add a try/catch around the Prisma call in `getFirearm()` — on error, throw so
Next.js shows the error boundary, or return null and show the retry UI.

**Test:** Navigate rapidly between pages in a 390px window — no blank 503 pages.

---

## TASK 3 — LAN access: QR code + prominent URL display

**Goal:** Non-technical users on the same WiFi can't easily find the URL to open
BlackVault on their phone. Fix this with a QR code and a more prominent LAN URL.

### 3a. `src/app/settings/page.tsx`
In the "Mobile Access (Local Network)" section, after the Mobile URL box:
- Add a QR code rendered in the browser using `qrcode` npm package
- Install it: `npm install qrcode` and `npm install --save-dev @types/qrcode`
- When `finalLanUrl` is available, render a QR code as a `<canvas>` or `<img>` element
  below the URL text using `QRCode.toDataURL(finalLanUrl)`
- Size: 160×160px, centered, with a small caption: "Scan to open on your phone"
- Only show the QR code when `finalLanUrl` is non-empty

### 3b. `src/app/page.tsx` (Command Center / dashboard)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [theaveragedeveloper/BlackVaultArmory](https://github.com/theaveragedeveloper/BlackVaultArmory) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-09 -->
