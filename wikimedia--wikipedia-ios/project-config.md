---
trigger: always_on
description: This is an overview of our architecture goals for new feature and refactoring work in the Wikipedia iOS app. This is subject to change and evolve.
---

# Wikipedia iOS Architecture Guide

This is an overview of our architecture goals for new feature and refactoring work in the Wikipedia iOS app. This is subject to change and evolve.

## WMFData

WMFData is a local Swift package designed to house all of the logic pertaining to the data layer of the app. Its goals are to implement all of the networking, persistence, and caching for every feature in the app. There are several foundational classes that perform these duties. These are intended to be feature-agnostic.

### Services
`Sources > WMFData > Services`

Services are classes that are capable of making API calls via URLSession. We have two services in our app: WMFBasicService, which is capable of making unauthenticated API calls, as well as our mediaWikiService, which is capable of making authenticated API calls. Both service classes can be accessed via the WMFDataEnvironment.current singleton (located in Sources > WMFData > Environment directory), through the mediaWikiService and basicService properties.

Eventually, we want to combine these into one simple service class, but all of our legacy authentication logic still lives outside of WMFData. So for now we have split them up - any authenticated calls still use the WMFDataEnvironment.current.mediaWikiService interface, but under-the-hood it calls back to the legacy area of the app to lean on its authentication + url session calls.

### Stores
`Sources > WMFData > Store`

Stores are classes that are capable of persisting data to the app. They can also be accessed via the WMFDataEnvironment.current singleton. Currently we have 3 stores:

1. userDefaultsStore - capable of saving and loading to user defaults
2. sharedCacheStore - capable of saving and loading to the file system (note this calls back to the legacy area of the app, eventually we want to move this wholly into WMFData).

(Note that both userDefaultsStore and sharedCacheStore are accessed via a generalized protocol WMFKeyValueStore)

3. coreDataStore - capable of saving and loading to WMFData's own Core Data xcdatamodel.

Stores and services should be strictly feature-agnostic.

### Data Controllers
`Sources > WMFData > Data Controllers`

On top of stores and services sits a publicly accessible layer of classes called data controllers. These classes are usually feature-specific, but can be shared amongst multiple features as well. Data controllers are simple classes that contain references to the stores and services it needs to interact with all data related to that feature or function. Data controllers are meant to serve as an abstraction layer so that callers do not need to know where a particular piece of data is coming from (that is, remotely or locally). It should only return basic struct or class models to the callers, not Core Data NSManagedObjects.

Sometimes data controllers serve up data that needs to be re-used across multiple features, such as saved article counts. When that occurs, we ensure the data controller is named in a generic manner (i.e. feature-agnostic), and it lives in the relative Shared subdirectory). Feature-specific data controllers then call into the shared data controllers to get their data.

Sometimes data controllers have simple built-in in-memory caching, so that repeated calls to fetch the same data do not trigger repeated network or database calls. In this instance, they will be set up as a singleton, with a .shared property. Ensure all usages of these data controllers lean on {DataController}.shared to avoid multiple instances.

Ideally, public data controller methods use async throws as a method signature.

#### Notable Shared data controllers

There are a couple of shared data controllers are worth noting:

**WMFImageDataController.shared**

Use this singleton to fetch image data in WMFComponents. This data controller performs better than SwiftUI's AsyncImage. You can use them in your SwiftUI views like so:

    final class ExampleViewModel: ObservableObject {
        @Published var uiImage: Data?

        init(imageURL: URL) {
            Task {
                try await loadImage(url: imageURL)
            }
        }
        
       private func loadImage(imageURL: URL) async throws {
            let data = try await WMFImageDataController.shared.fetchImageData(url: imageURL)
            self.uiImage = UIImage(data: data)
        }
    }

    struct ExampleView: View {
        @ObservedObject var viewModel: ExampleViewModel
        
        init (viewModel: ExampleViewModel) {
            self.viewModel = viewModel
        }
        
        var body: some View {
            if let uiImage = viewModel.uiImage {
            }
        }
    }
    
**WMFArticleSummaryDataController.shared**


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [wikimedia/wikipedia-ios](https://github.com/wikimedia/wikipedia-ios) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-07-24 -->
