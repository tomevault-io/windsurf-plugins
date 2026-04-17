---
trigger: always_on
description: // ✅ Model: Data structures and business entities
---

# iOS Architecture & Design Patterns

## **MVVM-C Architecture Implementation**

### Core Components Structure
```swift
// ✅ Model: Data structures and business entities
struct Game: Codable, Identifiable {
    let id: UUID
    let name: String
    let status: GameStatus
    let players: [Player]
    let boundaries: GameBoundary
}

// ✅ View: SwiftUI views focused on presentation
struct GameDetailView: View {
    @StateObject private var viewModel: GameDetailViewModel
    let coordinator: GameCoordinator
    
    var body: some View {
        // UI implementation only
    }
}

// ✅ ViewModel: Business logic with @MainActor
@MainActor
final class GameDetailViewModel: ObservableObject {
    @Published var game: Game?
    @Published var isLoading = false
    @Published var error: Error?
    
    private let gameService: GameServiceProtocol
    
    func loadGame() async {
        // Business logic implementation
    }
}

// ✅ Coordinator: Navigation and flow control
final class GameCoordinator: ObservableObject {
    @Published var path = NavigationPath()
    
    func showGameDetail(gameId: String) {
        path.append(GameDestination.detail(gameId: gameId))
    }
}
```

## **Dependency Injection Pattern**

### Singleton Services with DI Registration
```swift
// ✅ DO: Implement singleton pattern for services
final class AuthenticationService: AuthenticationServiceProtocol {
    static let shared = AuthenticationService()
    private init() {}
    
    // DI Container registration
    static func register(in container: Container) {
        container.register(AuthenticationServiceProtocol.self) { _ in
            AuthenticationService.shared
        }.inObjectScope(.container)
    }
}

// ✅ DO: Use protocol-based dependency injection
protocol AuthenticationServiceProtocol {
    func authenticate(credentials: LoginCredentials) async throws -> User
    func logout() async
}

// ✅ DO: Inject dependencies through initializers
final class LoginViewModel: ObservableObject {
    private let authService: AuthenticationServiceProtocol
    
    init(authService: AuthenticationServiceProtocol = AuthenticationService.shared) {
        self.authService = authService
    }
}
```

### DI Container Setup
```swift
final class DIContainer {
    static let shared = DIContainer()
    private let container = Container()
    
    init() {
        registerServices()
    }
    
    private func registerServices() {
        // Core services
        AuthenticationService.register(in: container)
        LocationService.register(in: container)
        GameService.register(in: container)
        
        // ViewModels
        container.register(LoginViewModel.self) { resolver in
            LoginViewModel(authService: resolver.resolve(AuthenticationServiceProtocol.self)!)
        }
    }
    
    func resolve<T>(_ type: T.Type) -> T {
        return container.resolve(type)!
    }
}
```

## **SwiftUI Best Practices**

### State Management
```swift
// ✅ DO: Choose appropriate property wrappers
struct PlayerProfileView: View {
    @State private var isEditing = false              // Local view state
    @Binding var player: Player                       // Two-way binding
    @StateObject private var viewModel = ProfileViewModel() // Owned object
    @ObservedObject var gameState: GameStateManager   // External object
    @EnvironmentObject var auth: AuthenticationService // Injected dependency
    @AppStorage("theme") var theme = "dark"           // Persistent storage
}

// ✅ DO: Use @MainActor for ViewModels
@MainActor
final class ProfileViewModel: ObservableObject {
    @Published var isLoading = false
    @Published var error: Error?
    
    func updateProfile() async {
        // Automatically runs on main thread
    }
}
```

### View Composition
```swift
// ✅ DO: Break down complex views
struct GameMapView: View {
    @StateObject private var viewModel = GameMapViewModel()
    
    var body: some View {
        ZStack {
            MapBackgroundView()
            PlayerAnnotationsView(players: viewModel.players)
            SafeZoneOverlayView(zones: viewModel.safeZones)
            GameHUDView(gameState: viewModel.gameState)
        }
        .task {
            await viewModel.startTracking()
        }
    }
}

// ✅ DO: Create reusable components
struct PlayerAnnotationView: View {
    let player: Player
    let isTarget: Bool
    
    var body: some View {
        ZStack {
            Circle()
                .fill(isTarget ? Color.red : Color.blue)
                .frame(width: 40, height: 40)
            
            Image(systemName: "person.fill")
                .foregroundColor(.white)
        }
        .scaleEffect(isTarget ? 1.2 : 1.0)
        .animation(.spring(), value: isTarget)
    }
}
```

## **Error Handling Architecture**

### Structured Error Types
```swift
// ✅ DO: Create comprehensive error hierarchies
protocol AppError: LocalizedError {
    var userMessage: String { get }
    var isRetryable: Bool { get }
}

enum AuthenticationError: AppError {
    case invalidCredentials
    case networkError(Error)
    case serverError(statusCode: Int)
    
    var userMessage: String {
        switch self {
        case .invalidCredentials:
            return "Invalid email or password"
        case .networkError:

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Converted and distributed by [TomeVault](https://tomevault.io/claim/sethdford) — claim your Tome and manage your conversions.
<!-- tomevault:4.0:windsurf_rules:2026-04-09 -->
