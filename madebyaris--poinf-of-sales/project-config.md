---
trigger: always_on
description: React Native mobile development patterns for POS System kitchen and server applications
---


# 📱 React Native Mobile Development Patterns

## 🎯 Project Overview - Mobile POS Applications

### GitHub Milestones Integration
Based on [GitHub Milestones](https://github.com/madebyaris/poinf-of-sales/milestones), we're developing:

1. **Kitchen Staff Mobile App (iOS & Android)** - Tablet and TV display optimization
2. **Server Group Mobile App (iOS & Android)** - Smartphone and tablet flexibility

## 🏗️ Cross-Platform Architecture

### Project Structure for React Native Apps
```
mobile/
├── kitchen-app/                 # Kitchen Staff Mobile App
│   ├── src/
│   │   ├── components/
│   │   │   ├── kitchen/         # Kitchen-specific components
│   │   │   ├── ui/              # Shared UI components
│   │   │   └── common/          # Cross-app components
│   │   ├── screens/
│   │   │   ├── KitchenDisplay/  # Main kitchen interface
│   │   │   ├── OrderDetails/    # Individual order management
│   │   │   └── Settings/        # App configuration
│   │   ├── services/
│   │   │   ├── api/             # API integration
│   │   │   ├── sync/            # Real-time synchronization
│   │   │   └── offline/         # Offline mode handling
│   │   └── utils/
│   ├── android/                 # Android-specific code
│   ├── ios/                     # iOS-specific code
│   └── package.json
├── server-app/                  # Server Group Mobile App
│   ├── src/
│   │   ├── components/
│   │   │   ├── pos/             # POS interface components
│   │   │   ├── payment/         # Mobile payment processing
│   │   │   └── tables/          # Table management
│   │   ├── screens/
│   │   │   ├── OrderEntry/      # Mobile order creation
│   │   │   ├── TableView/       # Table management interface
│   │   │   └── PaymentFlow/     # Mobile payment processing
│   │   └── services/
│   └── package.json
└── shared/                      # Shared code between apps
    ├── components/              # Reusable UI components
    ├── types/                   # TypeScript definitions
    ├── api/                     # API client
    └── utils/                   # Utility functions
```

## 🍳 Kitchen Staff Mobile App Patterns

### Tablet & TV Display Optimization
```typescript
// Kitchen app main component with device optimization
import React, { useEffect, useState } from 'react'
import { Dimensions, Platform } from 'react-native'
import DeviceInfo from 'react-native-device-info'
import Orientation from 'react-native-orientation-locker'

interface DeviceConfig {
  type: 'smartphone' | 'tablet' | 'tv'
  screenSize: 'small' | 'medium' | 'large' | 'extra-large'
  touchTargetSize: number
  fontSize: number
  spacing: number
}

export const KitchenApp: React.FC = () => {
  const [deviceConfig, setDeviceConfig] = useState<DeviceConfig>()
  const [orders, setOrders] = useState<KitchenOrder[]>([])

  useEffect(() => {
    initializeDeviceOptimization()
    setupRealTimeSync()
    enableOfflineMode()
  }, [])

  // ✅ CORRECT: Device-specific optimization
  const initializeDeviceOptimization = async () => {
    const { width, height } = Dimensions.get('window')
    const isTablet = await DeviceInfo.isTablet()
    const deviceType = await DeviceInfo.getDeviceType()

    // Determine device configuration
    let config: DeviceConfig

    if (deviceType === 'tv' || width > 1200) {
      // Large screen TV display
      config = {
        type: 'tv',
        screenSize: 'extra-large',
        touchTargetSize: 60, // Extra large for wall-mounted displays
        fontSize: 24,
        spacing: 32
      }
      
      // TV-specific optimizations
      Orientation.lockToLandscape()
      await setupTVDisplayMode()
      
    } else if (isTablet || width > 768) {
      // Tablet optimization
      config = {
        type: 'tablet',
        screenSize: 'large',
        touchTargetSize: 50, // Standard tablet touch targets
        fontSize: 18,
        spacing: 24
      }
      
      // Tablet-specific optimizations
      Orientation.lockToLandscape()
      await setupTabletMode()
      
    } else {
      // Smartphone fallback (not primary use case for kitchen)
      config = {
        type: 'smartphone',
        screenSize: 'medium',
        touchTargetSize: 44,
        fontSize: 16,
        spacing: 16
      }
    }

    setDeviceConfig(config)
  }

  // TV display mode configuration
  const setupTVDisplayMode = async () => {
    // Enable full-screen mode
    if (Platform.OS === 'android') {
      // Hide navigation bar for TV displays
      await DeviceInfo.getSystemName() // Android TV detection
    }
    
    // High contrast mode for distance viewing
    const tvSettings = {
      contrast: 'high',
      colorScheme: 'high-visibility',
      animations: 'reduced', // Minimize distractions
      autoRefresh: 3000 // 3-second refresh for TV displays
    }
    
    await applyDisplaySettings(tvSettings)
  }

  // Tablet mode configuration
  const setupTabletMode = async () => {
    // Enable gesture navigation
    const tabletSettings = {
      swipeGestures: true,
      hapticFeedback: true,
      multiTouch: false, // Prevent accidental gestures
      autoRefresh: 5000 // 5-second refresh for tablets
    }
    
    await applyDisplaySettings(tabletSettings)
  }

  return (
    <KitchenDisplayLayout 

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [madebyaris/poinf-of-sales](https://github.com/madebyaris/poinf-of-sales) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
