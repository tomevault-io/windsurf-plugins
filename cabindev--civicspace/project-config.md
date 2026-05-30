---
trigger: always_on
description: CivicSpace : พื้นที่พลเมืองร่วมหาทางออกปัญหาแอลกอฮอล์ เป็นศูนย์ข้อมูลและพื้นที่สำหรับเจ้าหน้าที่ในการทำงานร่วมกันหาทางออกปัญหาแอลกอฮอล์ รวบรวมข้อมูล บทความ และโครงการต่างๆ โดยใช้ API จาก CivicSpace API สำหรับแสดงเนื้อหา
---

# CivicSpace Project - Claude Context

## Project Overview
CivicSpace : พื้นที่พลเมืองร่วมหาทางออกปัญหาแอลกอฮอล์ เป็นศูนย์ข้อมูลและพื้นที่สำหรับเจ้าหน้าที่ในการทำงานร่วมกันหาทางออกปัญหาแอลกอฮอล์ รวบรวมข้อมูล บทความ และโครงการต่างๆ โดยใช้ API จาก CivicSpace API สำหรับแสดงเนื้อหา

## Technical Stack
- **Frontend**: Next.js 14 with React 18, TypeScript
- **UI Libraries**: Tailwind CSS, Lucide React Icons
- **Database**: MySQL with Prisma ORM (auth only)
- **Authentication**: NextAuth.js with Prisma adapter
- **External Data**: CivicSpace API integration
- **Styling**: Custom CSS with yellow color scheme
- **Design**: Clean, minimal design focused on small fonts and readability

## Project Structure
```
app/
├── api/                    # API routes (proxy to external API)
│   ├── auth/              # Authentication endpoints
│   ├── post/              # Posts proxy API with pagination
│   ├── categories/        # Categories proxy API
│   ├── videos/            # Videos proxy API with pagination
│   └── surveys/           # Surveys proxy API with pagination
├── dashboard/             # Admin dashboard for staff
│   ├── page.tsx           # Dashboard with API statistics
│   ├── posts/             # Posts management
│   ├── categories/        # Categories management
│   └── surveys/           # Surveys management
├── post/                  # All posts page with pagination
│   └── page.tsx           # Display all posts (24 per page)
├── videos/                # All videos page with pagination
│   └── page.tsx           # Display all videos (24 per page)
├── components/            # Reusable components
│   ├── Navbar.tsx         # Navigation bar
│   ├── Footer.tsx         # Footer component
│   ├── Loading.tsx        # Loading states
│   └── SurveyCard.tsx     # Survey card component
├── auth/                  # Authentication pages
├── page.tsx               # Homepage with latest content
├── post/[slug]/           # Individual post detail pages
├── layout.tsx             # Root layout
└── globals.css            # Global styles with yellow theme

lib/
└── api.ts                 # API library with interfaces

prisma/
├── schema.prisma          # Minimal schema (User + Role only)
└── migrations/            # Database migrations
```

## Key Features
1. **หน้าแรก** - แสดงบทความล่าสุด 12 รายการ, วิดีโอล่าสุด 8 รายการ, และแบบสำรวจล่าสุด 3 รายการ
2. **หน้าบทความทั้งหมด** (/post) - แสดงบทความทั้งหมดแบบ Masonry grid พร้อม pagination (24 รายการต่อหน้า)
3. **หน้าวิดีโอทั้งหมด** (/videos) - แสดงวิดีโอทั้งหมดแบบ grid พร้อม pagination (24 รายการต่อหน้า)
4. **แดชบอร์ด** - สถิติและข้อมูลภาพรวมสำหรับเจ้าหน้าที่ (ไม่มี mock data)
5. **ระบบผู้ใช้** - การจัดการผู้ใช้และสิทธิ์ (NextAuth.js)
6. **API Integration** - เชื่อมต่อกับ CivicSpace API ผ่าน proxy routes (ไม่มี mock data)
7. **Surveys Management** - จัดการและดาวน์โหลดแบบสำรวจ
8. **Clean Design** - ดีไซน์เรียบง่าย เน้นตัวหนังสือขนาดเล็ก สีเหลือง
9. **เฉพาะเจ้าหน้าที่** - ระบบสำหรับเจ้าหน้าที่เท่านั้น

## Database Configuration
- **Development**: `mysql://root:root@localhost:3306/civicspace`
- **Authentication**: NextAuth with custom secret

## Development Commands
- `npm run dev` - Start development server
- `npm run build` - Build for production
- `npm run start` - Start production server
- `npm run lint` - Run ESLint

## External API Integration

### CivicSpace API
Base URL: `https://civicspace-gqdcg0dxgjbqe8as.southeastasia-01.azurewebsites.net/api/v1`

**Posts Endpoints:**
- `GET /posts/` - All posts with pagination
- `GET /posts/latest/?limit=N` - Latest posts
- `GET /posts/popular/?limit=N` - Popular posts by view count
- `GET /posts/{slug}/` - Single post details
- `GET /categories/{slug}/posts/` - Posts by category

**Categories Endpoints:**
- `GET /categories/` - All categories with post counts
- `GET /categories/{slug}/` - Category details

**Videos Endpoints:**
- `GET /videos/?page=X&page_size=N` - All videos with pagination (returns {count, next, previous, results})
- `GET /videos/latest/?limit=N` - Latest videos (returns array)

**Surveys Endpoints (NEW):**
- `GET /surveys/` - All surveys with pagination
- `GET /surveys/latest/?limit=N` - Latest surveys
- `GET /surveys/popular/?limit=N` - Popular surveys by view count
- `GET /surveys/{slug}/` - Single survey details (increments view_count)
- `GET /categories/{slug}/surveys/` - Surveys by category

**Survey Response Structure:**
```json
{
  "id": 1,
  "title": "แบบสำรวจ...",
  "slug": "survey-slug",
  "description": "คำอธิบาย",
  "author": "email@example.com",
  "category": {
    "id": 10,
    "name": "หมวดหมู่",
    "slug": "category-slug"
  },
  "survey_file_url": "https://.../file.docx",
  "is_published": true,
  "survey_date": "2025-10-08",
  "response_count": 0,
  "view_count": 0,
  "created_at": "2025-10-08T11:44:48+07:00",
  "updated_at": "2025-10-08T11:44:48+07:00",
  "published_at": "2025-10-08T11:44:48+07:00"
}
```

## Color Scheme
Yellow-based theme with careful attention to readability:
```css
--primary: #f59e0b;        /* Main yellow */
--secondary: #92400e;      /* Dark brown */
--accent: #fbbf24;         /* Light yellow */

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [cabindev/civicspace](https://github.com/cabindev/civicspace) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-30 -->
