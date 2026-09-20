---
trigger: always_on
description: Cost control and scaling strategy for EverPrompt
---


# Cost Control & Scaling Strategy

## Core Principles

### 1. **Start Small, Scale Smart**

- Free tier with generous limits
- Paid tier at €5/month for supporters
- No LLM usage for core features
- Database-first approach with minimal external APIs

### 2. **Cost Control from Day 1**

- All costs must be predictable and controllable
- No surprise bills
- Clear usage limits and alerts
- Regular cost monitoring

### 3. **Data Ownership & Backup**

- Full database backups daily
- User data export capabilities
- No vendor lock-in
- Easy migration options

## Free Tier Strategy

### **Free Tier Limits (Generous but Controlled)**

- **Prompts**: 100 prompts per workspace
- **Labels**: 20 labels per workspace
- **Workflows**: 10 workflow collections
- **Storage**: 10MB total (text only)
- **API Calls**: 1000 per month
- **Workspaces**: 1 per user

### **What's Free**

- Core prompt management
- n8n workflow JSON parsing (no LLM needed)
- Basic label system
- Public prompt sharing
- Community library access
- Basic search and filtering

## Paid Tier Strategy (€5/month)

### **Paid Tier Benefits**

- **Unlimited prompts** (10,000+ prompts)
- **Unlimited labels** (100+ labels)
- **Unlimited workflows** (100+ collections)
- **30-day money-back guarantee** (no questions asked)
- **Advanced features** (versioning, collaboration)
- **Export capabilities** (JSON, CSV, PDF)
- **Custom themes** (dark/light mode customization)
- **API access** (for power users)

### **Value Proposition**

- "Support EverPrompt development"
- "Unlock unlimited potential"
- "30-day money-back guarantee"
- "Help build the n8n community"

### **Solo Developer Messaging**

**Transparent Communication:**

- "Built by a solo developer passionate about n8n automation"
- "Your support helps fund development and server costs"
- "We hope for your understanding as we grow together"
- "Community-driven development with your feedback"

### **Money-Back Guarantee Strategy**

- **30-day money-back guarantee** (no questions asked)
- **Solo developer project** - hope for understanding
- **Community support** (Discord/Forum) for all users
- **Comprehensive documentation** and tutorials
- **Export capabilities** - users can always take their data
- **Transparent communication** about project status

## Cost Structure

### **Infrastructure Costs (Monthly)**

- **Vercel Pro**: €20/month (unlimited bandwidth)
- **Neon Database**: €19/month (1GB storage, 100GB transfer)
- **Vercel Blob**: €5/month (100GB storage)
- **Error Tracking**: €0/month (Vercel Analytics + custom logging)

**Free Error Tracking Alternatives:**

- **Vercel Analytics**: Built-in error tracking and performance monitoring
- **Custom Error Logging**: Simple console.error + database logging
- **LogRocket Free Tier**: 1,000 sessions/month (if needed later)
- **Bugsnag Free Tier**: 7,500 errors/month (if needed later)
- **Total**: ~€44/month

### **Revenue Targets**

- **Break-even**: 9 paid users (€44/month)
- **Sustainable**: 25 paid users (€125/month)
- **Growth**: 200+ paid users (€1000+/month)

## No-LLM Architecture

### **JSON Parser (Pure JavaScript)**

```typescript
// No LLM needed - pure regex and parsing
class N8nWorkflowParser {
  parseWorkflow(json: string): ParsedWorkflow {
    // Pure JSON parsing - no AI needed
    const workflow = JSON.parse(json);
    return this.validateWorkflow(workflow);
  }

  extractPrompts(workflow: ParsedWorkflow): ExtractedPrompt[] {
    // Regex-based extraction - no AI needed
    const prompts: ExtractedPrompt[] = [];

    workflow.nodes.forEach((node) => {
      if (this.isAINode(node)) {
        prompts.push(...this.extractFromNode(node));
      }
    });

    return prompts;
  }
}
```

### **Prompt Categorization (Rule-Based)**

```typescript
// Rule-based categorization - no LLM needed
class PromptCategorizer {
  categorizePrompt(prompt: ExtractedPrompt): string[] {
    const categories: string[] = [];

    // Node type categorization
    if (prompt.nodeType.includes("perplexity")) categories.push("search");
    if (prompt.nodeType.includes("openai")) categories.push("generation");
    if (prompt.nodeType.includes("chatgpt")) categories.push("conversation");

    // Content-based categorization
    if (prompt.content.includes("summarize")) categories.push("summarization");
    if (prompt.content.includes("translate")) categories.push("translation");
    if (prompt.content.includes("analyze")) categories.push("analysis");

    return categories;
  }
}
```

## Database Strategy

### **PostgreSQL-Only Approach**

- **Primary DB**: Neon PostgreSQL
- **Full-text search**: PostgreSQL built-in
- **Caching**: Redis (optional, can start without)
- **File storage**: Vercel Blob (for future attachments)

### **Backup Strategy**

```sql
-- Daily automated backups
pg_dump $DATABASE_URL > backup_$(date +%Y%m%d).sql

-- Weekly full backups
pg_dump --format=custom $DATABASE_URL > weekly_backup_$(date +%Y%m%d).dump
```

### **Data Export**

```typescript
// User data export
interface UserDataExport {
  prompts: Prompt[];
  labels: Label[];
  workflows: WorkflowCollection[];
  settings: UserSettings;
  exportDate: Date;
  version: string;
}
```

## Scaling Strategy

### **Phase 1: MVP (0-100 users)**

- **Cost**: €70/month
- **Revenue**: €0-500/month

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [mitsue-eth/everprompt-n8n-shadcn](https://github.com/mitsue-eth/everprompt-n8n-shadcn) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-09-20 -->
