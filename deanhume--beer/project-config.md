---
trigger: always_on
description: Progressive Beer is a Progressive Web App (PWA) that showcases beer styles from around the world. The app leverages modern web technologies including Service Workers, Cache API, and Web App Manifest to provide offline functionality and an app-like experience.
---

# Progressive Beer - Copilot Instructions

## Project Overview
Progressive Beer is a Progressive Web App (PWA) that showcases beer styles from around the world. The app leverages modern web technologies including Service Workers, Cache API, and Web App Manifest to provide offline functionality and an app-like experience.

## Technology Stack
- **Frontend**: HTML5, CSS3, JavaScript (ES6+)
- **UI Framework**: Material Design Lite (MDL)
- **PWA Features**: Service Workers, Cache API, Web App Manifest
- **Data Sources**: 
  - BreweryDB (beer styles data)
  - Open Brewery DB API (brewery information)
- **Build Tools**: Grunt

## Design System & Branding

### Color Palette
- **Primary Brand Color**: `#FB8C00` (Orange)
- **Gradients**: 
  - Hero/Nav: `linear-gradient(135deg, rgba(102, 126, 234, 0.85) 0%, rgba(118, 75, 162, 0.85) 100%)`
  - Feature sections: `linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%)`
  - Footer: `linear-gradient(135deg, #2c3e50 0%, #34495e 100%)`
- **Text Colors**: 
  - Dark: `#333`
  - Medium: `#666`, `#555`
  - White: `#fff`

### SRM (Standard Reference Method) Color Gradients
Beer cards use accurate SRM color representations:
- **Pale Beer (SRM < 10)**: `linear-gradient(135deg, #F5E6B3 0%, #F4D03F 50%, #E8B923 100%)`
- **Amber Beer (SRM 10-20)**: `linear-gradient(135deg, #D4722B 0%, #C65D21 50%, #8B3A12 100%)`
- **Dark Beer (SRM > 20)**: `linear-gradient(135deg, #3D2817 0%, #251710 50%, #0D0805 100%)`

### Typography
- **Primary Font**: 'Open Sans' (body text)
- **Display Font**: 'Gabriela' (headings, brand name)
- **Icon Font**: Material Icons

## Architecture & File Structure

### Key Pages
1. **index.html** - Home page with beer styles grid
2. **style.html** - Individual beer style with list of beers
3. **beer.html** - Individual beer details
4. **about.html** - About the PWA with features and tech stack
5. **settings.html** - Offline storage settings

### JavaScript Files
- **index.js** - Handles beer styles display on home page
- **style.js** - Manages beer list for specific styles
- **beer.js** - Displays individual beer details and fetches brewery data from Open Brewery DB API
- **service-worker.js** - Handles caching and offline functionality

### CSS Files
- **site.css** - Main stylesheet with all custom styles
- **material.min.css** - Material Design Lite framework

## Modern Design Implementation

### Navigation & Hero
- **Clean Navigation**: White background with black text, orange hover states
- **Modern Hero Section**: Purple gradient with background image overlay
- **Hero Structure**: 
  - Index page: Full hero with large title and subtitle
  - Sub-pages: Smaller hero with page-specific content

### Component Patterns

#### Beer Cards (index.html)
- Simple clean design with SRM gradient backgrounds
- Beer style name in supporting text
- "Learn more" button
- CSS Grid layout (auto-fill, minmax(280px, 1fr))
- Hover effects: lift and shadow enhancement

#### Enhanced Beer Cards (style.html)
- All features from index cards PLUS:
- **Stat Badges**: ABV and IBU in top-right corner with glass-morphism effect
- **Beer Glass Indicator**: Animated glass with SRM-colored fill in top-left
- **Glass Shine Effect**: Diagonal gradient overlay for realism
- **Carbonation Effect**: Bubble animation on hover
- **Grain Texture**: Subtle texture overlay

#### Feature Cards
- White cards on gradient backgrounds
- Material Icons for visual interest
- Hover lift effects
- Used on index.html and about.html

#### Footer
- 4-column grid layout (responsive)
- Brand, Quick Links, Resources, Features
- Dark gradient background
- Orange accent colors for links

### Filter Chips (index.html)
- Emoji + text labels
- Orange active state
- Categories: Pale Ales, IPAs, Stouts, Wheat, Lagers, All Styles
- IPA filter matches both "India Pale Ale" and "IPA"

### Search Functionality
- Modern rounded search box
- Material Icons search icon
- Real-time filtering with list.js library

## Data Integration

### Beer Styles Data
- Source: BreweryDB
- Local JSON files: `./data/styles.json`, `./data/beers-style-{id}-page-{page}.json`
- Structure includes: name, description, SRM, ABV, IBU, etc.

### Brewery Data Enhancement
- **API**: Open Brewery DB (`https://api.openbrewerydb.org/v1/breweries/search?query={name}`)
- **Implementation**: beer.js fetches additional brewery data and displays:
  - Brewery type
  - Location (address, city, state, postal code, country)
  - Phone number
- **Error Handling**: Silently fails if API unavailable

## Service Worker & PWA Features

### Caching Strategy
- Cache-first approach for offline functionality
- Named caches: 'beer-data', 'brewery-api'
- Network timeout: 4 seconds
- Offline notification shown when cached content is available

### Manifest
- Theme color: `#FB8C00`
- Icons: 96px, 144px, 192px, 240px
- Display: standalone
- Installable as native app

## Development Guidelines

### Code Style
- Use ES6+ features (arrow functions, template literals, const/let)
- Async/await for API calls when possible
- Minimal comments (code should be self-explanatory)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [deanhume/beer](https://github.com/deanhume/beer) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-30 -->
