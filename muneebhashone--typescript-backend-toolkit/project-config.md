---
trigger: always_on
description: Email system using React Email and Mailgun with queue-based sending
---


# Email System

## Architecture

- **Templates**: React Email components in [src/email/templates/](mdc:src/email/templates/)
- **Service**: Email service in [src/email/email.service.ts](mdc:src/email/email.service.ts)
- **Provider**: Email provider abstraction in [src/lib/email.ts](mdc:src/lib/email.ts) (supports Mailgun & SMTP)
- **Queue**: Background sending via [src/queues/email.queue.ts](mdc:src/queues/email.queue.ts)
- **Development**: Preview server for templates

## Email Configuration

### Environment Variables

```bash
# Option 1: Mailgun (Recommended)
MAILGUN_API_KEY=your-mailgun-api-key
MAILGUN_DOMAIN=your-domain.com
MAILGUN_FROM_EMAIL=noreply@your-domain.com

# Option 2: SMTP (Fallback)
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USERNAME=your-email@gmail.com
SMTP_PASSWORD=your-app-password
SMTP_FROM=noreply@your-domain.com
EMAIL_FROM=noreply@your-domain.com

# Note: Provider auto-selects Mailgun if configured, otherwise SMTP
```

## Creating Email Templates

### Step 1: Create React Component

Create new file in `src/email/templates/TemplateName.tsx`:

```typescript
import {
  Html,
  Head,
  Body,
  Container,
  Section,
  Text,
  Button,
  Hr,
  Img,
} from "@react-email/components";

interface TemplateNameProps {
  name: string;
  actionUrl: string;
}

export default function TemplateName({ name, actionUrl }: TemplateNameProps) {
  return (
    <Html>
      <Head />
      <Body style={styles.body}>
        <Container style={styles.container}>
          <Section style={styles.section}>
            <Img
              src="https://your-domain.com/logo.png"
              alt="Logo"
              width="150"
              height="50"
              style={styles.logo}
            />

            <Text style={styles.heading}>Hello, {name}!</Text>

            <Text style={styles.text}>
              Your email content goes here.
            </Text>

            <Button style={styles.button} href={actionUrl}>
              Click Here
            </Button>

            <Hr style={styles.hr} />

            <Text style={styles.footer}>
              © 2025 Your Company. All rights reserved.
            </Text>
          </Section>
        </Container>
      </Body>
    </Html>
  );
}

const styles = {
  body: {
    backgroundColor: "#f6f9fc",
    fontFamily: "-apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif",
  },
  container: {
    margin: "0 auto",
    padding: "20px 0",
  },
  section: {
    backgroundColor: "#ffffff",
    borderRadius: "8px",
    padding: "40px",
  },
  logo: {
    margin: "0 auto 20px",
    display: "block",
  },
  heading: {
    fontSize: "24px",
    fontWeight: "bold",
    margin: "20px 0",
    color: "#1a1a1a",
  },
  text: {
    fontSize: "16px",
    lineHeight: "24px",
    color: "#525252",
    margin: "16px 0",
  },
  button: {
    backgroundColor: "#007bff",
    color: "#ffffff",
    padding: "12px 32px",
    borderRadius: "6px",
    textDecoration: "none",
    display: "inline-block",
    margin: "20px 0",
  },
  hr: {
    borderColor: "#e6e6e6",
    margin: "30px 0",
  },
  footer: {
    fontSize: "14px",
    color: "#8c8c8c",
    textAlign: "center" as const,
  },
};

// Preview props for development
TemplateName.PreviewProps = {
  name: "John Doe",
  actionUrl: "https://example.com/action",
} as TemplateNameProps;
```

### Step 2: Test Template

```bash
# Start email development server
pnpm run email:dev

# Open browser to preview
# http://localhost:3001
```

## Sending Emails

### Method 1: Direct Send (Simple)

```typescript
import { sendEmail } from '@/email/email.service';

await sendEmail({
  to: 'user@example.com',
  subject: 'Welcome!',
  template: 'TemplateName',
  data: {
    name: 'John Doe',
    actionUrl: 'https://example.com/verify',
  },
});
```

### Method 2: Queue-based (Recommended)

```typescript
import { emailQueue } from '@/queues/email.queue';

await emailQueue.add('sendEmail', {
  to: 'user@example.com',
  subject: 'Welcome!',
  template: 'TemplateName',
  data: {
    name: 'John Doe',
    actionUrl: 'https://example.com/verify',
  },
});
```

## Email Service Usage

The email service in [email.service.ts](mdc:src/email/email.service.ts) handles:

- Template rendering
- HTML/text generation
- Queue job creation

### Function Signature

```typescript
interface SendEmailOptions {
  to: string | string[]; // Recipient(s)
  subject: string;
  template: string; // Template name (without .tsx)
  data: Record<string, any>; // Props for template
  from?: string; // Optional: override default sender
  replyTo?: string; // Optional: reply-to address
  attachments?: Array<{
    filename: string;
    content: Buffer | string;
    contentType?: string;
  }>;
}

export const sendEmail = async (options: SendEmailOptions): Promise<void>;
```

## Queue System

Email queue in [email.queue.ts](mdc:src/queues/email.queue.ts) provides:

- Async sending (doesn't block API response)
- Automatic retries on failure
- Queue monitoring via dashboard

### Queue Configuration

```typescript
// Default options
{
  attempts: 3, // Retry up to 3 times
  backoff: {
    type: "exponential",
    delay: 1000, // Start with 1 second delay
  },
}
```

### Custom Queue Options

```typescript
await emailQueue.add(
  'sendEmail',

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [muneebhashone/typescript-backend-toolkit](https://github.com/muneebhashone/typescript-backend-toolkit) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-01 -->
