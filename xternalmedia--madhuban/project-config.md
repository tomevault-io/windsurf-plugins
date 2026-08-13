---
trigger: always_on
description: These instructions apply to the entire repository rooted at this directory.
---

# Madhuban Garden Resort Website

## Scope

These instructions apply to the entire repository rooted at this directory.

## Project Overview

- Build a full-stack resort website for **Madhuban Garden Resort** in **Agar Malwa District, Madhya Pradesh, India**.
- Domain: `madhubangarden.com`
- This is a client project. Build clean, production-grade code. No shortcuts.

## Current Build Phase

- **Phase 1: Frontend with dummy data only**
- Build all pages with static or dummy data first.
- Do not connect the real database yet.
- Do not integrate real payments yet.
- Goal: get the full UI approved by the client before backend work begins.

## UI Design Reference

- All screen designs, layouts, and visual references are inside the `/UI design/` folder at the project root.
- Before building any page or component, open and read the relevant file in `/UI design/` first.
- Match the design as closely as possible.
- Do not invent layouts that are not in the design files.

## Required Tech Stack

Do not deviate from this stack:

| Layer         | Tool                       |
| ------------- | -------------------------- |
| Framework     | Next.js 14 (App Router)    |
| Styling       | Tailwind CSS               |
| UI Components | shadcn/ui                  |
| Animations    | Framer Motion              |
| Icons         | Lucide React               |
| Database      | Supabase (PostgreSQL)      |
| ORM           | Prisma                     |
| Validation    | Zod                        |
| Auth          | Supabase Auth              |
| Payments      | Razorpay                   |
| Email         | Resend                     |
| Caching       | Upstash Redis              |
| iCal Sync     | node-ical + ical-generator |
| Hosting       | Vercel                     |

## Design System

### Theme

- Style: Crayons Light Green
- Feel: light, fresh, natural, premium, peaceful, lush, resort-like
- Do not make the site feel corporate or generic.
- Use an elegant serif for headings and a clean sans-serif for body text.
- Do not use Inter, Roboto, or Arial.

### Colors

- Primary: `#4CAF50`
- Background: `#f5f9f0`
- Accent: `#2e7d32`
- Text: `#1a1a1a`
- White: `#ffffff`

### Design Rules

- Mobile-first and fully responsive
- Smooth Framer Motion page transitions
- Use Next.js `Image` for optimized images
- No purple gradients
- No generic AI aesthetics
- Layouts should feel light, airy, and nature-inspired

## Tagline

Use this exact tagline across the site, including hero section, meta description, and OG tags:

> "The most peaceful & lush green premises in Agar Malwa District."

## Pages To Build

| Route           | Page                        | Priority |
| --------------- | --------------------------- | -------- |
| `/`             | Homepage                    | High     |
| `/rooms`        | All Rooms (19)              | High     |
| `/rooms/[slug]` | Individual Room             | High     |
| `/wedding`      | Wedding Venue               | High     |
| `/contact`      | Contact + Query Form        | High     |
| `/banquet`      | Banquet Hall                | Medium   |
| `/restaurant`   | Indoor + Outdoor Restaurant | Medium   |
| `/pool`         | Swimming Pool               | Medium   |
| `/events`       | Small Events + Parties      | Medium   |
| `/gallery`      | Image Gallery               | Medium   |
| `/attractions`  | Nearby Attractions          | Low      |
| `/admin`        | Admin Dashboard (protected) | Low      |

## Homepage Sections

Build homepage sections in this order:

1. Hero: full-screen, tagline, Book Now CTA, background resort image
2. Quick Highlights: Rooms, Banquet, Pool, Restaurant icon cards
3. Wedding Feature: big bold section with "Make your wedding unforgettable"
4. Rooms Preview: 3 featured rooms with Book Now
5. Core Services: Hotel, Banquet, Restaurant, Pool, Events
6. Amenities Strip: Parking, WiFi, Laundry, In-Room Dining, Breakfast
7. Instagram Feed: Behold.so embed
8. Google Reviews: Google Places API source later
9. Nearby Attractions: Maa Baglabukhi Temple, Mahakaleshwar Temple
10. Footer: links, contact, WhatsApp button, social links

## Core Services

- Hotel Rooms: 19 rooms, likely increasing later
- Banquet Hall: 1 hall, likely increasing later
- Indoor Restaurant
- Outdoor Restaurant
- Swimming Pool: 1

## Addons And Events

- Birthday Parties
- Small Parties
- Catering
- Event Management
- Decoration
- Corporate Meets & Conferences

## Amenities

- Indoor Parking
- Free WiFi
- Laundry Service
- In-Room Dining
- Complimentary Breakfast

## Major Selling Point

- Wedding Venue is the number one revenue driver.
- The `/wedding` page must be the most polished and detailed page on the site.
- Feature the wedding offering prominently on the homepage.
- Use emotional language appropriately.
- Show venue photos, capacity, and packages in the final implementation.

## Booking System

Planned booking flow:

1. User selects room plus check-in and check-out dates
2. System checks availability via `/api/availability` and Supabase later
3. User fills guest details: name, phone, email
4. User chooses payment method:
   - Pay Now: Razorpay checkout, webhook confirms, booking saved
   - Pay at Reception: booking saved with `pending` status

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [xternalmedia/Madhuban](https://github.com/xternalmedia/Madhuban) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-08-12 -->
