---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a full-stack Node.js web application for managing and analyzing video game collections. It provides a REST API backend with JSON file storage (ready for PostgreSQL migration) and a web interface for collection management and analysis.

**Core Features**:
- Complete CRUD operations for game collection items
- Advanced collection analysis and value estimation
- Platform-specific statistics and recommendations
- Interactive web dashboard with charts and visualizations
- RESTful API for external integrations
- Optimized for Raspberry Pi deployment

## Architecture

**Backend Structure**:
```
src/
├── config/          # Database connection and app configuration
├── models/          # Data models (Platform, CollectionItem, Category, Region) - PostgreSQL ready
├── services/        # Business logic (DataService, EbayPriceService, PriceEstimationService)
├── controllers/     # API controllers (Collection, Analysis, View) - Uses DataService
├── middleware/      # Custom middleware and validation
├── database/        # Database migrations and seeders (for future PostgreSQL migration)
└── utils/           # Helper utilities
```

**Key Technologies**:
- **Backend**: Node.js + Express.js
- **Storage**: JSON files with DataService (PostgreSQL-ready architecture)
- **Frontend**: EJS templates with vanilla JavaScript
- **Charts**: Chart.js for data visualization
- **Validation**: express-validator
- **Security**: Helmet, CORS, rate limiting

## Data Structure

**Current Storage (JSON Files)**:
- `data/collection.json` - Main collection items with embedded platform/category/region strings
- `data/platforms.json` - Platform definitions (Nintendo Switch, PlayStation, etc.)
- `data/market_prices.json` - Price database for value estimation (ready for scraping integration)

**Future PostgreSQL Schema**:
- `platforms` - Gaming platforms (PlayStation, Nintendo, etc.)
- `categories` - Item types (Games, Systems, Controllers, Accessories) 
- `regions` - Geographic regions (PAL España, NTSC USA, NTSC-J)
- `collection_items` - Main collection data with foreign key relationships
- `market_prices` - Price database for value estimation
- `publishers` & `developers` - Game publisher/developer information

**Current Data Model**:
```json
{
  "id": "uuid-string",
  "title": "Game Title",
  "platform": "Nintendo Switch", 
  "category": "Games",
  "region": "PAL España",
  "publisher": "Publisher Name",
  "developer": "Developer Name",
  "releaseType": "Official",
  "ownershipCondition": "CIB",
  "acquisitionDate": "2024-12-27",
  "estimatedValue": 50,
  "notes": "Game description"
}
```

## API Endpoints

**Collection Management** (All Working with DataService):
- `GET /api/collection` - List all items with filtering
- `POST /api/collection` - Create new collection item ✅ FIXED
- `GET /api/collection/:id` - Get item by ID
- `PUT /api/collection/:id` - Update existing item
- `DELETE /api/collection/:id` - Delete item
- `GET /api/collection/search?q=term` - Search collection

**Analysis & Statistics**:
- `GET /api/stats` - Basic collection statistics
- `GET /api/analysis/platforms` - Platform-wise analysis  
- `GET /api/analysis/timeline` - Acquisition timeline data
- `GET /api/filters` - Get all filter options (platforms, categories, regions) ✅ FIXED

**Web Views**:
- `GET /` - Dashboard with analytics
- `GET /collection` - Collection view with filters
- `GET /add` - Add new item form

## Service Layer

**DataService** (Currently Active):
- JSON file-based storage for collection and platforms
- Complete CRUD operations with UUID generation
- Built-in filtering, search, and statistics
- Platform analysis and timeline data
- Ready for PostgreSQL migration

**CollectionService** (PostgreSQL Ready):
- Model-based operations with foreign key relationships
- Advanced validation and constraints
- Ready for production database migration

**PriceEstimationService** (Active):
- eBay API integration for international price data
- Local database for Spanish/European market fallback
- No web scraping - 100% legal approach
- Platform-specific filtering and confidence scoring

**EbayPriceService** (Active):
- eBay Finding API integration for international prices
- Multi-platform support with category filtering
- Currency conversion (USD/GBP to EUR)
- Detailed filtering by condition and region

**CollectionAnalysisService** (Future Enhancement):
- Combined price estimation service integrating all sources
- Platform efficiency calculations with real market data
- Collection value analysis and trend tracking
- Strategic recommendations generation

## Key Business Logic

**Price Estimation**: 
- Multi-source data aggregation (Web scraping + eBay API)
- Weighted averaging based on source confidence
- Condition-based pricing adjustments (CIB > Boxed > Loose)
- Fallback to local price database for Spanish market

**Efficiency Scoring**: Combines game count (70%) and CIB percentage (30%) 
**Recommendations**: Platform-specific advice based on collection composition and value trends

## Development Commands

**Node.js Version**:
```bash

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [javierludena/game-collection-app](https://github.com/javierludena/game-collection-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-02 -->
