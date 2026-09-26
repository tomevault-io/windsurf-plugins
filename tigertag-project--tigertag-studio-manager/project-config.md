---
trigger: always_on
description: Validates a Key6 API key and returns the associated user information.
---

# AGENT.md — TigerTag Inventory Web Page

## Goal

Create a clean, single-page HTML application to manage a user's TigerTag inventory.

The page must allow a logged-in user to:

1. Authenticate with Firebase.
2. Generate or delete their Key6 API key.
3. Verify a Key6 API key.
4. Export their inventory as JSON.
5. Display inventory items in a readable table.
6. Test or update a spool weight using the public API.
7. Optionally link an RFID UID to a spool through Firebase callable function `indexRfidForSpool`.

This page is dedicated to **inventory management only**.

Do not include TigerTag Manager media/image/file administration endpoints such as:
- `UploadProductImg`
- `DeleteProductImg`
- `UploadFiles`
- `uploadFilesLocal`
- `DeleteFiles`
- `UploadMedia`
- `DeleteMedia`

Those endpoints are internal/backoffice tools and must not be exposed in this public inventory page.

---

## Base URLs

Prefer the CDN domain for public usage:

```txt
https://cdn.tigertag.io
```

Cloud Functions direct URL may be used only as fallback:

```txt
https://us-central1-tigertag-connect.cloudfunctions.net
```

---

## Useful Inventory Functions

### 1. `createAccessKey6`

Creates, rotates, or deletes the user's 6-character API key.

This endpoint requires a Firebase ID Token.

#### Endpoint

```txt
POST https://cdn.tigertag.io/createAccessKey6
```

#### Headers

```http
Authorization: Bearer <FIREBASE_ID_TOKEN>
Content-Type: application/json
```

#### Create Key Request

```json
{
  "data": {
    "action": "create",
    "label": "inventory-web"
  }
}
```

#### Create Key Response

```json
{
  "result": {
    "success": true,
    "key": "Tk237U",
    "label": "inventory-web"
  }
}
```

#### Delete Key Request

```json
{
  "data": {
    "action": "delete"
  }
}
```

#### Delete Key Response

```json
{
  "result": {
    "success": true,
    "message": "All API keys deleted (access disabled)"
  }
}
```

#### UI Requirements

The page must provide:
- a "Generate API Key" button;
- a "Delete API Key" button;
- a visible field showing the current generated key after creation;
- a warning that deleting the key disables external access such as TigerScale.

---

### 2. `pingByApiKey`

Validates a Key6 API key and returns the associated user information.

#### Endpoint

```txt
GET https://cdn.tigertag.io/pingbyapikey?ApiKey=<KEY6>
```

Also support:

```txt
GET https://cdn.tigertag.io/pingByApiKey?ApiKey=<KEY6>
```

depending on Hosting rewrites.

#### Example Request

```bash
curl "https://cdn.tigertag.io/pingbyapikey?ApiKey=Tk237U"
```

#### Success Response

```json
{
  "success": true,
  "uid": "xe1zTc8Op3dmV5mC9SfUnziuSaF2",
  "displayName": "Benoît",
  "message": "TigerTag API key valid"
}
```

#### Error Response

```json
{
  "success": false,
  "reason": "invalid_api_key"
}
```

#### UI Requirements

The page must provide:
- an input field for Key6;
- a "Test API Key" button;
- a status badge:
  - green if valid;
  - red if invalid;
  - grey if untested.

---

### 3. `exportInventoryByApiKey`

Exports the user's inventory as JSON.

This endpoint requires:
- a valid Key6 API key;
- the Firebase Auth email of the user.

#### Endpoint

```txt
GET https://cdn.tigertag.io/exportInventory?ApiKey=<KEY6>&email=<USER_EMAIL>
```

#### Example Request

```bash
curl "https://cdn.tigertag.io/exportInventory?ApiKey=Tk237U&email=user%40example.com"
```

#### Success Response

The response is an object keyed by spool UID:

```json
{
  "8396248126918784": {
    "uid": 8396248126918784,
    "measure_gr": 1000,
    "container_weight": 232,
    "weight_available": 519,
    "material": "PLA",
    "brand": "TigerTag",
    "color_name": "Red"
  },
  "8396248126918785": {
    "uid": 8396248126918785,
    "measure_gr": 750,
    "container_weight": 120,
    "weight_available": 380
  }
}
```

#### Error Responses

```json
{
  "success": false,
  "reason": "missing_email"
}
```

```json
{
  "success": false,
  "reason": "email_mismatch"
}
```

```json
{
  "success": false,
  "reason": "invalid_api_key"
}
```

#### UI Requirements

The page must:
- automatically use the logged-in Firebase user's email;
- ask for the Key6 API key;
- fetch the inventory;
- render the inventory in a table;
- show raw JSON in a collapsible `<details>` block;
- allow refresh.

Recommended table columns:
- UID / spool ID
- Material
- Brand
- Color
- Weight available
- Container weight
- Measure / capacity
- Last update
- Actions

Do not assume every field exists. Use graceful fallbacks such as `"-"`.

---

### 4. `setSpoolWeightByRfid`

Updates the weight of a spool through the public API.

Despite its name, the current implementation accepts the spool UID directly. The endpoint does not require Firebase login. It uses Key6.

#### Endpoint

```txt
GET https://cdn.tigertag.io/setSpoolWeightByRfid?ApiKey=<KEY6>&uid=<SPOOL_ID>&weight=<RAW_WEIGHT>
```

#### Example Request

```bash
curl "https://cdn.tigertag.io/setSpoolWeightByRfid?ApiKey=Tk237U&uid=8396248126918784&weight=500"
```

#### POST Alternative

```txt
POST https://cdn.tigertag.io/setSpoolWeightByRfid
```

#### POST Headers

```http
Content-Type: application/json
```

#### POST Body

```json
{
  "ApiKey": "Tk237U",
  "uid": "8396248126918784",
  "weight": 500
}
```

#### Success Response

```json
{

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [TigerTag-Project/TigerTag-Studio-Manager](https://github.com/TigerTag-Project/TigerTag-Studio-Manager) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-26 -->
