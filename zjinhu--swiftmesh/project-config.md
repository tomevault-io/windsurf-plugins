---
trigger: always_on
description: > A comprehensive AI reference for using SwiftMesh — an Alamofire + Codable wrapper with async/await, Combine, fluent configuration, file upload/download, JSON key path parsing, resilient Codable wrappers, and built-in logging.
---

# SwiftMesh AI Skill

> A comprehensive AI reference for using SwiftMesh — an Alamofire + Codable wrapper with async/await, Combine, fluent configuration, file upload/download, JSON key path parsing, resilient Codable wrappers, and built-in logging.

---

## Quick Reference Card

| Feature | Method | Description |
|---------|--------|-------------|
| **GET Request** | `.request(of: Model.self)` | Decode response to Codable model |
| **Key Path** | `.request(of: Model.self, modelKeyPath: "data.user")` | Extract nested JSON |
| **File Upload** | `.upload(of: Result.self)` | Upload file/Data/stream/multipart |
| **File Download** | `.download()` | Download or resume download |
| **Raw Data** | `.requestData()` | Get raw response Data |
| **Raw String** | `.requestString()` | Get response as String |
| **Retry Policy** | `.setInterceptor(RetryPolicy())` | Auto-retry with backoff |
| **Logging** | `Mesh.enableLog()` | Enable network logging |

---

## Core Architecture

SwiftMesh uses a **builder pattern** on the `Mesh` class. Every configuration method returns `Self`, enabling fluent chaining. The flow is:

```
Configure (Mesh + Config) → Execute (Request/Upload/Download) → Handle (Handle)
```

### File Structure

| File | Purpose |
|------|---------|
| `Mesh.swift` | Core builder class with all properties |
| `Config.swift` | Fluent chainable setters + global config |
| `Request.swift` | async/await request execution |
| `Handle.swift` | URL construction, error handling, response processing, RetryPolicy |
| `Upload.swift` | File upload (file, data, stream, multipart) |
| `Download.swift` | File download (standard, resumable) |
| `KeyPath.swift` | JSON key path decoder for nested extraction |
| `Codable+.swift` | Resilient property wrappers (@Default, @IgnoreError, @ConvertTo*) |
| `Log.swift` | Network logger (cURL, status, timing, JSON) |

---

## Usage Patterns

### 1. Basic GET Request

```swift
let result = try await Mesh()
    .setRequestMethod(.get)
    .setUrlHost("https://api.example.com")
    .setUrlPath("/weather/city/101030100")
    .request(of: Weather.self)
```

### 2. GET with JSON Key Path Extraction

Extract only a nested portion of the JSON response without parsing the entire structure:

```swift
// JSON: { "code": 200, "data": { "yesterday": { "temp": 25, "notice": "Sunny" } } }
let yesterday = try await Mesh()
    .setRequestMethod(.get)
    .setUrlHost("https://api.example.com")
    .setUrlPath("/weather")
    .request(of: Forecast.self, modelKeyPath: "data.yesterday")
```

### 3. POST Request with Parameters

```swift
let result = try await Mesh()
    .setRequestMethod(.post)
    .setUrlHost("https://api.example.com")
    .setUrlPath("/login")
    .setParameters(["username": "admin", "password": "123456"])
    .request(of: LoginResult.self)
```

### 4. POST with JSON Encoding

```swift
let result = try await Mesh()
    .setRequestMethod(.post)
    .setUrlHost("https://api.example.com")
    .setUrlPath("/api/data")
    .setRequestEncoding(JSONEncoding.default)
    .setParameters(["key": "value"])
    .request(of: Response.self)
```

### 5. Request with Custom Headers

```swift
let result = try await Mesh()
    .setUrlHost("https://api.example.com")
    .setUrlPath("/secure/data")
    .setHeads(["Authorization": "Bearer token123"])
    .request(of: SecureData.self)
```

### 6. Request with Retry Policy

```swift
let result = try await Mesh()
    .setUrlHost("https://api.example.com")
    .setUrlPath("/unstable-api")
    .setInterceptor(RetryPolicy(maxRetryCount: 3))
    .request(of: Data.self)
```

### 7. URLRequest-based Request

When you already have a `URLRequestConvertible`:

```swift
let urlRequest = try URLRequest(url: URL(string: "https://api.example.com/data")!, method: .get)
let result = try await Mesh()
    .urlRequest(urlRequest, type: Response.self)
```

### 8. Raw Data Response

```swift
let data = try await Mesh()
    .setRequestMethod(.get)
    .setUrlHost("https://api.example.com")
    .setUrlPath("/raw")
    .requestData()
```

### 9. Raw String Response

```swift
let string = try await Mesh()
    .setRequestMethod(.get)
    .setUrlHost("https://api.example.com")
    .setUrlPath("/text")
    .requestString()
```

### 10. File Download

```swift
let fileURL = try await Mesh()
    .setRequestMethod(.get)
    .setUrlHost("https://example.com")
    .setUrlPath("/files/document.pdf")
    .setDestination { _, _ in
        let dest = FileManager.default.urls(for: .documentDirectory, in: .userDomainMask)[0]
            .appendingPathComponent("document.pdf")
        return (dest, [.removePreviousFile, .createIntermediateDirectories])
    }
    .download()
```

### 11. Resumable Download

```swift
let fileURL = try await Mesh()
    .setUrlHost("https://example.com")
    .setUrlPath("/files/large-file.zip")
    .setDownloadType(.resume)
    .setResumeData(savedResumeData)  // from previous interrupted download
    .download()
```

### 12. Single File Upload (URL)

```swift
let result = try await Mesh()
    .setRequestMethod(.post)
    .setUrlHost("https://api.example.com")
    .setUrlPath("/upload")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [zjinhu/SwiftMesh](https://github.com/zjinhu/SwiftMesh) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-06-17 -->
