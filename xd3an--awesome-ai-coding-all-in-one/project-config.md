---
trigger: always_on
description: Cursor rules for Swift Uikit.
---

# .cursorrules-mvvm-rxswift

# Swift UIKit MVVM + RxSwift Development Rules

You are an expert Swift iOS developer specializing in UIKit with MVVM architecture and RxSwift for reactive programming. Write clean, maintainable, and scalable code following Apple's Human Interface Guidelines and Swift best practices.

## Core Technologies
- **Language**: Swift 5.9+
- **Framework**: UIKit
- **Architecture**: MVVM (Model-View-ViewModel)
- **Reactive Programming**: RxSwift/RxCocoa
- **Dependency Management**: Swift Package Manager or CocoaPods

## Project Structure

```
MyApp/
├── App/
│   ├── AppDelegate.swift
│   ├── SceneDelegate.swift
│   └── Info.plist
├── Models/
│   ├── User.swift
│   ├── Product.swift
│   └── APIResponse.swift
├── Views/
│   ├── Controllers/
│   │   ├── HomeViewController.swift
│   │   ├── ProfileViewController.swift
│   │   └── BaseViewController.swift
│   ├── Custom/
│   │   ├── CustomButton.swift
│   │   ├── CustomTextField.swift
│   │   └── LoadingView.swift
│   └── Cells/
│       ├── UserTableViewCell.swift
│       └── ProductCollectionViewCell.swift
├── ViewModels/
│   ├── HomeViewModel.swift
│   ├── ProfileViewModel.swift
│   └── BaseViewModel.swift
├── Services/
│   ├── NetworkService.swift
│   ├── AuthService.swift
│   ├── CacheService.swift
│   └── UserDefaultsService.swift
├── Repositories/
│   ├── UserRepository.swift
│   └── ProductRepository.swift
├── Utilities/
│   ├── Extensions/
│   ├── Constants/
│   ├── Helpers/
│   └── Coordinators/
└── Resources/
    ├── Assets.xcassets
    ├── Localizable.strings
    └── Storyboards/
```

## MVVM Architecture Guidelines

### Model
- Use `Codable` for JSON parsing
- Implement `Equatable` when needed
- Keep models immutable when possible
- Use structs for simple data containers

```swift
struct User: Codable, Equatable {
    let id: Int
    let name: String
    let email: String
    let profileImageURL: String?
    
    private enum CodingKeys: String, CodingKey {
        case id, name, email
        case profileImageURL = "profile_image_url"
    }
}
```

### View (UIViewController)
- Keep view controllers lightweight
- Handle only UI-related logic
- Bind to ViewModels using RxSwift
- Use weak references to avoid retain cycles

```swift
class HomeViewController: UIViewController {
    @IBOutlet weak var tableView: UITableView!
    
    private let viewModel = HomeViewModel()
    private let disposeBag = DisposeBag()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        setupUI()
        bindViewModel()
    }
    
    private func bindViewModel() {
        viewModel.users
            .bind(to: tableView.rx.items(cellIdentifier: "UserCell")) { _, user, cell in
                // Configure cell
            }
            .disposed(by: disposeBag)
        
        viewModel.isLoading
            .bind(to: loadingIndicator.rx.isAnimating)
            .disposed(by: disposeBag)
    }
}
```

### ViewModel
- Use RxSwift subjects for data binding
- Handle business logic and data transformation
- Expose observables for the view to subscribe to
- Implement input/output pattern

```swift
class HomeViewModel {
    // MARK: - Inputs
    let refreshTrigger = PublishSubject<Void>()
    let loadMoreTrigger = PublishSubject<Void>()
    
    // MARK: - Outputs
    let users: Observable<[User]>
    let isLoading: Observable<Bool>
    let error: Observable<Error>
    
    private let userRepository: UserRepositoryProtocol
    private let disposeBag = DisposeBag()
    
    init(userRepository: UserRepositoryProtocol = UserRepository()) {
        self.userRepository = userRepository
        
        // Setup reactive streams
        users = refreshTrigger
            .startWith(())
            .flatMapLatest { [unowned self] in
                self.userRepository.getUsers()
                    .catchErrorJustReturn([])
            }
            .share(replay: 1)
        
        isLoading = Observable.merge(
            refreshTrigger.map { true },
            users.map { _ in false }
        )
        .startWith(false)
        
        error = userRepository.getUsers()
            .materialize()
            .compactMap { $0.error }
    }
}
```

## RxSwift Best Practices

### Binding Guidelines
- Always use `disposed(by: disposeBag)` to prevent memory leaks
- Use `weak self` in closures to avoid retain cycles
- Prefer `drive()` for UI binding instead of `bind(to:)`
- Use `share(replay: 1)` for expensive operations

### Error Handling
```swift
userRepository.getUsers()
    .observe(on: MainScheduler.instance)
    .catch { error in
        self.handleError(error)
        return Observable.empty()
    }
    .bind(to: tableView.rx.items)
    .disposed(by: disposeBag)
```

### Networking with RxSwift
```swift
protocol NetworkServiceProtocol {
    func request<T: Codable>(_ endpoint: Endpoint) -> Observable<T>
}

class NetworkService: NetworkServiceProtocol {
    func request<T: Codable>(_ endpoint: Endpoint) -> Observable<T> {
        return Observable.create { observer in
            let task = URLSession.shared.dataTask(with: endpoint.request) { data, response, error in
                if let error = error {
                    observer.onError(error)
                    return
                }
                
                guard let data = data else {

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [XD3an/awesome-ai-coding-all-in-one](https://github.com/XD3an/awesome-ai-coding-all-in-one) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-14 -->
