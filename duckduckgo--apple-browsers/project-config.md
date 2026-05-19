---
trigger: always_on
description: // Use weak/unowned references appropriately
---


# Performance Optimization Guidelines

## Memory Management

### Avoid Retain Cycles
```swift
// Use weak/unowned references appropriately
class ViewController: UIViewController {
    private var timer: Timer?
    
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Bad - Creates retain cycle
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { _ in
            self.updateUI()
        }
        
        // Good - Weak reference prevents retain cycle
        timer = Timer.scheduledTimer(withTimeInterval: 1.0, repeats: true) { [weak self] _ in
            self?.updateUI()
        }
    }
    
    deinit {
        timer?.invalidate()
    }
}
```

### Lazy Loading
```swift
class DataManager {
    // Load expensive resources only when needed
    private lazy var database: Database = {
        return Database()
    }()
    
    // Use computed properties for lightweight calculations
    var itemCount: Int {
        return items.count
    }
    
    // Cache expensive computations
    private var _processedData: [ProcessedItem]?
    var processedData: [ProcessedItem] {
        if let cached = _processedData {
            return cached
        }
        let processed = items.map { ProcessedItem($0) }
        _processedData = processed
        return processed
    }
}
```

### Memory-Efficient Collections
```swift
// Use appropriate collection types
struct LargeDataSet {
    // Bad - Loads all data into memory
    var allItems: [Item] {
        return database.fetchAll()
    }
    
    // Good - Use lazy sequences
    var items: LazySequence<[Item]> {
        return database.fetchAll().lazy
    }
    
    // Better - Use pagination
    func items(page: Int, pageSize: Int = 50) -> [Item] {
        return database.fetch(offset: page * pageSize, limit: pageSize)
    }
}
```

## UI Performance

### Main Thread Protection
```swift
class ImageLoader {
    func loadImage(from url: URL, completion: @escaping (UIImage?) -> Void) {
        Task {
            // Perform heavy work on background queue
            let data = try? await URLSession.shared.data(from: url).0
            let image = data.flatMap { UIImage(data: $0) }
            
            // Always update UI on main thread
            await MainActor.run {
                completion(image)
            }
        }
    }
}
```

### Efficient Table/Collection Views
```swift
class OptimizedTableViewController: UITableViewController {
    override func viewDidLoad() {
        super.viewDidLoad()
        
        // Register reusable cells
        tableView.register(CustomCell.self, forCellReuseIdentifier: "Cell")
        
        // Set estimated heights for better scrolling
        tableView.estimatedRowHeight = 44.0
        tableView.rowHeight = UITableView.automaticDimension
        
        // Enable prefetching
        tableView.prefetchDataSource = self
    }
    
    // Reuse cells efficiently
    override func tableView(_ tableView: UITableView, cellForRowAt indexPath: IndexPath) -> UITableViewCell {
        let cell = tableView.dequeueReusableCell(withIdentifier: "Cell", for: indexPath) as! CustomCell
        
        // Configure cell with minimal work
        cell.configure(with: items[indexPath.row])
        
        // Cancel any ongoing async work
        cell.prepareForReuse()
        
        return cell
    }
}

extension OptimizedTableViewController: UITableViewDataSourcePrefetching {
    func tableView(_ tableView: UITableView, prefetchRowsAt indexPaths: [IndexPath]) {
        // Preload data for upcoming cells
        let urls = indexPaths.compactMap { items[$0.row].imageURL }
        ImageCache.shared.preload(urls: urls)
    }
}
```

### Image Optimization
```swift
extension UIImage {
    // Resize images to appropriate size
    func resized(to targetSize: CGSize) -> UIImage? {
        let renderer = UIGraphicsImageRenderer(size: targetSize)
        return renderer.image { _ in
            self.draw(in: CGRect(origin: .zero, size: targetSize))
        }
    }
    
    // Decode images on background queue
    func decodedImage() -> UIImage? {
        guard let cgImage = cgImage else { return nil }
        
        let colorSpace = CGColorSpaceCreateDeviceRGB()
        let context = CGContext(
            data: nil,
            width: cgImage.width,
            height: cgImage.height,
            bitsPerComponent: 8,
            bytesPerRow: cgImage.width * 4,
            space: colorSpace,
            bitmapInfo: CGImageAlphaInfo.premultipliedLast.rawValue
        )
        
        context?.draw(cgImage, in: CGRect(x: 0, y: 0, width: cgImage.width, height: cgImage.height))
        
        guard let decodedImage = context?.makeImage() else { return nil }
        return UIImage(cgImage: decodedImage)
    }
}
```

## Network Performance

### Efficient API Calls
```swift
class APIClient {
    private let session: URLSession
    private let cache = URLCache(
        memoryCapacity: 10 * 1024 * 1024,  // 10 MB
        diskCapacity: 50 * 1024 * 1024,     // 50 MB
        diskPath: nil
    )
    
    init() {
        let configuration = URLSessionConfiguration.default
        configuration.urlCache = cache
        configuration.requestCachePolicy = .returnCacheDataElseLoad

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
