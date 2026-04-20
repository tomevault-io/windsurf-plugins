---
trigger: always_on
description: Internationalization (i18n) patterns and conventions for whosee-whois
---


# Internationalization Patterns for Whosee WHOIS

## Configuration

### next-intl Setup
Configuration in [src/i18n/config.ts](mdc:src/i18n/config.ts):

```typescript
export const locales = ['en', 'zh'] as const;
export type Locale = (typeof locales)[number];

export const defaultLocale: Locale = 'en';

export const localeConfig = {
  en: {
    label: 'English',
    flag: '🇺🇸',
    dir: 'ltr'
  },
  zh: {
    label: '中文',
    flag: '🇨🇳', 
    dir: 'ltr'
  }
} as const;
```

### Middleware Configuration
[src/middleware.ts](mdc:src/middleware.ts) handles locale routing:

```typescript
import createMiddleware from 'next-intl/middleware';
import { locales, defaultLocale } from './src/i18n/config';

export default createMiddleware({
  locales,
  defaultLocale,
  localePrefix: 'as-needed'
});

export const config = {
  matcher: ['/((?!api|_next|_vercel|.*\\..*).*)']
};
```

## Translation Files

### File Structure
Translation files in [src/messages/](mdc:src/messages):
- **English**: [src/messages/en.json](mdc:src/messages/en.json)
- **Chinese**: [src/messages/zh.json](mdc:src/messages/zh.json)

### Translation Organization
Structure translations by feature/component:

```json
{
  "common": {
    "loading": "Loading...",
    "error": "An error occurred",
    "success": "Success",
    "copy": "Copy",
    "copied": "Copied!",
    "retry": "Retry",
    "cancel": "Cancel",
    "submit": "Submit"
  },
  "navigation": {
    "home": "Home",
    "domain": "Domain Lookup",
    "dns": "DNS Records",
    "health": "Health Check",
    "screenshot": "Screenshot",
    "language": "Language"
  },
  "domain": {
    "title": "Domain WHOIS Lookup",
    "description": "Get detailed information about any domain",
    "placeholder": "Enter domain name (e.g., example.com)",
    "search": "Search Domain",
    "results": {
      "title": "Domain Information",
      "registrar": "Registrar",
      "registrant": "Registrant",
      "created": "Created Date",
      "updated": "Updated Date",
      "expires": "Expiration Date",
      "status": "Status",
      "nameservers": "Name Servers",
      "contacts": "Contacts"
    },
    "errors": {
      "invalid": "Please enter a valid domain name",
      "notFound": "Domain not found",
      "timeout": "Request timed out"
    }
  },
  "dns": {
    "title": "DNS Records Lookup",
    "description": "Check DNS records for any domain",
    "types": {
      "A": "A Record (IPv4)",
      "AAAA": "AAAA Record (IPv6)",
      "MX": "MX Record (Mail)",
      "TXT": "TXT Record (Text)",
      "NS": "NS Record (Name Server)",
      "CNAME": "CNAME Record (Canonical)",
      "SOA": "SOA Record (Start of Authority)"
    },
    "table": {
      "type": "Type",
      "name": "Name", 
      "value": "Value",
      "ttl": "TTL",
      "priority": "Priority"
    }
  }
}
```

## Usage Patterns

### Basic Translation Hook
```typescript
import { useTranslations } from 'next-intl';

function Component() {
  const t = useTranslations('namespace');
  
  return (
    <div>
      <h1>{t('title')}</h1>
      <p>{t('description')}</p>
    </div>
  );
}
```

### Multiple Namespaces
```typescript
function ComplexComponent() {
  const tCommon = useTranslations('common');
  const tDomain = useTranslations('domain');
  const tNav = useTranslations('navigation');
  
  return (
    <div>
      <h1>{tDomain('title')}</h1>
      <button>{tCommon('submit')}</button>
      <nav>{tNav('home')}</nav>
    </div>
  );
}
```

### Translation with Variables
```typescript
// Translation file
{
  "welcome": "Welcome, {name}!",
  "found": "Found {count} {count, plural, one {result} other {results}}"
}

// Component usage
function Component({ username, resultCount }: Props) {
  const t = useTranslations('messages');
  
  return (
    <div>
      <h1>{t('welcome', { name: username })}</h1>
      <p>{t('found', { count: resultCount })}</p>
    </div>
  );
}
```

### Rich Text Translations
```typescript
// Translation file
{
  "richText": "Visit our <link>documentation</link> for more info"
}

// Component usage
function Component() {
  const t = useTranslations('messages');
  
  return (
    <p>
      {t.rich('richText', {
        link: (chunks) => <a href="/docs">{chunks}</a>
      })}
    </p>
  );
}
```

## Server Components

### Server-side Translations
```typescript
import { getTranslations } from 'next-intl/server';

export default async function ServerComponent() {
  const t = await getTranslations('namespace');
  
  return (
    <div>
      <h1>{t('title')}</h1>
      <p>{t('description')}</p>
    </div>
  );
}
```

### Metadata Translations
```typescript
import { getTranslations } from 'next-intl/server';

export async function generateMetadata({ params: { locale } }) {
  const t = await getTranslations({ locale, namespace: 'metadata' });
  
  return {
    title: t('title'),
    description: t('description')
  };
}
```

## Locale Management

### Language Toggle Component
Pattern from [src/components/ui/language-toggle.tsx](mdc:src/components/ui/language-toggle.tsx):

```typescript
import { useLocale } from 'next-intl';
import { useRouter, usePathname } from 'next/navigation';
import { locales, localeConfig } from '@/i18n/config';

export function LanguageToggle() {
  const locale = useLocale();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/AsisYu) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-10 -->
