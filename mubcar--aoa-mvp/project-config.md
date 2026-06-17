---
trigger: always_on
description: AOA (Analyze, Optimize, Automate) is an AI-powered lead recovery platform for home service SMBs in Brazil (HVAC, plumbing, solar, electrical, landscaping). These businesses lose ~40% of inbound leads because owners are physically on jobs when prospects call or message.
---

# AOA MVP — Master Build Prompt

## What is AOA?

AOA (Analyze, Optimize, Automate) is an AI-powered lead recovery platform for home service SMBs in Brazil (HVAC, plumbing, solar, electrical, landscaping). These businesses lose ~40% of inbound leads because owners are physically on jobs when prospects call or message.

AOA deploys AI agents on WhatsApp and voice calls that answer instantly, qualify prospects in natural Brazilian Portuguese, and capture every lead into a real-time dashboard. After qualification, the AI generates a Solana Pay USDC deposit link, moving the prospect from "interested" to "committed." Deposits are held on-chain and released on job completion.

## Architecture Overview

```
LEAD CAPTURE LAYER
├── WhatsApp → Evolution API (webhook) → Fastify backend → Claude API → qualify → Supabase
├── Voice Call → Vapi.ai (voice agent) → webhook → Supabase
└── Both channels feed the same AI brain and the same database

PAYMENT LAYER
├── After qualification → generate Solana Pay link (USDC deposit)
├── Deposit held in escrow (Solana program or simple transfer for MVP)
└── Provider confirms job done → funds release

DASHBOARD
├── React app (Vite + TailwindCSS)
├── Real-time lead cards via Supabase Realtime subscriptions
├── Lead status: new → contacted → deposit_paid → job_complete
└── Simple metrics: leads today, conversion rate, revenue recovered
```

## Tech Stack

| Layer | Technology | Why |
|-------|-----------|-----|
| Backend | Node.js + Fastify | Fast, lightweight, great for webhooks |
| AI | Anthropic Claude API (Sonnet) | Best Portuguese quality, tool use for structured extraction |
| WhatsApp | Evolution API (self-hosted or cloud) | Open-source, free, Brazilian community, REST API |
| Voice | Vapi.ai | Pre-built voice agents, Portuguese TTS/STT, webhook output |
| Database | Supabase (PostgreSQL + Realtime) | Free tier, real-time subscriptions, auth built-in |
| Frontend | React + Vite + TailwindCSS | Fast dev, clean UI, easy deployment |
| Blockchain | Solana Pay SDK + @solana/web3.js | Generate USDC payment links |
| Deployment | Railway (backend) + Vercel (frontend) | Free/cheap tiers, easy deploy |

## Database Schema (Supabase)

### Table: `businesses`
```sql
CREATE TABLE businesses (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  name TEXT NOT NULL,
  slug TEXT UNIQUE NOT NULL,
  phone TEXT,
  whatsapp_number TEXT,
  services TEXT[] DEFAULT '{}',
  service_area TEXT,
  business_hours JSONB DEFAULT '{"start": "08:00", "end": "18:00"}',
  ai_prompt_context TEXT, -- custom info for the AI agent
  solana_wallet_address TEXT,
  created_at TIMESTAMPTZ DEFAULT now()
);
```

### Table: `leads`
```sql
CREATE TABLE leads (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  business_id UUID REFERENCES businesses(id),
  channel TEXT NOT NULL CHECK (channel IN ('whatsapp', 'voice')),
  status TEXT DEFAULT 'new' CHECK (status IN ('new', 'qualifying', 'qualified', 'deposit_sent', 'deposit_paid', 'job_scheduled', 'job_complete', 'lost')),
  
  -- Contact info
  contact_name TEXT,
  contact_phone TEXT,
  
  -- Qualification data (extracted by AI)
  service_needed TEXT,
  urgency TEXT CHECK (urgency IN ('low', 'medium', 'high', 'emergency')),
  problem_description TEXT,
  preferred_schedule TEXT,
  location TEXT,
  
  -- AI conversation
  conversation_summary TEXT,
  raw_messages JSONB DEFAULT '[]',
  
  -- Payment
  deposit_amount_usdc DECIMAL(10,2),
  solana_pay_url TEXT,
  solana_tx_signature TEXT,
  deposit_confirmed_at TIMESTAMPTZ,
  
  -- Timestamps
  created_at TIMESTAMPTZ DEFAULT now(),
  updated_at TIMESTAMPTZ DEFAULT now(),
  qualified_at TIMESTAMPTZ
);
```

### Table: `messages`
```sql
CREATE TABLE messages (
  id UUID DEFAULT gen_random_uuid() PRIMARY KEY,
  lead_id UUID REFERENCES leads(id),
  role TEXT NOT NULL CHECK (role IN ('prospect', 'assistant')),
  content TEXT NOT NULL,
  channel TEXT NOT NULL CHECK (channel IN ('whatsapp', 'voice')),
  created_at TIMESTAMPTZ DEFAULT now()
);
```

## AI Agent System Prompt

The AI agent plays the role of a virtual receptionist for the home service business. The prompt must:

1. Greet warmly in Brazilian Portuguese
2. Identify itself as the business's virtual assistant (e.g., "Olá! Sou a assistente virtual da ClimaTech Refrigeração")
3. Understand what service the prospect needs
4. Assess urgency (broken AC in 38°C = emergency, routine maintenance = low)
5. Capture: name, phone (if WhatsApp doesn't provide), location/neighborhood, preferred schedule
6. Summarize and confirm details
7. Close with: "Um técnico entrará em contato para confirmar o agendamento"
8. After qualification, generate the deposit link message

The AI should use Claude's tool_use to extract structured data:

```json
{
  "tool": "qualify_lead",
  "input": {
    "contact_name": "Maria Santos",
    "service_needed": "Instalação de ar-condicionado split",
    "urgency": "high",
    "problem_description": "Precisa instalar 2 splits no apartamento novo antes de se mudar semana que vem",
    "preferred_schedule": "Segunda ou terça de manhã",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mubcar/aoa-mvp](https://github.com/mubcar/aoa-mvp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
