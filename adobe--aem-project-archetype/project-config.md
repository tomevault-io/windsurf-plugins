---
trigger: always_on
description: This is an AEM as a Cloud Service project using the Java stack.
---

# ${appTitle}
#if ( $aemVersion == "cloud")

This is an AEM as a Cloud Service project using the Java stack.

It is built locally using Maven#if ( $frontendModule != "decoupled" ) and can be tested against a local Quickstart#if ( $includeDispatcherConfig != "n" ), and the Dispatcher configuration is validated locally using the Dispatcher Tools#end#end.

Production deployments are done through Adobe Cloud Manager using Full Stack Pipelines#if ( $frontendModule == "decoupled" ) and Frontend Pipelines for decoupled frontend code#end.

The Java version used in Cloud Manager pipelines is defined in the `.cloudmanager/java-version` file. Assume the same is used for local builds.

#else

This is an AEM project targeting Adobe Managed Services or on-prem using the Java stack.

It is built locally using Maven and can be tested against a local Quickstart. 

#end
#if ( $includeCif == "y" || $includeForms == "y" || $includeFormsenrollment == "y" || $includeFormscommunications == "y" || $includeFormsheadless == "y" || $precompiledScripts == "y" )
${hash}${hash} Add-ons and extensions

#if ( $includeCif == "y" )
- **Commerce Integration Framework (CIF)**: The commerce backend endpoint is configured in `ui.config` OSGi configurations. CIF Core Components are included for building commerce experiences (product pages, catalog, search, cart, checkout). See README-CIF.md for more details.
#end
#if ( $includeForms == "y" || $includeFormsenrollment == "y" || $includeFormscommunications == "y" )
#if ( $aemVersion == "cloud" )
- **AEM Forms**: Forms Core Components are provided OOTB in AEM as a Cloud Service. The project contains Adaptive Forms components, templates, themes, and configurations for building form experiences.
#else
- **AEM Forms**: The project contains Adaptive Forms components, templates, themes, and configurations for building form experiences.
#end
#end
#if ( $includeFormsheadless == "y" )
- **Headless Adaptive Forms**: The `ui.frontend.react.forms.af` module provides a React-based rendering layer for forms consumed via the form model JSON. Forms can be rendered in external applications while leveraging AEM Forms capabilities for form logic and data handling.
#end
#if ( $precompiledScripts == "y" )
- **Precompiled Scripts**: HTL scripts from `ui.apps` are precompiled into a bundle during the build and attached as a secondary bundle artifact for improved performance. See README-precompiled-scripts.md for more details.
#end

#end
${hash}${hash} Modules

- `core`: OSGi bundle. Contains the Java code for backend services, models, and business logic#if ( $includeCif == "y" ), including commerce-specific models and servlets#end. Uses OSGi for dependency injection, Sling models for exposing content to Sling scripts and JUnit for unit testing.
#if ( $includeDispatcherConfig != "n" )
#if ( $aemVersion == "cloud" )
- `dispatcher`: Contains the cloud-optimized Dispatcher configuration, including caching and security settings. Uses immutable files that are validated by the Dispatcher SDK. 
#else
- `dispatcher`: Contains Dispatcher configuration suitable for Adobe Managed Services or on-prem deployments, including caching and security settings. 
#end
#end
- `ui.apps`: FileVault content package. Contains the application code, including components, templates, client libraries, and content structure. Uses HTL as the scripting engine.
- `ui.apps.structure`: FileVault content package. Empty module that defines the structure of the repository content.
- `ui.config`: FileVault content package. Contains OSGi configurations for the application.
- `ui.content`: FileVault content package. Contains the mutable content for the application, such as the initial site structure, templates, sample assets.
#if ( $frontendModule == "general" )
- `ui.frontend`: Frontend module built with Webpack. Compiles TypeScript/JavaScript and Sass/SCSS. During the build it's copied to the `ui.apps` module as client libraries. Uses Node.js, npm, and webpack.
#end
#if ( $frontendModule == "react" )
- `ui.frontend`: React-based SPA module built with Create React App. Uses `@adobe/aem-react-editable-components` for SPA Editor integration. During the build it's copied to the `ui.apps` module as client libraries. Run `npm start` to develop locally with a proxy to AEM (port 3000)#if ( $enableSSR == "y" ). Includes server-side rendering capabilities using Adobe I/O Runtime#end. Uses Node.js, npm, and webpack.
#end
#if ( $frontendModule == "angular" )
- `ui.frontend`: Angular-based SPA module built with Angular CLI. Uses `@adobe/aem-angular-editable-components` for SPA Editor integration. During the build it's copied to the `ui.apps` module as client libraries. Run `npm start` to develop locally with a proxy to AEM (port 4200)#if ( $enableSSR == "y" ). Includes server-side rendering capabilities using Adobe I/O Runtime#end. Uses Node.js, npm, and webpack.
#end
#if ( $frontendModule == "decoupled" )
#if ( $aemVersion == "cloud" )
- `ui.frontend`: Decoupled frontend module (headless). Consumes AEM content via JSON model APIs. Deployed via the AEM as a Cloud Service Frontend Pipeline separately from backend code. No client libraries are generated in `ui.apps`.
#else

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [adobe/aem-project-archetype](https://github.com/adobe/aem-project-archetype) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-21 -->
