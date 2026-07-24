---
trigger: always_on
description: - **App Name**: RomM iOS
---

# Claude Code Assistenz - RomM iOS App

## 📋 Projekt Übersicht
- **App Name**: RomM iOS
- **Architektur**: Clean Architecture + MVVM
- **UI Framework**: SwiftUI
- **iOS Target**: 16.0+
- **Navigation**: NavigationStack (modern)

## 🏗️ Projekt Struktur

```
romm/
├── Data/
│   ├── DataSources/     # API Clients, Persistence
│   ├── Repositories/    # Repository Implementierungen
│   └── Services/        # Helper Services (SFTP, etc.)
├── Domain/
│   ├── Models/          # Domain Models
│   ├── UseCases/        # Business Logic (einzelne Use Cases)
│   ├── Errors/          # Custom Error Types
│   └── RepositoryProtocols/  # Repository Interfaces
├── UI/
│   ├── App/            # App Entry Points, Main Views
│   ├── Collection/     # Collection Views & ViewModels
│   ├── Platforms/      # Platform Views & ViewModels
│   ├── Search/         # Search Views & ViewModels
│   ├── SFTP/          # SFTP Device Management
│   ├── Rom/           # ROM Detail Views
│   ├── Shared/        # Reusable Components
│   ├── Components/    # UI Components
│   └── DI/           # Dependency Injection
└── CLAUDE.md         # Diese Datei (nicht in Git)
```

## 🎯 Architektur Prinzipien

### Clean Architecture
- **Domain**: Business Logic, Models, Use Cases
- **Data**: Repository Implementierungen, API Clients
- **UI**: Views, ViewModels, Components

### Dependency Injection
- **DependencyFactory**: Zentrale Factory für alle Dependencies
- **Protocol-basiert**: Alle Repositories als Protocols definiert
- **Testability**: Einfach mockbare Dependencies

### MVVM Pattern
```swift
@Observable
@MainActor  
class SomeViewModel {
    // Properties für UI State
    // Use Cases als Dependencies
    // Business Logic Methods
}
```

## 🔧 Wichtige Code Standards

### API Guidelines
- **OpenAPI First**: Verwende IMMER die OpenAPI-generierten API Wrapper wenn verfügbar
- **RommAPIClient Extensions**: Alle API Calls sollten durch RommAPIClient Wrapper-Extensions gehen
- **Authentication Setup**: Jeder API Call muss `setupAPIConfiguration()` aufrufen für Auth Headers
- **Beispiel**: Verwende `apiClient.getPlatforms()` statt direkte HTTP Requests
- **Sonderfälle**: Manche APIs (wie Collections) benötigen manuelle Implementation wenn OpenAPI unvollständig ist

### Keychain Management
- **Generischer KeychainService**: Verwende `KeychainService(service: "com.romm.servicename")` für alle Keychain-Operationen
- **Vordefinierte Services**: `KeychainService.setup` und `KeychainService.sftp` für Setup und SFTP Credentials
- **Keine direkten Security-APIs**: Immer über KeychainService abstrahieren
- **Beispiel**: `try keychain.save(key: "password", value: password)`

### API Authentication
- **Standard**: Die meisten APIs verwenden Basic Auth mit Username:Password (Base64)
- **Collections API Sonderfall**: Benötigt echten Basic Auth statt JWT Bearer Token
- **Manual Basic Auth**: `"admin:password".data(using: .utf8).base64EncodedString()`
- **Multipart Form Data**: Collections API erfordert `multipart/form-data` statt JSON

### ViewModels
```swift
@Observable
@MainActor
class MyViewModel {
    var isLoading: Bool = false
    var error: String?
    var items: [Item] = []
    
    private let getSomeItemsUseCase: GetSomeItemsUseCase
    private let deleteSomeItemUseCase: DeleteSomeItemUseCase
    private let updateSomeItemUseCase: UpdateSomeItemUseCase
    
    // Dependency Injection via Factory
    init(factory: DependencyFactoryProtocol = DefaultDependencyFactory.shared) {
        self.getSomeItemsUseCase = factory.makeGetSomeItemsUseCase()
        self.deleteSomeItemUseCase = factory.makeDeleteSomeItemUseCase()
        self.updateSomeItemUseCase = factory.makeUpdateSomeItemUseCase()
    }
    
    // Alternative: Direkte Use Case Injection (für Tests)
    init(
        getSomeItemsUseCase: GetSomeItemsUseCase,
        deleteSomeItemUseCase: DeleteSomeItemUseCase,
        updateSomeItemUseCase: UpdateSomeItemUseCase
    ) {
        self.getSomeItemsUseCase = getSomeItemsUseCase
        self.deleteSomeItemUseCase = deleteSomeItemUseCase
        self.updateSomeItemUseCase = updateSomeItemUseCase
    }
    
    func loadItems() async {
        isLoading = true
        error = nil
        
        do {
            let loadedItems = try await getSomeItemsUseCase.execute()
            self.items = loadedItems
        } catch {
            self.error = error.localizedDescription
        }
        
        isLoading = false
    }
    
    func deleteItem(_ item: Item) async {
        do {
            try await deleteSomeItemUseCase.execute(itemId: item.id)
            items.removeAll { $0.id == item.id }
        } catch {
            self.error = error.localizedDescription
        }
    }
}
```

### Factory Pattern
```swift
// Protocol für Dependency Factory
protocol DependencyFactoryProtocol {
    func makeGetSomeItemsUseCase() -> GetSomeItemsUseCase
    func makeDeleteSomeItemUseCase() -> DeleteSomeItemUseCase
    func makeMyViewModel() -> MyViewModel
}

// Implementierung
class DefaultDependencyFactory: DependencyFactoryProtocol {
    static let shared = DefaultDependencyFactory()
    
    func makeGetSomeItemsUseCase() -> GetSomeItemsUseCase {
        GetSomeItemsUseCase(repository: makeSomeRepository())
    }
    
    func makeMyViewModel() -> MyViewModel {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ilyas-hallak/romm-ios-app](https://github.com/ilyas-hallak/romm-ios-app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
