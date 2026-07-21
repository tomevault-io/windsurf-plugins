---
trigger: always_on
description: Agent workflow endpoints return Server-Sent Events (SSE) streams using the AI SDK v7 UI message format. `/api/agents/router` is the exception: it returns a JSON intent-classification result.
---

# AI Agents

Agent workflow endpoints return Server-Sent Events (SSE) streams using the AI SDK v7 UI message format. `/api/agents/router` is the exception: it returns a JSON intent-classification result.

## Streaming Overview

Agent workflow endpoints use Server-Sent Events (SSE) for streaming responses. JavaScript clients consume the AI SDK v7 UI message stream from a `fetch()` response with `ReadableStream`.

**Authentication Note**: All agent endpoints require authentication. Use the `sb-access-token` cookie (Supabase default cookie name) or pass the JWT token via `Authorization: Bearer <token>` header.

### TypeScript Example

```typescript
const response = await fetch("http://localhost:3000/api/agents/flights", {
  method: "POST",
  headers: {
    "Content-Type": "application/json",
    Cookie: `sb-access-token=${jwtToken}`,
  },
  body: JSON.stringify({
    origin: "JFK",
    destination: "CDG",
    departureDate: "2025-07-01",
  }),
});

const reader = response.body?.getReader();
const decoder = new TextDecoder();

while (true) {
  const { done, value } = await reader.read();
  if (done) break;
  const chunk = decoder.decode(value);
  // Process SSE messages
}
```

---

## `POST /api/agents/flights`

Streaming flight search agent.

**Authentication**: Required  
**Rate Limit Key**: `agents:flight`  
**Content-Type**: `application/json`  
**Response**: `text/event-stream` (SSE)

### Request Body

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `origin` | string | Yes | Origin airport IATA code (min 3 chars) |
| `destination` | string | Yes | Destination airport IATA code (min 3 chars) |
| `departureDate` | string | Yes | Departure date (YYYY-MM-DD) |
| `returnDate` | string | No | Return date (YYYY-MM-DD) |
| `passengers` | number | No | Passenger count (default: 1) |
| `cabinClass` | string | No | Cabin class: `economy`, `premium_economy`, `business`, `first` (default: `economy`) |
| `currency` | string | No | ISO currency code (default: "USD") |
| `nonstop` | boolean | No | Require nonstop flights only (default: false) |

### Response

`200 OK` - SSE stream with flight search results

### Errors

- `400` - Invalid request parameters
- `401` - Not authenticated
- `429` - Rate limit exceeded

### Example

```bash
curl -N -X POST "http://localhost:3000/api/agents/flights" \
  --cookie "sb-access-token=$JWT" \
  -H "Content-Type: application/json" \
  -d '{
    "origin": "JFK",
    "destination": "CDG",
    "departureDate": "2025-07-01",
    "returnDate": "2025-07-15",
    "passengers": 2,
    "cabinClass": "economy"
  }'
```

---

## `POST /api/agents/accommodations`

Streaming accommodation search agent.

**Authentication**: Required  
**Rate Limit Key**: `agents:accommodations`  
**Content-Type**: `application/json`  
**Response**: `text/event-stream` (SSE)

### Request Body

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `location` | string/object | Yes | Location string or geocoordinates {latitude, longitude} |
| `checkIn` | string | Yes | Check-in date (YYYY-MM-DD) |
| `checkOut` | string | Yes | Check-out date (YYYY-MM-DD) |
| `guests` | object | Yes | Guest composition {adults: number, children: number (optional)} |
| `rooms` | number | No | Number of rooms (default: 1) |
| `roomPreferences` | array | No | Preferences: bed_type, smoking_allowed, non_smoking |
| `amenities` | array | No | Required amenities (e.g., "wifi", "parking", "gym") |
| `priceRange` | object | No | Budget constraints {min: number, max: number} |
| `currency` | string | No | ISO currency code (default: "USD") |
| `propertyTypes` | array | No | Filter by type: hotel, apartment, villa, hostel, etc. |
| `starRating` | object | No | Star rating filter {min: number, max: number} |
| `flexibleDates` | boolean | No | Allow flexible dates (default: false) |
| `accessibilityNeeds` | array | No | Accessibility requirements |
| `cancellationPolicy` | string | No | Preferred cancellation policy |
| `sortBy` | string | No | Sort results: price, rating, distance, popularity |
| `limit` | number | No | Maximum results (default: 10) |

### Response

`200 OK` - SSE stream with accommodation search results

### Errors

- `400` - Invalid request parameters
- `401` - Not authenticated
- `429` - Rate limit exceeded

---

## `POST /api/agents/destinations`

Destination research agent.

**Authentication**: Required  
**Rate Limit Key**: `agents:destinations`  
**Response**: `text/event-stream` (SSE)

### Request Body

| Field | Type | Required | Description |
| ----- | ---- | -------- | ----------- |
| `interests` | array | Yes | Array of interests: adventure, culture, relaxation, nature, food, history, shopping, family, beach, mountain |
| `travelStyle` | string | No | Travel style: relaxation, adventure, culture, family (default: balanced) |
| `budget` | object/string | No | Budget constraints: {min: number, max: number} or "low"/"medium"/"high" |
| `timeOfYear` | string | No | Preferred time: "spring", "summer", "fall", "winter" or month range "Apr-Jun" |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [BjornMelin/tripsage-ai](https://github.com/BjornMelin/tripsage-ai) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
