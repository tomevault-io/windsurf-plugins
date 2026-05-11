---
trigger: always_on
description: > Last Updated: 2025-10-06
---

# Red Energy API Response Structure

> Last Updated: 2025-10-06
> 
> This document defines the actual API response structures returned by the Red Energy API and how they map to our internal data model.

## Overview

The Red Energy API returns data in a specific structure that differs from typical REST API conventions. This document serves as the authoritative reference for understanding and validating API responses.

---

## 1. Properties/Accounts Response

### Endpoint
`GET /api/properties` or similar endpoint

### Actual API Response Structure

```json
[
  {
    "propertyPhysicalNumber": 82227160,
    "propertyNumber": "82227160.8490263",
    "accountNumber": 8490263,
    "address": {
      "unit": null,
      "unitType": null,
      "house": "27",
      "floor": null,
      "building": null,
      "street": "SUNNYSIDE CRES",
      "streetType": null,
      "suburb": "CASTLECRAG",
      "pobox": null,
      "townCity": null,
      "postcode": "2068",
      "state": "NSW",
      "country": null,
      "gentrackDisplayAddress": "27 SUNNYSIDE CRES, CASTLECRAG, NSW 2068",
      "displayAddresses": {
        "shortForm": "27 Sunnyside Crescent, Castlecrag",
        "shortFormAlt": "27 Sunnyside Crescent, Castlecrag",
        "extraShortForm": "27 Sunnyside Crescent",
        "longForm": "27 Sunnyside Crescent\nCastlecrag NSW 2068",
        "longFormAlt": "27 Sunnyside Crescent, Castlecrag, New South Wales 2 0 6 8"
      },
      "displayAddress": "27 SUNNYSIDE CRES\nCASTLECRAG  NSW  2068"
    },
    "consumers": [
      {
        "consumerNumber": 4235478511,
        "propertyNumber": "82227160.8490263",
        "accountNumber": 8490263,
        "entryDate": "2024-09-13",
        "finalDate": null,
        "status": "ON",
        "nmi": "4103296839",
        "nmiWithChecksum": "41032968395",
        "utility": "E",
        "meterType": "INTERVAL",
        "chargeClass": "RES",
        "solar": true,
        "lastBillDate": "2025-09-10",
        "nextBillDate": "2025-10-11",
        "latitude": -33.799045,
        "longitude": 151.212185,
        "balanceDollar": -75.0,
        "arrearsDollar": 0.0,
        "productName": "Qantas Red Saver",
        "linesCompany": "Ausgrid",
        "jurisdiction": "NSW",
        "billingFrequency": "MONTHLY"
      }
    ]
  }
]
```

### Key Field Mappings

| API Field | Our Internal Field | Notes |
|-----------|-------------------|-------|
| `accountNumber` | `id` | Primary identifier for the property |
| `consumers` | `services` | Array of services (electricity/gas) |
| No direct field | `name` | Built from `address.displayAddresses.shortForm` or address parts |
| `address` | `address` | Transformed to our address structure |

### Property ID Resolution

The integration looks for property ID in this order:
1. `data.get("id")`
2. `data.get("propertyId")`
3. `data.get("property_id")`
4. `data.get("accountNumber")` ✅ **Used by Red Energy API**
5. Generated from address if none found

---

## 2. Consumer/Service Structure

### Actual API Structure

```json
{
  "consumerNumber": 4235478511,
  "accountNumber": 8490263,
  "utility": "E",
  "status": "ON",
  "nmi": "4103296839",
  "meterType": "INTERVAL",
  "solar": true,
  "productName": "Qantas Red Saver",
  "linesCompany": "Ausgrid",
  "balanceDollar": -75.0
}
```

### Field Mappings

| API Field | Our Internal Field | Transformation |
|-----------|-------------------|----------------|
| `consumerNumber` | `consumer_number` | Convert to string |
| `utility` | `type` | `"E"` → `"electricity"`, `"G"` → `"gas"` |
| `status` | `active` | `"ON"` → `true`, `"OFF"` → `false` |

### Utility Code Mapping

```python
# API → Internal
"E" → "electricity"
"G" → "gas"
```

### Status Mapping

```python
# API → Internal
"ON" → True
"OFF" → False
```

---

## 3. Address Structure

### Actual API Structure

```json
{
  "unit": null,
  "unitType": null,
  "house": "27",
  "floor": null,
  "building": null,
  "street": "SUNNYSIDE CRES",
  "streetType": null,
  "suburb": "CASTLECRAG",
  "pobox": null,
  "townCity": null,
  "postcode": "2068",
  "state": "NSW",
  "country": null,
  "gentrackDisplayAddress": "27 SUNNYSIDE CRES, CASTLECRAG, NSW 2068",
  "displayAddresses": {
    "shortForm": "27 Sunnyside Crescent, Castlecrag",
    "shortFormAlt": "27 Sunnyside Crescent, Castlecrag",
    "extraShortForm": "27 Sunnyside Crescent",
    "longForm": "27 Sunnyside Crescent\nCastlecrag NSW 2068",
    "longFormAlt": "27 Sunnyside Crescent, Castlecrag, New South Wales 2 0 6 8"
  },
  "displayAddress": "27 SUNNYSIDE CRES\nCASTLECRAG  NSW  2068"
}
```

### Nullable Fields

Address fields can be `null` in the API (e.g. unit-only, PO Box, or incomplete data). Validation must use `(data.get("field") or "").strip()` so that `None` does not cause `AttributeError: 'NoneType' object has no attribute 'strip'`.

### Field Mappings

| API Field | Our Internal Field | Transformation |
|-----------|-------------------|----------------|
| `house` + `street` | `street` | Combined: `"27 SUNNYSIDE CRES"` (handle null) |
| `suburb` | `city` | Direct mapping |
| `state` | `state` | Direct mapping |
| `postcode` | `postcode` | Direct mapping |

### Display Address Priority

For property names, we use in order:
1. `displayAddresses.shortForm` ✅ **Preferred** - "27 Sunnyside Crescent, Castlecrag"

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [craibo/ha-red-energy-au](https://github.com/craibo/ha-red-energy-au) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-07 -->
