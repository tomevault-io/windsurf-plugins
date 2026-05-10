---
trigger: always_on
description: Documentation for understanding how to use the Swift Composable Architecture's Tree Navigation feature.
---


Basics
The tools for this style of navigation include the Presents() macro, PresentationAction, the ifLet(_:action:destination:fileID:filePath:line:column:) operator, and that is all. Once your feature is properly integrated with those tools you can use all of SwiftUI’s normal navigation view modifiers, such as sheet(item:), popover(item:), etc.

The process of integrating two features together for navigation largely consists of 2 steps: integrating the features’ domains together and integrating the features’ views together. One typically starts by integrating the features’ domains together. This consists of adding the child’s state and actions to the parent, and then utilizing a reducer operator to compose the child reducer into the parent.

For example, suppose you have a list of items and you want to be able to show a sheet to display a form for adding a new item. We can integrate state and actions together by utilizing the Presents() macro and PresentationAction type:

@Reducer
struct InventoryFeature {
  @ObservableState
  struct State: Equatable {
    @Presents var addItem: ItemFormFeature.State?
    var items: IdentifiedArrayOf<Item> = []
    // ...
  }


  enum Action {
    case addItem(PresentationAction<ItemFormFeature.Action>)
    // ...
  }


  // ...
}
Note

The addItem state is held as an optional. A non-nil value represents that feature is being presented, and nil presents the feature is dismissed.

Next you can integrate the reducers of the parent and child features by using the ifLet(_:action:destination:fileID:filePath:line:column:) reducer operator, as well as having an action in the parent domain for populating the child’s state to drive navigation:

@Reducer
struct InventoryFeature {
  @ObservableState
  struct State: Equatable { /* ... */ }
  enum Action { /* ... */ }
  
  var body: some ReducerOf<Self> {
    Reduce { state, action in 
      switch action {
      case .addButtonTapped:
        // Populating this state performs the navigation
        state.addItem = ItemFormFeature.State()
        return .none


      // ...
      }
    }
    .ifLet(\.$addItem, action: \.addItem) {
      ItemFormFeature()
    }
  }
}
Note

The key path used with ifLet focuses on the @PresentationState projected value since it uses the $ syntax. Also note that the action uses a case path, which is analogous to key paths but tuned for enums.

That’s all that it takes to integrate the domains and logic of the parent and child features. Next we need to integrate the features’ views. This is done by passing a binding of a store to one of SwiftUI’s view modifiers.

For example, to show a sheet from the addItem state in the InventoryFeature, we can hand the sheet(item:) modifier a binding of a Store as an argument that is focused on presentation state and actions:

struct InventoryView: View {
  @Bindable var store: StoreOf<InventoryFeature>


  var body: some View {
    List {
      // ...
    }
    .sheet(
      item: $store.scope(state: \.addItem, action: \.addItem)
    ) { store in
      ItemFormView(store: store)
    }
  }
}
Note

We use SwiftUI’s @Bindable property wrapper to produce a binding to a store, which can be further scoped using SwiftUI/Binding/scope(state:action:fileID:filePath:line:column:).

With those few steps completed the domains and views of the parent and child features are now integrated together, and when the addItem state flips to a non-nil value the sheet will be presented, and when it is nil’d out it will be dismissed.

In this example we are using the .sheet view modifier, but every view modifier SwiftUI ships can be handed a store in this fashion, including popover(item:), fullScreenCover(item:), navigationDestination(item:)`, and more. This should make it possible to use optional state to drive any kind of navigation in a SwiftUI application.

Enum state
While driving navigation with optional state can be powerful, it can also lead to less-than-ideal modeled domains. In particular, if a feature can navigate to multiple screens then you may be tempted to model that with multiple optional values:

@ObservableState
struct State {
  @Presents var detailItem: DetailFeature.State?
  @Presents var editItem: EditFeature.State?
  @Presents var addItem: AddFeature.State?
  // ...
}
However, this can lead to invalid states, such as 2 or more states being non-nil at the same time, and that can cause a lot of problems. First of all, SwiftUI does not support presenting multiple views at the same time from a single view, and so by allowing this in our state we run the risk of putting our application into an inconsistent state with respect to SwiftUI.

Second, it becomes more difficult for us to determine what feature is actually being presented. We must check multiple optionals to figure out which one is non-nil, and then we must figure out how to interpret when multiple pieces of state are non-nil at the same time.

And the number of invalid states increases exponentially with respect to the number of features that can be navigated to. For example, 3 optionals leads to 4 invalid states, 4 optionals leads to 11 invalid states, and 5 optionals leads to 26 invalid states.


<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [vocorize/app](https://github.com/vocorize/app) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-05-05 -->
