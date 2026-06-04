---
trigger: always_on
description: This document provides a comprehensive reference of table compositions in the ResearchHub database.
---

# ResearchHub Database Tables (Cursor Rule)

This document provides a comprehensive reference of table compositions in the ResearchHub database.

## User Management

### nonprofit_org

**Description**: Nonprofit organizations that can receive donations.

| Column               | Type                     | Description                        |
| ----- | --- | ---- |
| id                   | integer                  | Primary key                        |
| name                 | character varying(255)   | Name of the nonprofit              |
| ein                  | character varying(20)    | Employer Identification Number     |
| endaoment_org_id     | character varying(100)   | Unique ID in Endaoment system      |
| base_wallet_address  | character varying(42)    | Blockchain wallet address          |
| created_date         | timestamp with time zone | Creation timestamp                 |
| updated_date         | timestamp with time zone | Last update timestamp              |

### nonprofit_fundraise_link

**Description**: Join table connecting nonprofits and fundraising campaigns.

| Column         | Type                       | Description                          |
| ----- | ----- | --- |
| id             | integer                    | Primary key                          |
| nonprofit_id   | integer                    | Foreign key to nonprofit_org         |
| fundraise_id   | integer                    | Foreign key to purchase_fundraise    |
| note           | text                       | Notes about this specific link       |
| created_date   | timestamp with time zone   | Creation timestamp                   |
| updated_date   | timestamp with time zone   | Last update timestamp                |

### user_user

**Description**: Core user account table.

| Column         | Type                       | Description                          |
| -------------- | -------------------------- | ------------------------------------ |
| id             | integer                    | Primary key                          |
| username       | character varying          | Unique username                      |
| email          | character varying          | User's email address                 |
| password       | character varying          | Hashed password                      |
| first_name     | character varying          | User's first name                    |
| last_name      | character varying          | User's last name                     |
| is_active      | boolean                    | Whether the account is active        |
| date_joined    | timestamp with time zone   | When the user joined                 |
| reputation     | integer                    | User's reputation score              |
| is_staff       | boolean                    | Whether user has staff privileges    |
| is_superuser   | boolean                    | Whether user has superuser privileges|

### user_author

**Description**: Author profile information linked to users.

| Column             | Type                   | Description                         |
| ------------------ | ---------------------- | ----------------------------------- |
| id                 | integer                | Primary key                         |
| first_name         | character varying      | Author's first name                 |
| last_name          | character varying      | Author's last name                  |
| user_id            | integer                | Foreign key to user_user            |
| orcid              | character varying      | ORCID identifier                    |
| scopus_author_id   | character varying      | Scopus author identifier            |
| google_scholar_id  | character varying      | Google Scholar identifier           |
| twitter            | character varying      | Twitter handle                      |
| linkedin           | character varying      | LinkedIn profile URL                |
| website            | character varying      | Personal website URL                |

### user_userverification

**Description**: User verification information.

| Column         | Type                       | Description                          |
| -------------- | -------------------------- | ------------------------------------ |
| id             | integer                    | Primary key                          |
| first_name     | text                       | User's first name                    |
| last_name      | text                       | User's last name                     |
| status         | text                       | Verification status                  |
| verified_by    | text                       | Verification method                  |
| external_id    | text                       | External identifier                  |
| created_date   | timestamp with time zone   | Creation timestamp                   |
| updated_date   | timestamp with time zone   | Last update timestamp                |
| user_id        | integer                    | Foreign key to user_user             |

### user_follow

**Description**: User following relationships.

| Column         | Type                       | Description                          |

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ResearchHub/web](https://github.com/ResearchHub/web) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
