---
trigger: always_on
description: As an AI assistant helping with iOS app releases, I should guide through the following process for successfully deploying iOS applications to the App Store.
---

# iOS App Release Process

As an AI assistant helping with iOS app releases, I should guide through the following process for successfully deploying iOS applications to the App Store.

## Versioning and Build Numbers

### Version Numbering Scheme

- Follow semantic versioning (MAJOR.MINOR.PATCH):
  - **MAJOR**: Breaking changes or significant UI redesigns
  - **MINOR**: New features with backward compatibility
  - **PATCH**: Bug fixes and minor improvements
- Update the version number in Xcode project settings (Info.plist)
- Increment build number for each submission, even for the same version

```swift
// Examining and updating Info.plist values programmatically
guard let infoPlistPath = Bundle.main.path(forResource: "Info", ofType: "plist"),
      let infoPlist = NSDictionary(contentsOfFile: infoPlistPath) as? [String: Any] else {
    return
}

let version = infoPlist["CFBundleShortVersionString"] as? String // Version number (e.g. "1.2.3")
let build = infoPlist["CFBundleVersion"] as? String // Build number (e.g. "42")
```

### Git Tagging

- Tag each release in Git with the version number
- Use annotated tags with release notes
- Consider using a prefix like 'v' (e.g., v1.2.0)

```bash
# Create an annotated tag
git tag -a v1.2.0 -m "Version 1.2.0: Added dark mode support and fixed login issues"

# Push tags to remote
git push origin v1.2.0
```

## Code Signing and Provisioning

### Certificate Management

- Use Xcode's Automatic Code Signing when possible
- For manual signing, ensure certificates are properly installed and maintained
- Consider using cert/match tools from fastlane for team management
- Keep track of certificate expiration dates to avoid crisis

### Provisioning Profiles

- Use appropriate profile types for different distribution methods:
  - Development: For testing on developer devices
  - Ad Hoc: For internal distribution to registered devices
  - App Store: For App Store and TestFlight distribution
- Regenerate provisioning profiles when adding new devices or capabilities
- Keep profiles organized and up to date

```ruby
# Example using fastlane match to sync certificates and profiles
match(
  type: "appstore",
  app_identifier: "com.yourcompany.yourapp",
  readonly: true # Set to false to generate new certificates if needed
)
```

## Pre-Release Checklist

### App Testing

- Test on all supported device types and iOS versions
- Verify all app capabilities and features work correctly
- Check for memory leaks and performance issues
- Run UI tests to verify critical user flows
- Test offline mode and poor network connections

### Asset Verification

- Verify all app icons are included at correct sizes
- Check launch screen on different devices
- Ensure all text is localized correctly
- Verify dark/light mode UI (if applicable)
- Check app size and optimize if necessary

### Legal Requirements

- Verify privacy policy is up to date and accessible
- Include all required legal disclosures
- Add attribution for third-party components if required
- Ensure GDPR/CCPA compliance if applicable
- Complete App Store privacy labels accurately

## TestFlight Distribution

### Internal Testing

- Upload build to App Store Connect
- Add internal testers (no review required)
- Use TestFlight groups to organize testers
- Provide clear testing instructions and focus areas
- Collect and address feedback

### External Testing

- Set up external testing groups in TestFlight
- Submit for Beta App Review
- Allow up to 48 hours for review approval
- Set an appropriate beta expiration date
- Monitor crash reports and feedback

```ruby
# Using fastlane to upload a build to TestFlight
lane :beta do
  increment_build_number
  build_app(scheme: "YourAppScheme")
  upload_to_testflight(
    skip_waiting_for_build_processing: true, 
    changelog: "New features to test: dark mode and improved search"
  )
end
```

## App Store Submission

### App Store Connect Setup

- Complete all required metadata:
  - App name and subtitle
  - Keywords for search optimization
  - Description (engaging, accurate, compliant)
  - Support URL and marketing URL
  - App Store screenshots (all required device sizes)
  - Preview videos (optional but recommended)
  - App Store icon

### App Review Guidelines

- Review Apple's guidelines before submission
- Check for common rejection reasons
- Ensure your app doesn't claim to include features it doesn't have
- Verify in-app purchases are properly implemented
- Test the app using TestFlight build, not development build

### Submission Process

- Upload final build to App Store Connect
- Complete App Store information
- Set pricing and availability
- Configure app privacy details
- Select phased release if desired
- Submit for review

```ruby
# Using fastlane to submit to App Store
lane :release do
  capture_screenshots
  increment_build_number
  build_app(scheme: "YourAppScheme")
  upload_to_app_store(
    force: true, # Skip HTML report verification
    submit_for_review: true,
    automatic_release: true,
    submission_information: {
      add_id_info_uses_idfa: false,
      export_compliance_uses_encryption: false
    }
  )
end
```

## Automating Releases with CI/CD


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [brunogama/ios-cursor-rules](https://github.com/brunogama/ios-cursor-rules) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-19 -->
