---
trigger: always_on
description: class BrowserWebView {
---


# WebKit & Browser Development Guidelines

## WebView Configuration

### Basic WebView Setup
```swift
import WebKit

class BrowserWebView {
    private lazy var webView: WKWebView = {
        let configuration = WKWebViewConfiguration()
        
        // Enable JavaScript
        configuration.preferences.javaScriptEnabled = true
        
        // Set user agent
        configuration.applicationNameForUserAgent = UserAgentManager.shared.userAgent
        
        // Configure content blockers
        configuration.userContentController = makeUserContentController()
        
        // Enable developer extras in debug
        #if DEBUG
        configuration.preferences.setValue(true, forKey: "developerExtrasEnabled")
        #endif
        
        let webView = WKWebView(frame: .zero, configuration: configuration)
        webView.allowsBackForwardNavigationGestures = true
        webView.allowsLinkPreview = true
        
        return webView
    }()
}
```

### User Scripts Management
```swift
private func makeUserContentController() -> WKUserContentController {
    let controller = WKUserContentController()
    
    // Add content blocking scripts
    let contentBlockingScript = WKUserScript(
        source: ContentBlockingUserScript.source,
        injectionTime: .atDocumentStart,
        forMainFrameOnly: false
    )
    controller.addUserScript(contentBlockingScript)
    
    // Add message handlers
    controller.add(self, name: "duckduckgo")
    
    return controller
}
```

## Tab Management

### Tab Model
```swift
class Tab: NSObject {
    let id = UUID()
    private(set) var url: URL?
    private(set) var title: String?
    private(set) var favicon: UIImage?
    
    weak var webView: WKWebView?
    weak var delegate: TabDelegate?
    
    private var observations: Set<NSKeyValueObservation> = []
    
    init(url: URL? = nil) {
        self.url = url
        super.init()
        setupWebView()
    }
    
    private func setupWebView() {
        let webView = WKWebView(frame: .zero, configuration: TabManager.shared.configuration)
        self.webView = webView
        
        // Observe properties
        observations.insert(
            webView.observe(\.url) { [weak self] _, _ in
                self?.urlDidChange()
            }
        )
        
        observations.insert(
            webView.observe(\.title) { [weak self] webView, _ in
                self?.title = webView.title
                self?.delegate?.tab(self!, didUpdateTitle: webView.title)
            }
        )
        
        observations.insert(
            webView.observe(\.estimatedProgress) { [weak self] webView, _ in
                self?.delegate?.tab(self!, didUpdateProgress: webView.estimatedProgress)
            }
        )
    }
}
```

### Tab Lifecycle
```swift
extension Tab {
    func load(url: URL) {
        let request = URLRequest(url: url)
        webView?.load(request)
    }
    
    func reload() {
        webView?.reload()
    }
    
    func stop() {
        webView?.stopLoading()
    }
    
    func goBack() {
        webView?.goBack()
    }
    
    func goForward() {
        webView?.goForward()
    }
    
    func close() {
        observations.forEach { $0.invalidate() }
        observations.removeAll()
        webView?.stopLoading()
        webView?.removeFromSuperview()
        webView = nil
    }
}
```

## Navigation Handling

### Navigation Delegate
```swift
extension BrowserViewController: WKNavigationDelegate {
    func webView(_ webView: WKWebView, decidePolicyFor navigationAction: WKNavigationAction) async -> WKNavigationActionPolicy {
        let url = navigationAction.request.url
        
        // Handle special URLs
        if let url = url, URLSchemeHandler.shared.canHandle(url) {
            URLSchemeHandler.shared.handle(url)
            return .cancel
        }
        
        // Apply content blocking
        if contentBlocker.shouldBlock(url) {
            return .cancel
        }
        
        // Check for downloads
        if shouldDownload(navigationAction) {
            startDownload(from: navigationAction.request)
            return .cancel
        }
        
        return .allow
    }
    
    func webView(_ webView: WKWebView, didStartProvisionalNavigation navigation: WKNavigation!) {
        updateProgressBar(animated: true)
        updateNavigationButtons()
    }
    
    func webView(_ webView: WKWebView, didFinish navigation: WKNavigation!) {
        hideProgressBar()
        captureHistory()
        updateFavicon()
    }
    
    func webView(_ webView: WKWebView, didFail navigation: WKNavigation!, withError error: Error) {
        handleNavigationError(error)
    }
}
```

## JavaScript Bridge

### Message Handling
```swift
extension BrowserViewController: WKScriptMessageHandler {
    func userContentController(_ userContentController: WKUserContentController, didReceive message: WKScriptMessage) {
        guard let dict = message.body as? [String: Any] else { return }
        
        switch message.name {
        case "duckduckgo":
            handleDuckDuckGoMessage(dict)
        case "autofill":
            handleAutofillMessage(dict)
        case "tracker":
            handleTrackerMessage(dict)
        default:
            break
        }
    }
    

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
