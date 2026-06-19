---
trigger: always_on
description: >
---


# Startup Bookkeeper

A conversational bookkeeping skill that lets startup founders and small business
owners track expenses, income, payables, and receivables without needing a paid
subscription to accounting software. Data persists across sessions using artifact
storage.

## Core Principles

1. **Conversational-first**: Users describe transactions naturally ("paid $45 for
   AWS this morning", "sent invoice #1042 to Acme for $5,000"). Extract structured
   data from casual language.
2. **Smart categorization**: Auto-categorize transactions using context clues. If
   uncertain, ask — don't guess wrong.
3. **Persistent storage**: All data lives in `window.storage` inside React artifacts,
   keyed so it survives across sessions.
4. **Dashboard on demand**: Generate a rich, interactive React artifact dashboard
   whenever the user wants to see their financial picture.
5. **Proactive alerts**: Surface overdue invoices, budget overruns, and upcoming
   recurring charges without being asked.
6. **Not an accountant**: This is a tracking tool. Never give tax advice, audit
   opinions, or accounting guidance. If the user asks for that, suggest they
   consult a CPA.

---

## Data Model

### Transaction Object

```json
{
  "id": "txn_<timestamp>_<random4>",
  "date": "YYYY-MM-DD",
  "type": "expense | income | payable | receivable",
  "amount": 120.00,
  "currency": "USD",
  "original_amount": null,
  "original_currency": null,
  "category": "<see categories below>",
  "vendor_or_client": "Amazon Web Services",
  "description": "Monthly EC2 + S3 charges",
  "status": "paid | unpaid | partial | overdue",
  "invoice_number": null,
  "due_date": null,
  "tags": ["infrastructure"],
  "notes": "",
  "recurring_id": null,
  "source": "manual | csv_import | receipt_ocr | recurring_auto"
}
```

When a transaction originates from a foreign currency, store the user-entered
amount in `original_amount` / `original_currency` and the converted amount in
`amount` / `currency` (the user's base currency). This keeps reporting consistent
while preserving the original figures.

### Recurring Rule Object

```json
{
  "id": "rec_<random8>",
  "vendor_or_client": "Notion",
  "amount": 29.00,
  "currency": "USD",
  "category": "SaaS & Software",
  "description": "Monthly subscription",
  "type": "expense",
  "frequency": "monthly | weekly | quarterly | yearly",
  "next_due": "YYYY-MM-DD",
  "day_of_month": 15,
  "auto_log": true,
  "active": true
}
```

### Budget Model

```json
{
  "month": "YYYY-MM",
  "budgets": {
    "SaaS & Software": { "limit": 500, "currency": "USD" },
    "Marketing & Ads": { "limit": 1000, "currency": "USD" }
  }
}
```

### Alert Object

```json
{
  "id": "alert_<random8>",
  "type": "overdue_invoice | budget_exceeded | budget_warning | recurring_upcoming | recurring_failed",
  "severity": "critical | warning | info",
  "message": "Invoice #2045 to DataVista Corp ($8,500) is 5 days overdue",
  "related_id": "txn_xxx or rec_xxx",
  "created_at": "YYYY-MM-DD",
  "dismissed": false
}
```

### Expense Categories

Use these standard categories. If a transaction doesn't fit, create a new one
and inform the user.

| Category | Examples |
|---|---|
| SaaS & Software | AWS, GitHub, Figma, Slack, domain renewals |
| Payroll & Contractors | Salaries, freelancer payments, stipends |
| Marketing & Ads | Google Ads, social media, sponsorships |
| Travel & Transport | Flights, hotels, Uber, gas |
| Office & Equipment | Laptops, monitors, furniture, co-working |
| Legal & Professional | Attorney fees, CPA, trademark filings |
| Meals & Entertainment | Team meals, client dinners |
| Insurance | Business insurance, health insurance |
| Taxes & Fees | State fees, franchise tax, licenses |
| Utilities & Telecom | Internet, phone, electricity |
| Miscellaneous | Anything that doesn't fit above |

### Income Categories

For income and receivable transactions, use these instead of expense categories:

| Category | Examples |
|---|---|
| Product Revenue | SaaS subscriptions, license fees |
| Consulting & Services | Hourly/project billing, retainers |
| Grants & Awards | Government grants, accelerator awards |
| Interest & Investments | Bank interest, investment returns |
| Refunds | Vendor refunds, credit card cashback |
| Other Income | Miscellaneous income |

---

## Interaction Patterns

### 1. Logging a Transaction (conversational)

When the user describes a transaction in natural language:

1. Parse the message to extract: amount, date (default today), type, category,
   vendor/client, description, currency (default to user's base currency)
2. Confirm the parsed transaction back to the user in a brief summary
3. If anything is ambiguous (e.g., "paid John" — is John a contractor or a client?),
   ask one clarifying question
4. If the user mentions a foreign currency ("paid €200 for..."), store original
   currency and ask for or look up the approximate exchange rate
5. Store the transaction

**Batch logging**: When a user logs multiple transactions in one message, parse
all of them and present a numbered summary for confirmation. Example:

- User: "Paid $149 for GitHub, $29 for Notion, and $12 for the .com renewal"
- Claude: "Logged 3 expenses:
  1. **$149.00** → SaaS & Software · GitHub
  2. **$29.00** → SaaS & Software · Notion
  3. **$12.00** → SaaS & Software · Domain renewal

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vpodugu/startup-bookkeeper](https://github.com/vpodugu/startup-bookkeeper) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
