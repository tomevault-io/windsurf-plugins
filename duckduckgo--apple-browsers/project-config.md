---
trigger: always_on
description: Use proper service management for background agents:
---


# macOS System Integration Patterns

## Background Agents and Services
Use proper service management for background agents:

```swift
// ✅ CORRECT - Background service management
final class BackgroundServiceManager {
    private let agentIdentifier = "com.duckduckgo.agent"
    private let extensionIdentifier = "com.duckduckgo.extension"
    
    func registerBackgroundAgent() throws {
        let service = SMAppService.agent(plistName: "BackgroundAgent.plist")
        
        do {
            try service.register()
            print("Background agent registered successfully")
        } catch {
            print("Failed to register background agent: \(error)")
            throw error
        }
    }
    
    func unregisterBackgroundAgent() throws {
        let service = SMAppService.agent(plistName: "BackgroundAgent.plist")
        
        do {
            try service.unregister()
            print("Background agent unregistered successfully")
        } catch {
            print("Failed to unregister background agent: \(error)")
            throw error
        }
    }
    
    func checkServiceStatus() -> SMAppService.Status {
        let service = SMAppService.agent(plistName: "BackgroundAgent.plist")
        return service.status
    }
}

// ❌ INCORRECT - Direct background processing in main app
final class FeatureManager {
    func startBackgroundWork() {
        // Don't run continuous background work in main app
        DispatchQueue.global().async {
            while true {
                // This will drain battery and violate sandboxing
                self.performWork()
                Thread.sleep(forTimeInterval: 60)
            }
        }
    }
}
```

## System Extensions
Use proper system extension lifecycle management:

```swift
// ✅ CORRECT - System extension management
import SystemExtensions

final class SystemExtensionManager: NSObject {
    private let extensionIdentifier = "com.duckduckgo.network-extension"
    
    func installExtension() {
        let request = OSSystemExtensionRequest.activationRequest(
            forExtensionWithIdentifier: extensionIdentifier,
            queue: .main
        )
        request.delegate = self
        OSSystemExtensionManager.shared.submitRequest(request)
    }
    
    func uninstallExtension() {
        let request = OSSystemExtensionRequest.deactivationRequest(
            forExtensionWithIdentifier: extensionIdentifier,
            queue: .main
        )
        request.delegate = self
        OSSystemExtensionManager.shared.submitRequest(request)
    }
    
    func checkExtensionStatus() async -> OSSystemExtensionRequest.Result? {
        // Check if extension is already installed
        return await withCheckedContinuation { continuation in
            let request = OSSystemExtensionRequest.propertiesRequest(
                forExtensionWithIdentifier: extensionIdentifier,
                queue: .main
            )
            
            // Handle the properties request to determine status
            // Implementation details...
            continuation.resume(returning: nil)
        }
    }
}

// MARK: - OSSystemExtensionRequestDelegate
extension SystemExtensionManager: OSSystemExtensionRequestDelegate {
    func request(
        _ request: OSSystemExtensionRequest,
        actionForReplacingExtension existing: OSSystemExtensionProperties,
        withExtension extension: OSSystemExtensionProperties
    ) -> OSSystemExtensionRequest.ReplacementAction {
        return .replace
    }
    
    func requestNeedsUserApproval(_ request: OSSystemExtensionRequest) {
        print("System extension requires user approval")
        // Show UI to guide user through approval process
        showUserApprovalGuidance()
    }
    
    func request(
        _ request: OSSystemExtensionRequest,
        didFinishWithResult result: OSSystemExtensionRequest.Result
    ) {
        switch result {
        case .completed:
            print("System extension request completed successfully")
            handleExtensionActivated()
        case .willCompleteAfterReboot:
            print("System extension will be activated after reboot")
            showRebootRequiredMessage()
        @unknown default:
            print("Unknown system extension result: \(result)")
        }
    }
    
    func request(_ request: OSSystemExtensionRequest, didFailWithError error: Error) {
        print("System extension request failed: \(error)")
        handleExtensionError(error)
    }
    
    private func showUserApprovalGuidance() {
        // Show UI to guide user through System Preferences
    }
    
    private func handleExtensionActivated() {
        // Update UI to reflect extension is active
    }
    
    private func showRebootRequiredMessage() {
        // Show UI indicating reboot is required
    }
    
    private func handleExtensionError(_ error: Error) {
        // Handle extension installation errors
    }
}
```

## Login Items Management
Use the modern SMAppService API for login items:

```swift
// ✅ CORRECT - Modern login items API
import ServiceManagement

final class LoginItemsManager {
    func enableLoginItem() throws {
        do {
            try SMAppService.mainApp.register()
            print("Login item enabled successfully")

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [duckduckgo/apple-browsers](https://github.com/duckduckgo/apple-browsers) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-18 -->
