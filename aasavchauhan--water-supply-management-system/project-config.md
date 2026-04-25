---
trigger: always_on
description: **Native Android App** for water supply management with offline-first architecture, optimized for field workers and rural connectivity.
---

# 📱 Water Supply Management - Mobile App Development Guide

## 🎯 Project Vision

**Native Android App** for water supply management with offline-first architecture, optimized for field workers and rural connectivity.

## 🏗️ Architecture Overview

**Native Android Application with MVVM Architecture**

### Core Stack
- **Language**: Java 11+ (Android SDK)
- **Architecture**: MVVM (Model-View-ViewModel)
- **Navigation**: Android Navigation Component (FragmentContainerView)
- **Database**: Room Persistence Library (SQLite wrapper)
- **UI Components**: Material Design 3 (com.google.android.material)
- **Async**: LiveData + ViewModel + Coroutines (optional)
- **Dependency Injection**: Hilt (Dagger 2)
- **Image Loading**: Glide
- **Charts**: MPAndroidChart
- **Auth**: BiometricPrompt API + SharedPreferences

### Platform Details
- **Target**: Android 8.0+ (API 26+)
- **Build**: Android Studio Hedgehog+ with Gradle 8.2+
- **Testing**: JUnit 4 + Espresso + Mockito
- **Code Quality**: Android Lint + CheckStyle

### Storage Strategy (Phase 1: Local-First)
```
User Actions → Room Database (Immediate Write via Repository)
              ↓
          LiveData (Observable)
              ↓
          ViewModel (Business Logic)
              ↓
          Activity/Fragment (UI Updates)

Future Phase 2: Cloud Sync
Room DB ↔ WorkManager (Background Sync) ↔ Retrofit API (Cloud)
```

## 📂 Native Android Project Structure

```
WaterSupplyApp/
├── app/
│   ├── src/
│   │   ├── main/
│   │   │   ├── java/com/watersupply/
│   │   │   │   ├── MainActivity.java              # Main entry point
│   │   │   │   ├── WaterSupplyApplication.java    # Application class (Hilt)
│   │   │   │   ├── ui/
│   │   │   │   │   ├── auth/
│   │   │   │   │   │   ├── LoginActivity.java     # Biometric + PIN login
│   │   │   │   │   │   ├── LoginViewModel.java
│   │   │   │   │   │   └── RegisterActivity.java
│   │   │   │   │   ├── dashboard/
│   │   │   │   │   │   ├── DashboardFragment.java # Home with cards
│   │   │   │   │   │   ├── DashboardViewModel.java
│   │   │   │   │   │   └── adapters/
│   │   │   │   │   │       └── StatCardAdapter.java
│   │   │   │   │   ├── farmers/
│   │   │   │   │   │   ├── FarmerListFragment.java      # RecyclerView list
│   │   │   │   │   │   ├── FarmerListViewModel.java
│   │   │   │   │   │   ├── FarmerDetailFragment.java
│   │   │   │   │   │   ├── AddFarmerActivity.java       # Form with camera
│   │   │   │   │   │   └── adapters/
│   │   │   │   │   │       └── FarmerAdapter.java       # RecyclerView adapter
│   │   │   │   │   ├── supply/
│   │   │   │   │   │   ├── SupplyListFragment.java      # Supply history
│   │   │   │   │   │   ├── SupplyListViewModel.java
│   │   │   │   │   │   ├── NewSupplyActivity.java       # Dual billing form
│   │   │   │   │   │   ├── NewSupplyViewModel.java
│   │   │   │   │   │   └── dialogs/
│   │   │   │   │   │       └── MeterInputDialog.java    # Bottom sheet
│   │   │   │   │   ├── payments/
│   │   │   │   │   │   ├── PaymentListFragment.java
│   │   │   │   │   │   ├── PaymentListViewModel.java
│   │   │   │   │   │   ├── NewPaymentActivity.java
│   │   │   │   │   │   └── ReceiptActivity.java         # Printable receipt
│   │   │   │   │   ├── reports/
│   │   │   │   │   │   ├── ReportsFragment.java
│   │   │   │   │   │   ├── ReportsViewModel.java
│   │   │   │   │   │   └── charts/
│   │   │   │   │   │       └── ChartView.java           # MPAndroidChart
│   │   │   │   │   └── common/
│   │   │   │   │       ├── BaseActivity.java
│   │   │   │   │       ├── BaseFragment.java
│   │   │   │   │       └── LoadingDialog.java
│   │   │   │   ├── data/
│   │   │   │   │   ├── database/
│   │   │   │   │   │   ├── AppDatabase.java             # Room database
│   │   │   │   │   │   ├── DatabaseMigrations.java
│   │   │   │   │   │   ├── dao/
│   │   │   │   │   │   │   ├── FarmerDao.java           # Farmer CRUD
│   │   │   │   │   │   │   ├── SupplyEntryDao.java
│   │   │   │   │   │   │   ├── PaymentDao.java
│   │   │   │   │   │   │   └── UserDao.java
│   │   │   │   │   │   └── entities/
│   │   │   │   │   │       ├── Farmer.java              # @Entity
│   │   │   │   │   │       ├── SupplyEntry.java
│   │   │   │   │   │       ├── Payment.java
│   │   │   │   │   │       ├── User.java
│   │   │   │   │   │       └── Settings.java
│   │   │   │   │   ├── repository/
│   │   │   │   │   │   ├── FarmerRepository.java        # Data layer
│   │   │   │   │   │   ├── SupplyRepository.java
│   │   │   │   │   │   ├── PaymentRepository.java
│   │   │   │   │   │   └── AuthRepository.java
│   │   │   │   │   └── models/
│   │   │   │   │       └── enums/
│   │   │   │   │           └── BillingMethod.java
│   │   │   │   ├── services/
│   │   │   │   │   ├── BiometricAuthService.java        # BiometricPrompt
│   │   │   │   │   ├── CameraService.java               # CameraX
│   │   │   │   │   ├── PrinterService.java              # Bluetooth printing
│   │   │   │   │   ├── BackupService.java               # Export/import
│   │   │   │   │   └── SyncService.java (future)        # WorkManager sync
│   │   │   │   ├── utils/

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [aasavchauhan/Water-Supply-Management-System](https://github.com/aasavchauhan/Water-Supply-Management-System) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-04-24 -->
