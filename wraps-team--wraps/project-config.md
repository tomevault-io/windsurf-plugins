---
trigger: always_on
description: Wraps is a CLI and TypeScript SDK that deploys production-ready email, SMS, and CDN infrastructure to a user's AWS account. Zero credentials stored. OIDC authentication. The user owns everything.
---

# AGENTS.md - Wraps

Wraps is a CLI and TypeScript SDK that deploys production-ready email, SMS, and CDN infrastructure to a user's AWS account. Zero credentials stored. OIDC authentication. The user owns everything.

## Prerequisites

- Node.js 20+
- AWS credentials configured (`aws configure` or environment variables)
- A verified domain (for email)

## Install

```bash
# Deploy infrastructure (no install needed)
npx @wraps.dev/cli email init

# Or install globally
npm install -g @wraps.dev/cli

# SDKs
npm install @wraps.dev/email
npm install @wraps.dev/sms
```

## CLI Commands

### Email

```
wraps email init          Deploy email infrastructure (SES, Lambda, DynamoDB, EventBridge)
  -p, --provider          Hosting provider (vercel, aws, railway, other)
  -r, --region            AWS region
  -d, --domain            Domain name
  --preset                Config preset (starter, production, enterprise, custom)
  -y, --yes               Skip confirmation
  --preview               Preview changes without deploying

wraps email connect       Connect to existing SES infrastructure
  -r, --region            AWS region
  --preview               Preview changes

wraps email status        Show email infrastructure details
  --account               AWS account ID
  -r, --region            AWS region

wraps email check [domain]  Check email deliverability
  -q, --quick             Fewer checks (top blacklists, fewer DKIM selectors)
  -j, --json              Output as JSON
  --verbose               Show all checks including passing
  --dkimSelector          Specific DKIM selector
  --skipBlacklists        Skip blacklist checks
  --skipTls               Skip MX TLS checks
  --timeout               DNS timeout in ms

wraps email verify        Verify domain DNS records
  -d, --domain            Domain name (required)

wraps email config        Apply CLI updates to infrastructure
  -r, --region            AWS region
  -y, --yes               Skip confirmation

wraps email upgrade       Add features to existing deployment
  -r, --region            AWS region
  -y, --yes               Skip confirmation
  --preview               Preview changes

wraps email restore       Restore from saved metadata
  -r, --region            AWS region
  -f, --force             Force without confirmation
  --preview               Preview changes

wraps email destroy       Remove all email infrastructure
  -f, --force             Force without confirmation
  -r, --region            AWS region
  --preview               Preview changes
```

### Email Domains

```
wraps email domains add       Add a domain to SES
  -d, --domain                Domain name (required)

wraps email domains list      List all SES domains with status

wraps email domains get-dkim  Get DKIM tokens for DNS configuration
  -d, --domain                Domain name (required)

wraps email domains verify    Verify DKIM, SPF, DMARC records
  -d, --domain                Domain name (required)

wraps email domains remove    Remove a domain from SES
  -d, --domain                Domain name (required)
  -f, --force                 Skip confirmation
```

### Email Inbound

```
wraps email inbound init      Enable inbound email receiving
  -r, --region                AWS region
  -d, --domain                Subdomain for inbound
  -y, --yes                   Skip confirmation
  --preview                   Preview changes

wraps email inbound status    Show inbound email status
  -r, --region                AWS region

wraps email inbound verify    Verify inbound DNS records
  -r, --region                AWS region

wraps email inbound test      Send test email and verify receipt
  -r, --region                AWS region

wraps email inbound destroy   Remove inbound infrastructure
  -r, --region                AWS region
  -f, --force                 Force without confirmation
```

### SMS

```
wraps sms init            Deploy SMS infrastructure (AWS End User Messaging)
  -p, --provider          Hosting provider
  -r, --region            AWS region
  --preset                Config preset (starter, production, enterprise, custom)
  -y, --yes               Skip confirmation

wraps sms status          Show SMS infrastructure details
  --account               AWS account ID

wraps sms test            Send a test SMS
  --to                    Destination number (E.164 format)
  --message               Message content

wraps sms verify-number   Verify a destination phone number
  --phoneNumber           Number to verify (E.164)
  --code                  Verification code
  --list                  List verified numbers
  --delete                Delete a verified number
  --resend                Resend verification code

wraps sms upgrade         Upgrade SMS features
  -r, --region            AWS region
  -y, --yes               Skip confirmation

wraps sms register        Register toll-free number
  -r, --region            AWS region

wraps sms sync            Sync infrastructure
  -r, --region            AWS region
  -y, --yes               Skip confirmation

wraps sms destroy         Remove SMS infrastructure

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wraps-team/wraps](https://github.com/wraps-team/wraps) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-30 -->
