---
trigger: always_on
description: This describes the design pattern and implementation guidance for implementing nextjs page.
---

---
description: This describes the design pattern and implementation guidance for implementing nextjs page.
globs: *.tsx, /app/[locale]
---

# NextJS Frontend Page Implementation Patterns

This document describes the patterns for implementing different types of pages in a NextJS application with authentication, localization, and server communication.

## Common Setup

- Always read [frontend.md](mdc:docs/frontend.md) for instructions on the styles, [brand-info.ts](mdc:lib/brand-info.ts) for constructing text and [architecture.md](mdc:docs/architecture.md) for overall architecture

### Directory Structure
app/
  [locale]/
    page.tsx                 # Public page
    about/
      page.tsx              # Public page
    account/
      settings/
        page.tsx           # Account page (authenticated)
      security/
        page.tsx          # Account page (authenticated)
    admin/
      contacts/
        page.tsx         # Admin page (authenticated + admin)
      analytics/
        page.tsx        # Admin page (authenticated + admin)
      AdminLayoutClient.tsx  # Admin layout wrapper
    layout.tsx  # Root layout with locale provider
    page.tsx    # Home page
  i18n/
    server-utils.ts  # Server utilities for localization
    client-utils.ts  # Client utilities for localization
  messages/
    en.json     # English translations
    fi.json     # Finnish translations
    sv.json     # Swedish translations

### Required Dependencies
```json
{
  "dependencies": {
    "next": "^14.0.0",
    "next-intl": "^3.0.0",
    "@supabase/supabase-js": "^2.0.0"
  }
}
```

### Localisation Configuration

1. **Root Layout Setup** (`app/[locale]/layout.tsx`):
```typescript
import { NextIntlClientProvider } from 'next-intl'
import { getMessages } from '@/app/i18n/server-utils'

interface Props {
  children: React.ReactNode
  params: { locale: string }
}

export default async function LocaleLayout({ children, params: { locale } }: Props) {
  const messages = await getMessages(locale)

  return (
    <NextIntlClientProvider locale={locale} messages={messages}>
      {children}
    </NextIntlClientProvider>
  )
}
```

2. **Server Utilities** (`app/i18n/server-utils.ts`):
```typescript
import { createSharedPathnamesNavigation } from 'next-intl/navigation'
import { getRequestConfig } from 'next-intl/server'

export const locales = ['en', 'fi', 'sv']
export const defaultLocale = 'en'

// For server components
export async function getMessages(locale: string) {
  return (await import(`@/messages/${locale}.json`)).default
}

// Setup locale for server components
export async function setupServerLocale(locale: string) {
  const messages = await getMessages(locale)
  return { locale, messages }
}

// Navigation utilities
export const { Link, redirect, usePathname, useRouter } = 
  createSharedPathnamesNavigation({ locales })
```

3. **Client Utilities** (`app/i18n/client-utils.ts`):
```typescript
'use client'

import { useTranslations } from 'next-intl'
import { createSharedPathnamesNavigation } from 'next-intl/navigation'

export const locales = ['en', 'fi', 'sv']

// Navigation utilities for client components
export const { Link, usePathname, useRouter } = 
  createSharedPathnamesNavigation({ locales })

// Hook for switching locales
export function useLocale() {
  const router = useRouter()
  const pathname = usePathname()
  
  const switchLocale = (newLocale: string) => {
    router.replace(pathname, { locale: newLocale })
  }

  return { switchLocale }
}
```
### Usage Examples

1. **Server Components**:
```typescript
import { getTranslations } from 'next-intl/server'

export default async function Page() {
  const t = await getTranslations('Namespace')
  return <h1>{t('title')}</h1>
}
```

2. **Client Components**:
```typescript
'use client'

import { useTranslations } from 'next-intl'

export default function Component() {
  const t = useTranslations('Namespace')
  return <button>{t('buttons.submit')}</button>
}
```

3. **Locale Switcher Component**:
```typescript
'use client'

import { useLocale, locales } from '@/app/i18n/client-utils'

export default function LocaleSwitcher() {
  const { switchLocale } = useLocale()
  
  return (
    <select onChange={(e) => switchLocale(e.target.value)}>
      {locales.map((locale) => (
        <option key={locale} value={locale}>
          {locale.toUpperCase()}
        </option>
      ))}
    </select>
  )
}
```

4. **Localized Links**:
```typescript
import { Link } from '@/app/i18n/client-utils'

export default function Navigation() {
  return (
    <nav>
      <Link href="/">Home</Link>
      <Link href="/about">About</Link>
    </nav>
  )
}
```

### Best Practices

1. **Translation Keys**:
- Use nested structures for better organization
- Use consistent naming conventions
- Keep keys descriptive and hierarchical
- Group common translations under shared namespaces

2. **Dynamic Values**:
```typescript
// Using variables
t('welcome', { name: user.name })

// Using plurals
t('items', { count: items.length })

// Using rich text
t.rich('terms', {
  link: (chunks) => <a href="/terms">{chunks}</a>
})
```

3. **Date and Number Formatting**:
```typescript
import { useFormatter } from 'next-intl'

export default function Component() {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [LastBotInc/nextjs-supabase-ai-webapp](https://github.com/LastBotInc/nextjs-supabase-ai-webapp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-20 -->
