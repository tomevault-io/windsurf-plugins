---
trigger: always_on
description: This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.
---

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

# CartWise iOS Application - Architecture Overview

## Executive Summary

CartWise is a comprehensive iOS shopping application built with SwiftUI that implements a sophisticated MVVM+Coordinator pattern with Core Data persistence. The app helps users manage shopping lists, compare prices across multiple stores, and participate in a social shopping community with reputation-based gamification.

---

## Development Commands

### Building and Running

**Build for iPhone 16 Simulator:**
```bash
xcodebuild -project CartWise.xcodeproj -scheme CartWise -destination "platform=iOS Simulator,name=iPhone 16" clean build
```

**Rebuild and Launch Simulator (Automated):**
```bash
./rebuild_simulator.sh
```
Note: The script is configured for a specific derived data path. You may need to adjust the `DERIVED_DATA_PATH` variable.

**Build using Xcode:**
- Open `CartWise.xcodeproj` in Xcode
- Select target device (iPhone 16 recommended)
- Press `Cmd + R` to build and run

### Testing

**Run Unit Tests:**
```bash
xcodebuild test -project CartWise.xcodeproj -scheme CartWise -destination "platform=iOS Simulator,name=iPhone 16"
```

**Run Tests in Xcode:**
- Press `Cmd + U` to run all tests
- Tests are located in `CartWiseTests/` directory

### Simulator Management

**List Available Simulators:**
```bash
xcrun simctl list devices
```

**Boot Specific Simulator:**
```bash
xcrun simctl boot "iPhone 16"
```

**Reset Simulator (Clear All Data):**
```bash
xcrun simctl erase all
```

**Install App on Booted Simulator:**
```bash
xcrun simctl install booted /path/to/CartWise.app
```

### Configuration

**Bundle Identifier:** `cs467.CartWise` (or `cs467.CartWise.brenna` for specific builds)

**MealMe API Key Setup:**
The app requires a MealMe API key for product image fetching:
1. Set environment variable: `MEALME_API_KEY=your_key_here`
2. Key is stored securely in Keychain via `KeychainHelper`
3. See `Config/APIKeys.example.swift` for reference
4. API key is gitignored for security

---

## 1. Overall Architecture Pattern

### MVVM with Coordinator Navigation

The application uses a **hybrid MVVM+Coordinator Pattern**:

- **ViewModels**: Handle business logic and state management (ProductViewModel, AuthViewModel, SocialFeedViewModel)
- **Coordinators**: Manage navigation and flow (AppCoordinator + 5 child coordinators for each tab)
- **Views**: SwiftUI views that bind to ViewModels and Coordinators
- **Services/Repository**: Data persistence and network integration

### Architecture Flow

```
CartWiseApp (Entry Point)
    ↓
AppCoordinator (Root Navigation)
    ├── AuthViewModel + LoginView/SignUpView
    └── Tab-based Navigation (when logged in)
        ├── YourListView → ShoppingListCoordinator
        ├── SearchItemsView → SearchItemsCoordinator
        ├── AddItemsView → AddItemsCoordinator
        ├── SocialFeedView → SocialFeedCoordinator (owns SocialFeedViewModel)
        └── MyProfileView → MyProfileCoordinator
```

**Key Principles**:
- Each tab has its own dedicated coordinator for managing feature-specific navigation
- Coordinators are lazily initialized only when their tab is first accessed
- ProductViewModel is shared across coordinators; SocialFeedViewModel is isolated within SocialFeedCoordinator
- All coordinators are cleaned up on logout to prevent memory leaks

---

## 2. Core Data Models and Schema

### Entity Relationship Diagram

The app uses **Core Data** with a model called `ProductModel.xcdatamodel` containing these entities:

#### Primary Entities

**GroceryItem** (Shopping List Items)
- `id: String` (UUID)
- `productName: String`
- `brand: String`
- `category: String` (ProductCategory enum)
- `barcode: String`
- `isOnSale: Bool`
- `isInShoppingList: Bool` (Soft-delete mechanism)
- `isCompleted: Bool` (For checking off items)
- `isFavorite: Bool` (For favorite items)
- `createdAt: Date`, `updatedAt: Date`, `lastUpdated: Date`
- **Relationships**:
  - `productImage: ProductImage?` (One-to-one)
  - `prices: NSSet<GroceryItemPrice>` (One-to-many)
  - `locations: NSSet<Location>` (Many-to-many)
  - `tags: NSSet<Tag>` (Many-to-many)
  - `experiences: NSSet<ShoppingExperience>` (One-to-many social feed entries)

**GroceryItemPrice** (Price Points Across Stores)
- `id: String`
- `price: Double`
- `currency: String` (Default: "USD")
- `store: String` (Store name for quick reference)
- `createdAt: Date`, `lastUpdated: Date`, `updatedBy: String`
- **Relationships**:
  - `groceryItem: GroceryItem` (Many-to-one)
  - `location: Location` (Many-to-one)

**Location** (Physical Store Locations)
- `id: String`
- `name: String` (Store name)
- `address: String`
- `city: String`, `state: String`, `zipCode: String`
- `isDefault: Bool`, `favorited: Bool`
- `createdAt: Date`, `updatedAt: Date`
- **Relationships**:
  - `user: UserEntity?` (Many-to-one)
  - `prices: NSSet<GroceryItemPrice>` (One-to-many)
  - `groceryItems: NSSet<GroceryItem>` (Many-to-many)
  - `experiences: NSSet<ShoppingExperience>` (One-to-many)

**ProductImage** (Product Images)
- `id: String`
- `imageURL: String?` (Remote URL from MealMe API)
- `imageData: Data?` (Cached image binary data)

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sergelents) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
