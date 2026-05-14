---
trigger: always_on
description: name: SmartLead MCP Tools Reference
---

---
name: SmartLead MCP Tools Reference
description: Complete reference guide for all 116+ MCP tools organized by category
author: LeadMagic Team
version: 1.0.0
# 🛠️ SmartLead MCP Tools - Complete Implementation Guide

> **Tool Master Reference**: Your comprehensive guide to all 116+ MCP tools in the SmartLead server. Find tools by category, understand implementation patterns, and master the complete SmartLead automation toolkit.

## 🎯 **Quick Tool Finder**

### **🔥 Most Used Tools**
1. **Campaign Management**: `smartlead_create_campaign`, `smartlead_list_campaigns`
2. **Lead Management**: `smartlead_add_leads_to_campaign`, `smartlead_list_leads_by_campaign`
3. **Analytics**: `smartlead_get_analytics_overall_stats_v2`, `smartlead_fetch_campaign_statistics`
4. **Email Accounts**: `smartlead_create_email_account`, `smartlead_fetch_all_email_accounts`

### **📊 Tool Categories Overview**
- 🎯 **Campaign Tools**: 14 tools for campaign management
- 👥 **Lead Tools**: 17 tools for prospect management
- 📧 **Email Account Tools**: 15 tools for account management
- 📊 **Analytics Tools**: 18 tools for reporting
- 📈 **Statistics Tools**: 18 tools for metrics
- 🚀 **Smart Delivery Tools**: 11 tools for deliverability
- 🤖 **Smart Senders Tools**: 12 tools for domain management
- 🔗 **Webhook Tools**: 9 tools for automation
- 👤 **Client Management Tools**: 8 tools for team management

## 🛠️ **Tool Categories Reference**

### **🎯 Campaign Tools** (`src/tools/campaigns.ts`)
> **14 tools** for complete campaign lifecycle management

#### **🚀 Core Campaign Operations**
```typescript
// Create and manage campaigns
smartlead_create_campaign              // Create new email campaigns
smartlead_update_campaign_schedule     // Modify sending schedules
smartlead_get_campaigns_with_analytics // Get campaigns with performance data
smartlead_list_campaigns              // List all campaigns with filtering
smartlead_get_campaign_by_id          // Fetch specific campaign details
smartlead_update_campaign_settings    // Modify campaign configuration
smartlead_delete_campaign             // Remove campaigns
```

#### **⚙️ Campaign Configuration**
```typescript
// Advanced campaign settings
smartlead_pause_campaign              // Pause active campaigns
smartlead_resume_campaign             // Resume paused campaigns
smartlead_clone_campaign              // Duplicate existing campaigns
smartlead_update_campaign_sequences   // Modify email sequences
smartlead_set_campaign_limits         // Configure sending limits
smartlead_update_campaign_tracking    // Enable/disable tracking
smartlead_schedule_campaign           // Set up campaign scheduling
```

**💡 Use Cases:**
- 🎯 **New Campaign Setup**: Use `create_campaign` → `add_leads_to_campaign` → `resume_campaign`
- 📊 **Performance Review**: Use `get_campaigns_with_analytics` for overview
- ⚙️ **Campaign Optimization**: Use `update_campaign_settings` and `update_campaign_sequences`

### **👥 Lead Tools** (`src/tools/leads.ts`)
> **17 tools** for comprehensive prospect and lead management

#### **📥 Lead Import & Management**
```typescript
// Core lead operations
smartlead_add_leads_to_campaign        // Import prospects to campaigns (bulk)
smartlead_list_leads_by_campaign       // Get all prospects in a campaign
smartlead_fetch_lead_by_email          // Find specific prospect by email
smartlead_update_lead_category         // Categorize prospects (interested, not_interested, etc.)
smartlead_remove_lead_from_campaign    // Remove prospects from campaigns
smartlead_get_lead_details             // Fetch detailed prospect information
```

#### **📊 Lead Analytics & Tracking**
```typescript
// Lead performance and tracking
smartlead_get_lead_activity_history    // View prospect interaction timeline
smartlead_fetch_lead_email_history     // Get all emails sent to prospect
smartlead_get_lead_response_data       // Analyze prospect responses
smartlead_track_lead_engagement        // Monitor engagement metrics
smartlead_get_lead_conversion_data     // Track conversion funnel
```

#### **🔄 Lead Automation**
```typescript
// Automated lead management
smartlead_bulk_update_leads            // Update multiple prospects at once
smartlead_auto_categorize_leads        // AI-powered lead categorization
smartlead_schedule_lead_followup       // Set up automated follow-ups
smartlead_sync_leads_with_crm          // CRM integration and sync
smartlead_export_leads_data            // Export prospect data
smartlead_import_leads_from_csv        // Bulk import from CSV files
```

**💡 Use Cases:**
- 📥 **Lead Import**: Use `import_leads_from_csv` → `add_leads_to_campaign`
- 📊 **Performance Analysis**: Use `get_lead_activity_history` and `get_lead_response_data`
- 🔄 **Lead Management**: Use `update_lead_category` and `bulk_update_leads`

### **📧 Email Account Tools** (`src/tools/email-accounts.ts`)
> **15 tools** for email account management and warmup

#### **🔧 Account Setup & Configuration**
```typescript
// Email account management
smartlead_create_email_account         // Add new sending accounts
smartlead_fetch_all_email_accounts     // List all email accounts
smartlead_update_email_account         // Modify account settings
smartlead_delete_email_account         // Remove email accounts

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LeadMagic/smartlead-mcp-server](https://github.com/LeadMagic/smartlead-mcp-server) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-13 -->
