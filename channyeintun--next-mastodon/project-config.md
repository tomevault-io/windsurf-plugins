---
trigger: always_on
description: A performant social network frontend for Mastodon built with Next.js 16 and modern React patterns.
---

# Project: Mastodon (Next.js)

A performant social network frontend for Mastodon built with Next.js 16 and modern React patterns.

## Project Structure
```
next-mastodon/
├── public/                     # Static assets
│   ├── fonts/
│   │   └── silkscreen-regular.woff2  # Custom font for Wrapstodon
│   ├── icons/                  # PWA icons
│   │   ├── icon-192.png
│   │   ├── icon-512.png
│   │   └── icon-maskable.png
│   ├── images/                 # Static images
│   │   └── archetypes/         # Wrapstodon personality archetype images
│   │       ├── booster.png
│   │       ├── lurker.png
│   │       ├── oracle.png
│   │       ├── pollster.png
│   │       ├── replier.png
│   │       └── space_elements.png
│   ├── boop.mp3                # Notification sound (MP3)
│   ├── boop.ogg                # Notification sound (OGG)
│   ├── file.svg
│   ├── globe.svg
│   ├── next.svg
│   ├── sw.js                   # Service worker for push notifications
│   ├── vercel.svg
│   └── window.svg
├── example/                    # Example files and documentation
│   └── compose/
│       └── README.md          # Compose feature documentation
├── src/
│   ├── app/                   # Next.js App Router with route groups
│   │   ├── (main)/           # Main app layout route group
│   │   │   ├── [acct]/       # User profile pages (/@username or /@username@domain)
│   │   │   │   ├── followers/
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── following/
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── styles.ts  # Profile page styled components
│   │   │   │   └── page.tsx
│   │   │   ├── about/        # Instance information pages
│   │   │   │   ├── privacy/
│   │   │   │   │   └── page.tsx  # Privacy policy
│   │   │   │   ├── terms/
│   │   │   │   │   └── page.tsx  # Terms of service
│   │   │   │   └── page.tsx     # Server info, rules, admin contact
│   │   │   ├── bookmarks/    # Bookmarks page
│   │   │   │   └── page.tsx
│   │   │   ├── compose/      # Create post page
│   │   │   │   └── page.tsx
│   │   │   ├── conversations/  # Direct messages (conversations)
│   │   │   │   ├── [id]/
│   │   │   │   │   └── page.tsx  # Conversation thread (legacy, redirects to /chat)
│   │   │   │   ├── chat/
│   │   │   │   │   └── page.tsx  # Chat interface with message bubbles
│   │   │   │   ├── new/
│   │   │   │   │   └── page.tsx  # Create new DM/conversation
│   │   │   │   └── page.tsx     # Conversations list
│   │   │   ├── explore/      # Explore/discover page
│   │   │   │   └── page.tsx
│   │   │   ├── follow-requests/  # Follow requests management
│   │   │   │   └── page.tsx
│   │   │   ├── lists/        # Lists management
│   │   │   │   ├── [id]/
│   │   │   │   │   ├── members/
│   │   │   │   │   │   ├── MemberComponents.tsx  # List member UI components
│   │   │   │   │   │   └── page.tsx
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── ListComponents.tsx  # List UI components
│   │   │   │   └── page.tsx
│   │   │   ├── notifications/  # Notifications page
│   │   │   │   ├── requests/
│   │   │   │   │   └── page.tsx  # Filtered notification requests
│   │   │   │   ├── NotificationsV1.tsx  # V1 notifications implementation
│   │   │   │   ├── NotificationsV2.tsx  # V2 grouped notifications implementation
│   │   │   │   └── page.tsx
│   │   │   ├── profile/      # Current user profile
│   │   │   │   └── edit/
│   │   │   │       └── page.tsx
│   │   │   ├── scheduled/    # Scheduled posts
│   │   │   │   └── page.tsx
│   │   │   ├── search/       # Search page
│   │   │   │   └── page.tsx
│   │   │   ├── settings/     # Settings pages
│   │   │   │   ├── blocks/
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── filters/
│   │   │   │   │   ├── [id]/
│   │   │   │   │   │   └── page.tsx  # Edit filter
│   │   │   │   │   ├── new/
│   │   │   │   │   │   └── page.tsx  # Create new filter
│   │   │   │   │   └── page.tsx     # Filters list
│   │   │   │   ├── mutes/
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── notifications/
│   │   │   │   │   └── page.tsx  # Push notification settings
│   │   │   │   ├── preferences/  # User preferences settings
│   │   │   │   │   ├── SelectStyles.tsx  # Styled select components
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── SettingsClient.tsx  # Settings client component
│   │   │   │   └── page.tsx
│   │   │   ├── status/[id]/  # Status detail pages
│   │   │   │   ├── edit/
│   │   │   │   │   └── page.tsx
│   │   │   │   ├── favourited_by/
│   │   │   │   │   └── page.tsx  # Users who favourited this post
│   │   │   │   ├── reblogged_by/
│   │   │   │   │   └── page.tsx  # Users who boosted this post
│   │   │   │   ├── quotes/
│   │   │   │   │   └── page.tsx  # Quote posts of this status
│   │   │   │   └── page.tsx
│   │   │   ├── tags/[tag]/   # Hashtag feed pages
│   │   │   │   └── page.tsx
│   │   │   ├── layout.tsx    # Main layout wrapper
│   │   │   └── page.tsx      # Home page (timeline)
│   │   ├── (auth)/           # Auth layout route group
│   │   │   ├── auth/
│   │   │   │   ├── callback/  # OAuth callback handler
│   │   │   │   │   └── page.tsx
│   │   │   │   └── signin/    # Sign in page
│   │   │   │       └── page.tsx

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [channyeintun/next-mastodon](https://github.com/channyeintun/next-mastodon) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-25 -->
