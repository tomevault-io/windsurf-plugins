---
trigger: always_on
description: Complete guide for adding new service integrations to Agentopia with OAuth, API keys, and tool implementation
---


# Adding New Integrations - Developer Guide

## Overview

This guide walks through the complete process of adding a new service integration to Agentopia. The process involves database configuration, edge function creation, tool registration, and UI integration.

## Integration Types

Agentopia supports three main integration types:

1. **OAuth 2.0**: Services requiring user authorization (Gmail, Microsoft, GitHub)
2. **API Key**: Services using API key authentication (web search, email APIs)
3. **SMTP**: Direct email server connections with credentials

## Step-by-Step Integration Process

### 1. Database Configuration

#### Add Service Provider

First, add the new service to the `service_providers` table:

```sql
-- Example: Adding Slack OAuth integration
INSERT INTO service_providers (
  name,
  display_name,
  provider_type,
  authorization_endpoint,
  token_endpoint,
  scopes_supported,
  configuration_metadata
) VALUES (
  'slack',
  'Slack',
  'oauth',
  'https://slack.com/oauth/v2/authorize',
  'https://slack.com/api/oauth.v2.access',
  '[
    "chat:write",
    "channels:read",
    "groups:read",
    "im:read",
    "mpim:read",
    "users:read"
  ]'::jsonb,
  '{
    "client_id_required": true,
    "pkce_required": false,
    "user_info_endpoint": "https://slack.com/api/users.identity",
    "revoke_endpoint": "https://slack.com/api/auth.revoke",
    "additional_auth_params": {
      "user_scope": "identity.basic,identity.email"
    }
  }'::jsonb
);
```

```sql
-- Example: Adding API key service
INSERT INTO service_providers (
  name,
  display_name,
  provider_type,
  configuration_metadata
) VALUES (
  'openai_api',
  'OpenAI API',
  'api_key',
  '{
    "api_key_header": "Authorization",
    "api_key_prefix": "Bearer ",
    "base_url": "https://api.openai.com/v1",
    "rate_limits": {
      "requests_per_minute": 60,
      "requests_per_hour": 1000
    },
    "supported_models": [
      "gpt-4",
      "gpt-3.5-turbo",
      "text-embedding-ada-002"
    ]
  }'::jsonb
);
```

#### Update Tool Catalog

Add tools to the `tool_catalog` table:

```sql
-- Add Slack tools
INSERT INTO tool_catalog (
  tool_name,
  display_name,
  description,
  category,
  provider_name,
  function_schema,
  is_active
) VALUES 
(
  'slack_send_message',
  'Send Slack Message',
  'Send a message to a Slack channel or direct message',
  'communication',
  'slack',
  '{
    "type": "function",
    "function": {
      "name": "slack_send_message",
      "description": "Send a message to a Slack channel or user",
      "parameters": {
        "type": "object",
        "properties": {
          "channel": {
            "type": "string",
            "description": "Channel ID or name (e.g., #general, @username)"
          },
          "text": {
            "type": "string",
            "description": "Message text to send"
          },
          "blocks": {
            "type": "array",
            "description": "Rich message blocks (optional)",
            "items": {"type": "object"}
          }
        },
        "required": ["channel", "text"]
      }
    }
  }'::jsonb,
  true
),
(
  'slack_list_channels',
  'List Slack Channels',
  'Get a list of channels in the workspace',
  'communication',
  'slack',
  '{
    "type": "function",
    "function": {
      "name": "slack_list_channels",
      "description": "List all channels in the Slack workspace",
      "parameters": {
        "type": "object",
        "properties": {
          "types": {
            "type": "string",
            "description": "Channel types to include (public_channel,private_channel,mpim,im)",
            "default": "public_channel,private_channel"
          },
          "exclude_archived": {
            "type": "boolean",
            "description": "Exclude archived channels",
            "default": true
          }
        }
      }
    }
  }'::jsonb,
  true
);
```

### 2. Create Edge Function

Create a new Supabase Edge Function for the integration:

```typescript
// File: supabase/functions/slack-api/index.ts
import { serve } from 'https://deno.land/std@0.168.0/http/server.ts';
import { createClient } from 'https://esm.sh/@supabase/supabase-js@2.39.7';

interface SlackAPIRequest {
  action: string;
  agent_id: string;
  user_id: string;
  params: Record<string, any>;
}

serve(async (req) => {
  try {
    // Parse request
    const { action, agent_id, user_id, params }: SlackAPIRequest = await req.json();
    
    // Initialize Supabase client
    const supabase = createClient(
      Deno.env.get('SUPABASE_URL')!,
      Deno.env.get('SUPABASE_SERVICE_ROLE_KEY')!
    );
    
    // Get user's Slack credentials
    const credentials = await getUserSlackCredentials(supabase, user_id, agent_id);
    if (!credentials) {
      throw new Error('Question: Your Slack workspace needs to be connected. Please set up your Slack integration in the settings.');
    }
    
    // Execute the requested action
    let result;
    switch (action) {
      case 'send_message':
        result = await sendSlackMessage(credentials.access_token, params);
        break;
      case 'list_channels':
        result = await listSlackChannels(credentials.access_token, params);
        break;
      case 'create_channel':

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/maverick-software) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
