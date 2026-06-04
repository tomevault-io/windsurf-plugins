---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Venture Planner 3 is a Django-based business consulting website with an integrated blog platform. The site showcases consulting services, hosts educational blog content, and generates leads through contact forms.

## Technology Stack

- **Framework**: Django 4.2.18
- **Database**: PostgreSQL
- **File Storage**: AWS S3 (for media files)
- **CAPTCHA**: Cloudflare Turnstile
- **Python Version**: 3.x (check your local environment)

## Environment Setup

### Required Environment Variables (.env)

The project requires a `.env` file in the root directory with:

```
SECRET_KEY=<django-secret-key>
DEBUG=True

# Database
DB_NAME=<database-name>
DB_USER=<database-user>
DB_PASSWORD=<database-password>
DB_HOST=localhost
DB_PORT=5432

# AWS S3
AWS_ACCESS_KEY_ID=<aws-access-key>
AWS_SECRET_ACCESS_KEY=<aws-secret-key>
AWS_STORAGE_BUCKET_NAME=<s3-bucket-name>
AWS_S3_REGION_NAME=<aws-region>

# Cloudflare Turnstile
TURNSTILE_SITE_KEY=<site-key>
TURNSTILE_SECRET_KEY=<secret-key>

# Email (add these if not present)
DEFAULT_FROM_EMAIL=<sender-email>
CONTACT_US_EMAIL=<recipient-email>
```

### Initial Setup

```bash
# Install dependencies
pip install -r requirements.txt  # Create if needed from pip freeze

# Run migrations
python manage.py migrate

# Create superuser for admin access
python manage.py createsuperuser

# Run development server
python manage.py runserver
```

## Common Commands

### Development Server
```bash
python manage.py runserver
```

### Database Operations
```bash
# Create migrations after model changes
python manage.py makemigrations

# Apply migrations
python manage.py migrate

# Access database shell
python manage.py dbshell
```

### Static Files
```bash
# Collect static files for production
python manage.py collectstatic
```

### Admin & Shell
```bash
# Django shell
python manage.py shell

# Create admin user
python manage.py createsuperuser
```

## Architecture

### Static vs Media File Separation

**IMPORTANT**: The project has a specific architecture for file handling:

- **Static files** (CSS, JS, fonts): Served locally from `/static/` directory during development
- **Media files** (blog images, user uploads): Served from AWS S3 via `django-storages`

This is configured in `settings.py`:
```python
STATIC_URL = '/static/'
STATICFILES_DIRS = [BASE_DIR / "static"]

MEDIA_URL = f"https://{AWS_STORAGE_BUCKET_NAME}.s3.amazonaws.com/"
DEFAULT_FILE_STORAGE = 'storages.backends.s3boto3.S3Boto3Storage'
```

### Template Structure

Templates use a component-based architecture with reusable partials:

- **Base templates**: `base.html`, `base2.html` - Define overall page structure
- **Page templates**: `pages/*.html` - Individual page content
- **Partials**: `partials/sitewide/*.html` and `partials/*-partial.html` - Reusable components

Components include: header, footer, preloader, hero sections, services, testimonials, pricing, blog listings.

### Context Processors

Two custom context processors inject data into all templates:

1. **`breadcrumbs`** (`pages/context_processors.py:4-57`): Generates page titles and breadcrumb navigation based on URL routing
2. **`turnstile_keys`** (`pages/context_processors.py:59-65`): Injects Cloudflare Turnstile site key globally

These are registered in `settings.py` TEMPLATES configuration.

### Blog System Architecture

The blog uses a relational model structure:

- **Blog** - Main blog post model with many-to-many relationships and SEO fields
- **BlogCategory** - Categorization (many-to-many with Blog)
- **BlogTag** - Tagging system (many-to-many with Blog)
- **BlogAuthor** - Author profiles (foreign key from Blog)
- **BlogComment** - Comment system with approval workflow

**Key behaviors**:
- Blog posts have a `draft` flag (only `draft=False` posts shown to users)
- Blog pagination shows 8 posts per page with SEO-friendly prev/next tags
- Blog images stored in S3 at `static/blog/images/` and `static/blog/thumbnails/`
- **Blog posts use SEO-friendly slug URLs**: `/blog/slug-goes-here/` (auto-generated from title)
- Slugs are automatically generated on save if not provided
- **SEO Fields**: `meta_description` (160 chars max), `meta_keywords`, and `slug` are available per post

### URL Routing

All page routes are defined in `pages/urls.py` (26 total routes). The project uses named URLs for internal linking and breadcrumb generation.

### Forms & CAPTCHA

Contact forms use Django Forms with Bootstrap styling:
- `ContactForm` in `pages/forms.py` handles contact page submissions
- Support submissions handled by `support_submit` view with Cloudflare Turnstile verification
- Email notifications sent via `django.core.mail.send_mail`

### Database Schema

Custom table names are defined via Meta classes:
- `blog_blog` - Blog posts
- `blog_blogcategory` - Categories
- `blog_blogtag` - Tags
- `blog_blogauthor` - Authors
- `blog_blogcomment` - Comments

This prevents Django's default `pages_*` naming convention.

### Email Configuration

The project sends emails for:
1. Contact form submissions (`contact` view)
2. Support requests (`support_submit` view)


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [DM-Software-Ltd/ventureplanner-website-v3](https://github.com/DM-Software-Ltd/ventureplanner-website-v3) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-04 -->
