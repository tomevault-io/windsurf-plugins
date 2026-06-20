---
trigger: always_on
description: Manage CRM contacts, companies, deals, tickets, pipelines, engagement activities, lists, and marketing emails via the HubSpot API.
---

# HubSpot

Manage CRM contacts, companies, deals, tickets, pipelines, engagement activities, lists, and marketing emails via the HubSpot API.

All commands go through `skill_exec` using CLI-style syntax.
Use `--help` at any level to discover actions and arguments.

## Contacts

### List contacts

```
hubspot list_contacts --limit 20 --properties '["email","firstname","lastname","phone","company"]'
```

| Argument     | Type     | Required | Default | Description               |
| ------------ | -------- | -------- | ------- | ------------------------- |
| `limit`      | int      | no       | 10      | Results to return (1-100) |
| `after`      | string   | no       |         | Pagination cursor         |
| `properties` | string[] | no       |         | Properties to include     |

Returns: list of `id`, `properties` (email, firstname, lastname, etc.), `createdAt`, `updatedAt`.

### Get contact

```
hubspot get_contact --contact_id "123" --properties '["email","firstname","lastname","phone","company","lifecyclestage"]'
```

| Argument     | Type     | Required | Description           |
| ------------ | -------- | -------- | --------------------- |
| `contact_id` | string   | yes      | Contact ID            |
| `properties` | string[] | no       | Properties to include |

Returns: `id`, `properties`, `createdAt`, `updatedAt`, `associations`.

### Create contact

```
hubspot create_contact --email "jane@example.com" --firstname "Jane" --lastname "Doe" --phone "+1234567890" --company "Acme Corp" --properties '{"lifecyclestage":"lead"}'
```

| Argument     | Type   | Required | Description                        |
| ------------ | ------ | -------- | ---------------------------------- |
| `email`      | string | yes      | Contact email                      |
| `firstname`  | string | no       | First name                         |
| `lastname`   | string | no       | Last name                          |
| `phone`      | string | no       | Phone number                       |
| `company`    | string | no       | Company name                       |
| `properties` | object | no       | Additional properties as key-value |

Returns: `id`, `properties`, `createdAt`.

### Update contact

```
hubspot update_contact --contact_id "123" --properties '{"lifecyclestage":"customer","phone":"+9876543210"}'
```

| Argument     | Type   | Required | Description                     |
| ------------ | ------ | -------- | ------------------------------- |
| `contact_id` | string | yes      | Contact ID                      |
| `email`      | string | no       | Updated email                   |
| `firstname`  | string | no       | Updated first name              |
| `lastname`   | string | no       | Updated last name               |
| `properties` | object | no       | Additional properties to update |

Returns: `id`, `properties`, `updatedAt`.

### Delete contact

```
hubspot delete_contact --contact_id "123"
```

| Argument     | Type   | Required | Description |
| ------------ | ------ | -------- | ----------- |
| `contact_id` | string | yes      | Contact ID  |

Returns: `success` boolean.

### Search contacts

```
hubspot search_contacts --query "jane@example.com" --filter_groups '[{"filters":[{"propertyName":"lifecyclestage","operator":"EQ","value":"customer"}]}]' --limit 10
```

| Argument        | Type     | Required | Default | Description                |
| --------------- | -------- | -------- | ------- | -------------------------- |
| `query`         | string   | no       |         | Free-text search query     |
| `filter_groups` | object[] | no       |         | HubSpot filter groups JSON |
| `sorts`         | object[] | no       |         | Sort criteria JSON         |
| `properties`    | string[] | no       |         | Properties to return       |
| `limit`         | int      | no       | 10      | Results to return (1-100)  |

Returns: list of matching contact objects.

## Companies

### List companies

```
hubspot list_companies --limit 20 --properties '["name","domain","industry","numberofemployees"]'
```

| Argument     | Type     | Required | Default | Description               |
| ------------ | -------- | -------- | ------- | ------------------------- |
| `limit`      | int      | no       | 10      | Results to return (1-100) |
| `after`      | string   | no       |         | Pagination cursor         |
| `properties` | string[] | no       |         | Properties to include     |

Returns: list of `id`, `properties` (name, domain, industry, etc.), `createdAt`, `updatedAt`.

### Get company

```
hubspot get_company --company_id "456" --properties '["name","domain","industry","annualrevenue"]'
```

| Argument     | Type     | Required | Description           |
| ------------ | -------- | -------- | --------------------- |
| `company_id` | string   | yes      | Company ID            |
| `properties` | string[] | no       | Properties to include |

Returns: `id`, `properties`, `createdAt`, `updatedAt`, `associations`.

### Create company

```
hubspot create_company --properties '{"name":"Acme Corp","domain":"acme.com","industry":"Technology","numberofemployees":"500"}'
```


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [officeos-co/skill-hubspot](https://github.com/officeos-co/skill-hubspot) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
