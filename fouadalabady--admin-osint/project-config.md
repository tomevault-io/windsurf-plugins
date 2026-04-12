---
trigger: always_on
description: This document serves as the comprehensive API reference for the OSINT Blog headless API system. It's designed to be used by:  - Frontend developers integrating with the blog API - Backend developers maintaining or extending the API - Third-party integrators building external applications - QA engineers testing API functionality - Technical writers creating derived documentation  The API documentation should be consulted when: - Implementing client applications that consume the blog API - Develop
---

# OSINT Blog Headless API Documentation

## Overview
This documentation describes the headless blog API system built with Next.js and Supabase. The system provides a complete backend for managing a multilingual blog with support for posts, categories, tags, media, and user management.

## Implementation Status Key
- ✅ **Implemented** - Feature is fully implemented and available
- 🟡 **Partially Implemented** - Basic functionality is available but may need refinement
- 🔄 **In Progress** - Feature is currently being implemented
- 📅 **Planned** - Feature is planned but not yet implemented

## Base URL
```
Production: https://blogosint-c5temew7u-osint2.vercel.app/api
Development: http://localhost:3000/api
```

## Authentication ✅
The API uses JWT tokens for authentication. Include the token in the Authorization header:
```
Authorization: Bearer <your_jwt_token>
```

## GraphQL Integration ✅
The API includes GraphQL support for more flexible querying:
```
GraphQL Endpoint: /api/graphql
```

Example GraphQL query:
```graphql
query GetBlogPosts($limit: Int, $offset: Int, $where: blog_posts_bool_exp, $order_by: [blog_posts_order_by!]) {
  blog_posts(limit: $limit, offset: $offset, where: $where, order_by: $order_by) {
    id
    title
    slug
    excerpt
    featured_image
    created_at
    updated_at
    author_id
  }
  blog_posts_aggregate {
    aggregate {
      count
    }
  }
}
```

## Database Schema

### Tables Structure
1. **blog_posts** ✅
   - Primary blog content storage
   - Supports multilingual content (RTL/LTR)
   - Includes SEO metadata
   - Tracks view counts and engagement

2. **blog_categories** 🟡
   - Hierarchical content organization
   - Supports multilingual categories

3. **blog_tags** 🟡
   - Content classification and search optimization
   - Many-to-many relationship with posts

4. **blog_media** ✅
   - Media asset management
   - Supports images, videos, and documents
   - Includes metadata and alt text

5. **blog_comments** 📅
   - User engagement tracking
   - Moderation support with approval workflow
   - Threaded discussions

6. **users** ✅
   - Author and administrator management
   - Role-based access control

## API Endpoints

### Posts

#### List Posts ✅
```http
GET /api/posts
```

Query Parameters:
- `page` (integer): Page number for pagination
- `limit` (integer): Items per page (default: 10)
- `status` (string): Filter by status ('draft', 'published')
- `category` (uuid): Filter by category ID
- `tag` (uuid): Filter by tag ID
- `author` (uuid): Filter by author ID
- `direction` (string): Filter by content direction ('ltr', 'rtl')
- `featured` (boolean): Filter featured posts
- `search` (string): Search in title and content 🔄

Response:
```json
{
  "data": [{
    "id": "uuid",
    "title": "string",
    "slug": "string",
    "excerpt": "string?",
    "content": "string?",
    "featured_image": "string?",
    "author": {
      "id": "uuid",
      "name": "string"
    },
    "category": {
      "id": "uuid",
      "name": "string"
    },
    "tags": [{
      "id": "uuid",
      "name": "string"
    }],
    "status": "string",
    "is_featured": boolean,
    "view_count": number,
    "published_at": "datetime?",
    "created_at": "datetime",
    "updated_at": "datetime",
    "seo_title": "string?",
    "seo_description": "string?",
    "seo_keywords": "string?",
    "direction": "string"
  }],
  "metadata": {
    "total": number,
    "page": number,
    "limit": number,
    "total_pages": number
  }
}
```

#### Get Single Post ✅
```http
GET /api/posts/{slug}
```

Response: Single post object with full details

#### Create Post ✅
```http
POST /api/posts
```

Request Body:
```json
{
  "title": "string",
  "content": "string",
  "excerpt": "string?",
  "featured_image": "string?",
  "category_id": "uuid?",
  "tag_ids": "uuid[]?",
  "status": "draft|published",
  "is_featured": "boolean?",
  "published_at": "datetime?",
  "seo_title": "string?",
  "seo_description": "string?",
  "seo_keywords": "string?",
  "direction": "ltr|rtl"
}
```

#### Update Post ✅
```http
PUT /api/posts/{id}
```

Request Body: Same as Create Post

#### Delete Post ✅
```http
DELETE /api/posts/{id}
```

### Categories

#### List Categories 🟡
```http
GET /api/categories
```

GraphQL equivalent (✅ Implemented):
```graphql
query GET_CATEGORIES {
  blog_categories {
    id
    name
    slug
    description
  }
}
```

#### Get Posts by Category 🟡
```http
GET /api/categories/{slug}/posts
```

GraphQL equivalent (✅ Implemented):
```graphql
query GET_POSTS_BY_CATEGORY($categoryId: uuid!, $limit: Int, $offset: Int) {
  blog_posts(where: {category_id: {_eq: $categoryId}}, limit: $limit, offset: $offset) {
    id
    title
    slug
    excerpt
    # other fields
  }
}
```

#### Create Category 🟡
```http
POST /api/categories
```

Request Body:
```json
{
  "name": "string",

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/fouadalabady) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
