---
trigger: always_on
description: WordPress Performance Community
---

# WordPress Performance Best Practices

**Version 1.0.0**
WordPress Performance Community
January 2026

> **Note:** This document is designed for AI agents and LLMs assisting with WordPress development. It provides structured performance guidelines with clear examples of incorrect and correct implementations.

## Abstract

Comprehensive performance optimization guide for WordPress development, designed for AI agents and LLMs assisting developers. This document covers database optimization, caching strategies, asset management, theme development, plugin architecture, media handling, API optimization, and advanced performance patterns. Rules are based on WordPress VIP coding standards, 10up engineering best practices, and official WordPress developer documentation. Each rule includes impact assessment, clear explanations, and practical code examples demonstrating both anti-patterns and recommended implementations.

## Table of Contents

### 1. Database Optimization (CRITICAL)

- [1.1 Avoid post__not_in in WP_Query](#11-avoid-post-not-in-in-wp-query)
- [1.2 Request Only Required Fields](#12-request-only-required-fields)
- [1.3 Always Limit Query Results](#13-always-limit-query-results)
- [1.4 Optimize Meta Queries with Proper Indexing](#14-optimize-meta-queries-with-proper-indexing)
- [1.5 Use Prepared Statements for Database Queries](#15-use-prepared-statements-for-database-queries)
- [1.6 Use WP_Query Instead of Direct Database Queries](#16-use-wp-query-instead-of-direct-database-queries)

### 2. Caching Strategies (CRITICAL)

- [2.1 Use Fragment Caching for Expensive Template Parts](#21-use-fragment-caching-for-expensive-template-parts)
- [2.2 Implement Proper Cache Invalidation](#22-implement-proper-cache-invalidation)
- [2.3 Leverage Object Cache with Cache Groups](#23-leverage-object-cache-with-cache-groups)
- [2.4 Cache Remote HTTP Requests](#24-cache-remote-http-requests)
- [2.5 Use Transients for Expensive External Operations](#25-use-transients-for-expensive-external-operations)

### 3. Asset Management (HIGH)

- [3.1 Load Assets Conditionally](#31-load-assets-conditionally)
- [3.2 Use Defer and Async for Non-Critical Scripts](#32-use-defer-and-async-for-non-critical-scripts)
- [3.3 Dequeue Unused Plugin Assets](#33-dequeue-unused-plugin-assets)
- [3.4 Minify and Combine Assets Appropriately](#34-minify-and-combine-assets-appropriately)
- [3.5 Use Proper Script and Style Enqueueing](#35-use-proper-script-and-style-enqueueing)

### 4. Theme Performance (HIGH)

- [4.1 Avoid Database Queries in Templates](#41-avoid-database-queries-in-templates)
- [4.2 Place Hooks at Appropriate Priority Levels](#42-place-hooks-at-appropriate-priority-levels)
- [4.3 Optimize WordPress Loops](#43-optimize-wordpress-loops)
- [4.4 Use Template Parts Efficiently](#44-use-template-parts-efficiently)

### 5. Plugin Architecture (MEDIUM-HIGH)

- [5.1 Use Activation and Deactivation Hooks Properly](#51-use-activation-and-deactivation-hooks-properly)
- [5.2 Use Autoloading for Plugin Classes](#52-use-autoloading-for-plugin-classes)
- [5.3 Load Plugin Code Conditionally](#53-load-plugin-code-conditionally)
- [5.4 Remove Hooks Properly When Needed](#54-remove-hooks-properly-when-needed)

### 6. Media Optimization (MEDIUM)

- [6.1 Define and Use Appropriate Image Sizes](#61-define-and-use-appropriate-image-sizes)
- [6.2 Implement Proper Lazy Loading](#62-implement-proper-lazy-loading)
- [6.3 Use Responsive Images Properly](#63-use-responsive-images-properly)

### 7. API and AJAX (MEDIUM)

- [7.1 Avoid Admin-Ajax Bottleneck](#71-avoid-admin-ajax-bottleneck)
- [7.2 Implement Proper Nonce Validation](#72-implement-proper-nonce-validation)
- [7.3 Optimize REST API Endpoints](#73-optimize-rest-api-endpoints)

### 8. Advanced Patterns (LOW-MEDIUM)

- [8.1 Optimize Options Autoloading](#81-optimize-options-autoloading)
- [8.2 Optimize WP-Cron Usage](#82-optimize-wp-cron-usage)
- [8.3 Manage Memory Usage Effectively](#83-manage-memory-usage-effectively)
- [8.4 Profile and Monitor Performance](#84-profile-and-monitor-performance)

---

## 1. Database Optimization

**Impact Level:** CRITICAL

Database queries are the primary bottleneck in WordPress performance. Unoptimized queries, missing indexes, direct database access, and improper use of WP_Query can cause severe slowdowns, especially on high-traffic sites. Following WordPress VIP standards for database operations is essential for scalable applications.

### 1.1 Avoid post__not_in in WP_Query

**Impact: CRITICAL (Can cause 10-100x slower queries on large sites)**

*Tags: database, wp_query, performance, vip*

The `post__not_in` parameter in WP_Query creates a `NOT IN` SQL clause that cannot use indexes effectively. On sites with large posts tables, this causes full table scans and severely impacts performance. This is explicitly flagged by WordPress VIP coding standards.

**Incorrect (using post__not_in):**

```php
// Avoid - causes performance issues on large sites
$query = new WP_Query([
    'post_type'    => 'post',
    'post__not_in' => get_option( 'sticky_posts' ),
]);

// Also problematic in pre_get_posts
add_action( 'pre_get_posts', function( $query ) {
    if ( $query->is_main_query() && $query->is_home() ) {
        $exclude_ids = get_posts_to_exclude();

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [bartekmis/wordpress-performance-best-practices](https://github.com/bartekmis/wordpress-performance-best-practices) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-03 -->
