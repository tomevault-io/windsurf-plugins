---
trigger: always_on
description: Strapi 5 REST API patterns for published state, populate parameters, and list-endpoint fallbacks
---


# Strapi 5 REST API Patterns

## **Overview**

This rule defines correct patterns for Strapi 5 Content API calls, particularly for published-state filtering and populate parameters. Incorrect syntax causes 400 Bad Request errors.

## **Problem Solved**

- **status=published** – Can cause validation errors in Strapi 5 setups; use `filters[publishedAt][$notNull]=true` instead
- **Comma-separated populate** – `populate=cover,category,author` triggers 400s; Strapi 5 expects array-style populate
- **Category slug case mismatch** – Strapi may store slugs as `Main-News` while the filter uses `main-news`; use `$eqi` for case-insensitive match

## **Tenant Filtering (Required — No Fallback)**

All directory and news list/detail requests **must** use the tenant filter. Display only data scoped to the current tenant.

### **DO: Always filter by tenant**

- **Directory** (e.g. bishops, parishes): `filters[tenant][tenantId][$eq]=${tenantId}` — no fallback. If the API returns 200 with 0 items, show 0 items.
- **News** (articles, flash-news-items, advertisement-slots): Same. Always include `filters[tenant][tenantId][$eq]=${tenantId}`. Do **not** retry without the tenant filter when the response is 200 with empty data.

### **DON'T: Fall back to no-tenant when response is 200 with 0 items**

```typescript
// ❌ DON'T: Retry without tenant when tenant filter returns 0 items
if (list.length === 0) {
  const noTenant = await fetch('/articles?filters[publishedAt][$notNull]=true&...');
  list = noTenant.data; // Wrong — shows other tenants' data
}
```

When the API returns **200** with an empty list, the correct behavior is to show no items (and ensure entries in Strapi have the tenant relation set). The only exception is the **400 Bad Request** fallback below (different endpoints, different reason).

## **Published State Filter**

### **DO: Use filters[publishedAt][$notNull]=true**

```typescript
// ✅ DO: Filter for published documents
const query = `filters[publishedAt][$notNull]=true`;
```

### **DON'T: Use status=published**

```typescript
// ❌ DON'T: Can cause validation errors in Strapi 5
const query = `status=published`;
```

## **Populate Parameter (Array-Style)**

### **DO: Use array-style populate**

```typescript
// ✅ DO: Array-style populate for Strapi 5
const POPULATE = 'populate[0]=cover&populate[1]=category&populate[2]=author';

// Full example
const path = `/articles?filters[tenant][tenantId][$eq]=${tenantId}&filters[publishedAt][$notNull]=true&populate[0]=cover&populate[1]=category&populate[2]=author&sort=publishedAt:desc&pagination[limit]=10`;
```

### **DON'T: Use comma-separated populate**

```typescript
// ❌ DON'T: Triggers 400 Bad Request in Strapi 5
const POPULATE = 'populate=cover,category,author';
```

## **Example Article Query URLs**

### **Featured News**

```
GET /api/articles?filters[isFeatured][$eq]=true&filters[tenant][tenantId][$eq]=tenant_demo_002&filters[publishedAt][$notNull]=true&populate[0]=cover&populate[1]=category&populate[2]=author&sort=publishedAt:desc&pagination[limit]=6
```

### **Featured News** (use category slug like Main News, not isFeatured)

```
GET /api/articles?filters[category][slug][$eqi]=featured-news&filters[tenant][tenantId][$eq]=tenant_demo_002&filters[publishedAt][$notNull]=true&populate[0]=cover&populate[1]=category&populate[2]=author&sort=publishedAt:desc&pagination[limit]=6
```

If you have a "Featured News" category in Strapi (slug `Featured-News`), filter by category slug. The `isFeatured` boolean is an alternative; use category when articles are assigned to the Featured News category.

### **Main News** (use `$eqi` for case-insensitive slug)

```
GET /api/articles?filters[category][slug][$eqi]=main-news&filters[tenant][tenantId][$eq]=tenant_demo_002&filters[publishedAt][$notNull]=true&populate[0]=cover&populate[1]=category&populate[2]=author&sort=publishedAt:desc&pagination[limit]=10
```

### **Press Release / Most Read**

Same pattern: use `filters[publishedAt][$notNull]=true` and `populate[0]=...&populate[1]=...&populate[2]=...`

## **Category Slug Filter (Case-Insensitive)**

### **DO: Use $eqi for case-insensitive slug match**

Strapi may store category slugs with different casing (e.g. `Main-News`, `Press-Release`) from the UI. Slug filters are case-sensitive; use `$eqi` to match regardless of case:

```typescript
// ✅ DO: Case-insensitive category slug filter
'filters[category][slug][$eqi]=main-news'
'filters[category][slug][$eqi]=press-release'
```

### **DON'T: Use $eq for slugs (case-sensitive)**

```typescript
// ❌ DON'T: May not match "Main-News" when stored with different casing
'filters[category][slug][$eq]=main-news'
```

## **Article Detail Page (slug or id)**

The detail route `/mosc/news/[slug]` and `/syro/news/[slug]` accept slug, numeric id, or documentId:

1. **documentId** (Strapi 5, first when param looks like documentId): `filters[documentId][$eq]=<documentId>` — alphanumeric 15–35 chars (e.g. `o42fs0slvzzj9os0g9xtc11d`). Try documentId first when param matches `/^[a-z0-9]{15,35}$/` so URLs from Strapi admin or links using documentId work even when slug differs.
2. **Slug** (text): `filters[slug][$eqi]=<slug>` — preferred for SEO

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [giventadevelop/md-strikers](https://github.com/giventadevelop/md-strikers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-22 -->
