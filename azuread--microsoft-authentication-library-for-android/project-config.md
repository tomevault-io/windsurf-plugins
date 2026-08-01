---
trigger: always_on
description: - Cline must use hello-msal-multiple-account and hello-msal-single-account apps in the examples folder as the golden examples when generating an MSAL-integrated application. hello-msal-multiple-account for when the user wants multiple account mode, and hello-msal-single-account for when the user wants single account mode.
---

# Application Creation Guidance
- Cline must use hello-msal-multiple-account and hello-msal-single-account apps in the examples folder as the golden examples when generating an MSAL-integrated application. hello-msal-multiple-account for when the user wants multiple account mode, and hello-msal-single-account for when the user wants single account mode.
- If user does not specify between multiple account mode and single account mode in MSAL, the default mode used should be MULTIPLE. Make sure the configuration json includes which account mode the application is being built for, and use the appropriate example based on this selection.
- Ensure that the latest MSAL Version is used. Reference the MSAL repo to see latest version. At the very least, MSAL 7.+ should be used.
- Cline must copy the following gradle files exactly when creating a new application:
  1. Root level build.gradle (project level) from examples/hello-msal-multiple-account/build.gradle or examples/hello-msal-single-account/build.gradle
  2. App level build.gradle from examples/hello-msal-multiple-account/app/build.gradle or examples/hello-msal-single-account/app/build.gradle
  3. gradle.properties from examples/hello-msal-multiple-account/gradle.properties or examples/hello-msal-single-account/gradle.properties
  4. settings.gradle from examples/hello-msal-multiple-account/settings.gradle or examples/hello-msal-single-account/settings.gradle

  These files must be placed in their corresponding locations in the new application's directory structure. The only modification allowed is updating the applicationId and namespace in app/build.gradle and rootProject.name in settings.gradle to match the new application's package name. Cline can divert from these files as needed once initial creation is complete.

# MSAL API Usage
- Broker integration (Microsoft Authenticator, Company Portal, or Link To Windows) should ALWAYS be enabled for enhanced security and SSO capabilities. The broker_redirect_uri_registered field in auth_config.json should be set to "true" unless explicitly disabled by the user's request.
- ALWAYS use MSAL's Parameters-based APIs instead of deprecated methods. Here are the required patterns:

For interactive token acquisition in multiple account applications:
```java
AcquireTokenParameters parameters = new AcquireTokenParameters.Builder()
    .withScopes(SCOPES)
    .withCallback(callback)
    .build();
mPCA.acquireToken(parameters);
```

For silent token acquisition:
```java
AcquireTokenSilentParameters parameters = new AcquireTokenSilentParameters.Builder()
    .withScopes(SCOPES)
    .forAccount(account)
    .forceRefresh(false)
    .withCallback(callback)
    .build();
mPCA.acquireTokenSilent(parameters);
```

For sign-in in single account applications only:
```java
SignInParameters parameters = new SignInParameters.Builder()
    .startActivity(activity)
    .withCallback(callback)
    .build();
mPCA.signIn(parameters);
```

Important notes:
- For multiple account applications, use acquireToken for sign in, removeAccount for sign out
- For single account applications, use signIn and signOut. Also, use signInAgain if you want to reauthenticate
- acquireTokenWithDeviceCode method is only meant for back compatibility, and will be deprecated soon. It is not recommended due to security concerns in the industry.
- Do not use deprecated methods like:
  - acquireToken(activity, scopes, callback)
  - acquireTokenSilentAsync(scopes, account, authority, callback)
  - signIn(activity, scopes, callback)

# Multiple Account Mode UI Requirements
- Account spinner must include a "No Account Selected" option at position 0
- Sign In button must always be enabled to allow adding new accounts
- Sign Out button must only be enabled when an account is selected in the spinner
- Acquire Token Silent button must only be enabled when an account is selected in the spinner

# Icon and Resource Guidelines
- For application icons, only use resources that are explicitly created in the project
- Do not reference any resources (mipmap/drawable) that haven't been created
- When using adaptive icons:
  1. Create the necessary foreground vector drawable
  2. Define the background color in colors.xml
  3. Create the adaptive icon XML files
  4. Remove any references to non-existent icon resources from AndroidManifest.xml

# Code Implementation Guidelines
- Use ArrayList/List instead of arrays for better API compatibility
- Always initialize member variables in their declaration or constructor
- Use proper access modifiers (private for member variables)
- Follow Android naming conventions (mVariable for member variables)
- Handle UI updates on the main thread using activity.runOnUiThread
- Validate PublicClientApplication (PCA) initialization before making any MSAL API calls
- Refresh account lists after authentication operations
- Use proper callback interfaces for communication between components

# Configuration and Manifest

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [AzureAD/microsoft-authentication-library-for-android](https://github.com/AzureAD/microsoft-authentication-library-for-android) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-27 -->
