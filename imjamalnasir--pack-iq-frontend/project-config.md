---
trigger: always_on
description: PackIQ backend API for frontend integration (auth, files, extractions, RAG)
---


# PackIQ Backend API – Frontend Integration

Use this rule when integrating the PackIQ frontend with the backend. The backend has two entry points: **Java API** (auth, files, extractions, documents) and **OCR/Model API** (RAG: ask, vector search, embed). Prefer the Java API for all calls that it proxies; use the OCR API base only for RAG endpoints if your deployment does not proxy them through Java.

## Base URLs (Next.js)

- **Java API** (primary): Use `NEXT_PUBLIC_API_URL` in `.env.local`. Example: `http://localhost:8080` or `https://your-app.example.com`.
- **OCR/Model API**: For RAG (ask, search, embed) if not proxied, use `NEXT_PUBLIC_OCR_API_URL` (e.g. `http://localhost:8080` when OCR app runs there).

## Authentication

All auth endpoints: `POST /auth/*`. No Bearer token for auth calls; use the returned tokens for subsequent requests.

### 1. Password check (get temp token)

```http
POST /auth/password-check
Content-Type: application/json

{ "email": "user@example.com", "password": "..." }
```

- **200**: `{ "valid": true, "token": "<tempToken>" }` — use `token` for send-otp / verify-otp.
- **200**: `{ "valid": false }` — wrong password.
- Temp token is short-lived (~15 min). Use it only for the OTP flow.

### 2. Send OTP

```http
POST /auth/send-otp
Content-Type: application/json

{ "tempToken": "<from password-check>", "channel": "EMAIL" }
```

- **channel**: `"EMAIL"` or `"SMS"` (SMS uses user's stored phone).
- **200**: `{ "valid": true }`.
- **401**: `{ "valid": false, "message": "Token expired" }` or `"Invalid token"`.
- **400**: missing/invalid body.

### 3. Resend OTP

```http
POST /auth/resend-otp
Content-Type: application/json

{ "tempToken": "<same>", "channel": "EMAIL" }
```

- **429**: rate limit (e.g. "Wait before resending").
- **401**: expired/invalid temp token.

### 4. Verify OTP (get access + refresh tokens)

```http
POST /auth/verify-otp
Content-Type: application/json

{ "tempToken": "<same>", "channel": "EMAIL", "otp": "123456" }
```

- **200**: `{ "valid": true, "token": "<accessToken>", "refreshToken": "<refreshToken>" }`.
- **401**: `{ "valid": false, "message": "OTP expired" }` or `"OTP invalid"` or token message.

### 5. Refresh access token

```http
POST /auth/refresh
Content-Type: application/json

{ "refreshToken": "<refreshToken>" }
```

- **200**: `{ "valid": true, "token": "<newAccessToken>", "refreshToken": "<newRefreshToken>" }`.
- **401**: expired/invalid refresh token.

### Using the access token

For endpoints that require auth (if your Java app enforces it), send:

```http
Authorization: Bearer <accessToken>
```

Store `token` and `refreshToken` (e.g. in memory or secure storage). On 401, retry once with refresh then redirect to login.

---

## Clients (Java API, session-based selection)

Base path: `/clients`. **Session:** Select and get-selected use the server session (cookie `JSESSIONID`). Use `credentials: 'include'` (or equivalent) so the browser sends cookies.

### List clients

```http
GET /clients
```

- **200**: `[ { "id": "<uuid>", "name": "...", "industry": "...", "location": "...", "logo": "..." | null } ]`.

### Get client by ID

```http
GET /clients/{id}
```

- **200**: `{ "id", "name", "industry", "location", "logo" }`.
- **404**: client not found.

### Select client for session

```http
POST /clients/select
Content-Type: application/json

{ "clientId": "<uuid>" }
```

- **200**: Returns the selected client object. Backend stores `clientId` in the session for subsequent requests from the same browser.
- **400**: missing or invalid `clientId`.
- **404**: client not found.

### Get selected client

```http
GET /clients/selected
```

- **200**: `{ "id", "name", "industry", "location" }` — client currently selected for this session.
- **204**: No client selected for this session.

---

## File and extraction endpoints (Java API)

Base path: `/files`. Use `multipart/form-data` for uploads. Responses are JSON. Include `Authorization: Bearer <accessToken>` if your backend requires it.

### Start OCR + extraction (upload file, get job_id)

```http
POST /files/ocr
Content-Type: multipart/form-data

file: <File>
doc_type: "spec" | "bom" | "sales"   (optional, default "spec")
user_id: <UUID>                      (optional)
```

- **202**: `{ "job_id": "<uuid>", "status": "pending", "doc_type": "spec", "document_id": "<uuid>" }`.
- **400**: missing file or invalid doc_type.

Poll for result:

```http
GET /files/extract/{jobId}
```

- **200**: `{ "job_id": "...", "status": "pending" }` or `"running"` — keep polling.
- **200**: `{ "job_id": "...", "status": "completed", "text": "...", "packaging": { ... }, "doc_type": "spec" }`.
- **200**: `{ "status": "failed", "error": "..." }`.
- **404**: job not found.

### Start extract

```http
POST /files/extract
Content-Type: multipart/form-data

file: <File>
user_id: <UUID>   (optional)
```

- **202**: same shape (job_id, document_id). Poll `GET /files/extract/{jobId}` as above.

### Ingest guidance PDF

```http
POST /files/ingest/guidance
Content-Type: multipart/form-data

file: <PDF>
jurisdiction: "California"     (required)
pro: "CalRecycle"             (required)
effective_date: "2025-09-02"   (required)
topics: "topic1,topic2"       (optional)
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [imjamalnasir/Pack-IQ-Frontend](https://github.com/imjamalnasir/Pack-IQ-Frontend) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-29 -->
