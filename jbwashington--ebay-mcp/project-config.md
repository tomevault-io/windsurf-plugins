---
trigger: always_on
description: This document provides context for Claude Code when working with this project.
---

# eBay Listing Automation Plugin - Memory & Context

This document provides context for Claude Code when working with this project.

## Project Identity

**Name**: eBay Listing Automation Plugin
**Type**: Claude Code Plugin
**Purpose**: Automate eBay listing creation, pricing research, fraud detection, and order fulfillment
**Primary Use Case**: Listing clothing, sneakers, and accessories on eBay

## Tech Stack

- **Runtime**: Bun (TypeScript)
- **Integration**: Model Context Protocol (MCP)
- **APIs**: eBay Developer API (Inventory, Browse, Fulfillment, Feedback)
- **Authentication**: OAuth 2.0 with Tailscale Funnel for HTTPS
- **Deployment**: Claude Code Plugin (local)

## Project Structure

```
ebay-listing-automation/
├── .claude-plugin/plugin.json    # Plugin manifest
├── agents/                        # 5 specialized AI agents
├── commands/                      # 10 slash commands
├── hooks/                         # 4 automation hooks
├── scripts/                       # Hook scripts
└── src/                          # MCP server & libs
    ├── mcp-server/               # Main MCP server
    ├── lib/                      # API clients & utilities
    ├── scripts/                  # OAuth authentication
    └── types/                    # TypeScript definitions
```

## Core Components

### 1. MCP Server (21 Tools)
Located in `src/mcp-server/index.ts`

**Inventory Management**:
- `ebay_create_inventory_item` - Create/update inventory
- `ebay_create_offer` - Create draft listing
- `ebay_publish_offer` - Publish to eBay

**Search & Research**:
- `ebay_search_by_image` - Visual similarity search
- `ebay_search_by_keyword` - Text-based search

**Order Fulfillment**:
- `ebay_get_orders` - Fetch order list
- `ebay_get_order` - Get order details
- `ebay_create_shipping_fulfillment` - Add tracking

**Best Offer Management** (Trading API):
- `ebay_get_best_offers` - Get buyer offers on a listing (filter by status)
- `ebay_respond_to_best_offer` - Accept, decline, or counter a buyer Best Offer

**Promoted Listings (Marketing API)**:
- `ebay_create_promotion_campaign` - Create a cost-per-sale ad campaign
- `ebay_get_campaigns` - List campaigns (filter by status)
- `ebay_add_listings_to_campaign` - Add listings with individual bid rates
- `ebay_get_ad_recommendations` - Get eBay-suggested bid rates
- `ebay_pause_campaign` - Pause a running campaign
- `ebay_resume_campaign` - Resume a paused campaign
- `ebay_end_campaign` - Permanently end a campaign

**Fraud Detection**:
- `ebay_get_buyer_info` - Buyer account info
- `ebay_calculate_fraud_risk` - Risk scoring

**Utilities**:
- `ebay_get_policies` - Seller policies
- `ebay_leave_feedback` - Buyer feedback

### 2. AI Agents (5 Specialized)

**Listing Creator** (`agents/listing-creator.md`)
- End-to-end listing workflow from photos to publication
- Invoked when: "List these items", "Create eBay listing"

**Price Researcher** (`agents/price-researcher.md`)
- Market analysis and competitive pricing
- Invoked when: "What should I price this at?", "Research pricing"

**Image Analyzer** (`agents/image-analyzer.md`)
- Product identification and condition assessment
- Invoked when: "What product is this?", "Analyze these photos"

**Fulfillment Manager** (`agents/fulfillment-manager.md`)
- Order processing, shipping, feedback
- Invoked when: "Ship my orders", "Check orders"

**Fraud Analyst** (`agents/fraud-analyst.md`)
- Buyer risk assessment and fraud detection
- Invoked when: "Check this buyer", "Is this fraud?"

### 3. Slash Commands (10 Manual Controls)

Authentication & Setup:
- `/ebay-auth` - OAuth with Tailscale Funnel
- `/ebay-status` - Health check

Listing Workflow:
- `/ebay-analyze [folder]` - Analyze product photos
- `/ebay-draft [folder]` - Create draft listing
- `/ebay-publish [offer-id]` - Publish to eBay

Order Management:
- `/ebay-orders` - View orders
- `/ebay-ship [order-id]` - Process shipping
- `/ebay-feedback [item-id] [tx-id]` - Leave feedback

Utilities:
- `/ebay-policies [type]` - View seller policies
- `/ebay-fraud-analysis [username|order-id]` - Fraud check

## Workflow Patterns

### Complete Listing Workflow
```
1. User organizes photos in folder: items/nike-jordan-1/
2. User: "List these Nike shoes"
3. Listing Creator agent:
   - Analyzes photos (Image Analyzer capabilities)
   - Researches pricing (Price Researcher capabilities)
   - Creates draft listing
   - Asks for approval
4. User: "Publish it"
5. Agent publishes listing
6. Hook: Notifies user to package item
```

### Manual Workflow (Alternative)
```
/ebay-analyze items/nike-jordan-1
/ebay-draft items/nike-jordan-1
/ebay-publish [offer-id]
```

### Order Fulfillment Workflow
```
1. Item sells
2. User: "Ship my orders" or /ebay-orders
3. Fulfillment Manager shows pending orders
4. User provides tracking number
5. Agent marks as shipped via API
6. After delivery: leave feedback automatically
```

### Best Offer Workflow
```
1. Buyer sends Best Offer on listing
2. User: "Check offers on item 12345" or "Respond to the offer"
3. Agent fetches offers via ebay_get_best_offers
4. Agent analyzes offer vs. listing price, buyer feedback
5. Agent recommends: accept, counter (with suggested price), or decline
6. User confirms action

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [jbwashington/ebay-mcp](https://github.com/jbwashington/ebay-mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-25 -->
