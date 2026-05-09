---
trigger: always_on
description: These rules guide development of email templates using React Email components. React Email components are designed to work across all major email clients (Gmail, Apple Mail, Outlook, Yahoo Mail, HEY, Superhuman).
---


# React Email Cursor Rules

## Overview
These rules guide development of email templates using React Email components. React Email components are designed to work across all major email clients (Gmail, Apple Mail, Outlook, Yahoo Mail, HEY, Superhuman).

## Core Structure

### Email Template Structure
Every email template should follow this structure:

```tsx
import {
  Html,
  Head,
  Body,
  Preview,
  Container,
  // ... other components
} from '@react-email/components';

interface EmailProps {
  // Define props with optional defaults
  propName?: string;
}

export const EmailTemplate = ({ propName }: EmailProps) => (
  <Html lang="en" dir="ltr">
    <Head />
    <Preview>Preview text shown in email client inbox</Preview>
    <Body style={main}>
      <Container style={container}>
        {/* Email content */}
      </Container>
    </Body>
  </Html>
);

// Always export PreviewProps for development
EmailTemplate.PreviewProps = {
  propName: 'default value',
} as EmailProps;

export default EmailTemplate;
```

## Component Guidelines

### Html Component
- **Always** wrap email content in `<Html>` component
- Use `lang="en"` and `dir="ltr"` attributes
- Required as root wrapper

```tsx
<Html lang="en" dir="ltr">
  {/* content */}
</Html>
```

### Head Component
- Place inside `<Html>` but before `<Body>`
- Use for metadata, fonts, and styles
- Can be self-closing: `<Head />`

```tsx
<Html>
  <Head>
    <title>Email Title</title>
    {/* Font components, styles */}
  </Head>
  {/* Body content */}
</Html>
```

### Preview Component
- Place immediately after `<Head>`
- Provides preview text shown in email client inbox
- Keep concise (50-100 characters recommended)
- Use dynamic content when appropriate

```tsx
<Preview>Log in with this magic link</Preview>
// or
<Preview>{`Join ${username} on ${platform}`}</Preview>
```

### Body Component
- Required wrapper for email content
- Use inline styles via `style` prop
- Set background color and font family here

```tsx
<Body style={main}>
  {/* content */}
</Body>

const main = {
  backgroundColor: '#ffffff',
  fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif',
};
```

### Container Component
- Centers content horizontally
- Use for main content wrapper
- Set max-width and padding via styles

```tsx
<Container style={container}>
  {/* content */}
</Container>

const container = {
  backgroundColor: '#ffffff',
  margin: '0 auto',
  padding: '20px 0 48px',
  maxWidth: '600px', // Optional, for fixed width
};
```

### Section Component
- Use for grouping related content
- Provides padding and layout control
- Can nest within Container

```tsx
<Section style={box}>
  {/* grouped content */}
</Section>

const box = {
  padding: '0 48px',
};
```

### Text Component
- Use instead of `<p>` tags for better email client compatibility
- Always use inline styles
- Set color, fontSize, lineHeight, fontFamily

```tsx
<Text style={paragraph}>
  Your text content here
</Text>

const paragraph = {
  color: '#333',
  fontSize: '16px',
  lineHeight: '24px',
  fontFamily: '-apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, sans-serif',
  margin: '0',
};
```

### Heading Component
- Use semantic headings (h1-h6)
- Specify `as` prop: `as="h1"`, `as="h2"`, etc.
- Style with inline styles

```tsx
<Heading as="h1" style={h1}>
  Welcome
</Heading>

const h1 = {
  color: '#333',
  fontSize: '24px',
  fontWeight: 'bold',
  margin: '40px 0',
  padding: '0',
};
```

### Button Component
- **Important**: This is actually a styled `<a>` tag, not a real button
- Requires `href` prop (required)
- Use `target="_blank"` for external links (default)
- Style with inline styles, including padding, backgroundColor, borderRadius

```tsx
<Button
  href="https://example.com"
  style={button}
>
  Click me
</Button>

const button = {
  backgroundColor: '#656ee8',
  borderRadius: '5px',
  color: '#fff',
  fontSize: '16px',
  fontWeight: 'bold',
  textDecoration: 'none',
  textAlign: 'center' as const,
  display: 'block',
  width: '100%',
  padding: '10px',
};
```

### Link Component
- Use for inline links within text
- Always include `target="_blank"` for external links
- Style with inline styles

```tsx
<Text>
  Visit our{' '}
  <Link href="https://example.com" style={anchor}>
    website
  </Link>
  {' '}for more info.
</Text>

const anchor = {
  color: '#556cd6',
  textDecoration: 'underline',
};
```

### Image Component (Img)
- **Always** specify `width` and `height` attributes (not just styles)
- Use `alt` text for accessibility
- Use absolute URLs (not relative paths)
- Consider using `baseUrl` pattern for development

```tsx
const baseUrl = process.env.VERCEL_URL
  ? `https://${process.env.VERCEL_URL}`
  : '';

<Img
  src={`${baseUrl}/static/logo.png`}
  width="49"
  height="21"
  alt="Company Logo"
  style={logo}
/>

const logo = {
  margin: '0 auto',
};
```

### Hr Component
- Use for horizontal dividers
- Style border color and margin

```tsx
<Hr style={hr} />

const hr = {
  borderColor: '#e6ebf1',
  margin: '20px 0',
};
```

### Row and Column Components

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [inboundemail/react-email-templates](https://github.com/inboundemail/react-email-templates) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
