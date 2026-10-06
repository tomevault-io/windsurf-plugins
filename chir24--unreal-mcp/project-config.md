---
trigger: always_on
description: `McpBlueprintBehaviour::Author` writes a behaviour INTO a Blueprint from a recipe: variables, event
---

# BLUEPRINT BEHAVIOUR — gameplay logic written into Blueprint assets

`McpBlueprintBehaviour::Author` writes a behaviour INTO a Blueprint from a recipe: variables, event
dispatchers, custom events, functions with their bodies, logic hooked onto events other code also
uses, an Enhanced Input binding, and event-graph nodes. Every edit is an ordinary
`blueprint.build_graph` step run in process, so the result is plain engine nodes that run in a
packaged game with no plugin code. All or nothing, and re-runnable.

Every restored action whose promise is gameplay behaviour (sprint, slide, reload, ammo, inventory
and equipment functions, trigger events, hit pause, interaction events...) is built on it: the
domain handler validates its own parameters, fills a recipe file's placeholders and calls `Author`.
Never hand-roll graph nodes in a domain handler, never answer "configured" for logic that was not
authored, and never add a second mechanism for something this folder does (hooks, input, rollback).

Read this whole file before writing a recipe. Source of truth when in doubt: the code in this folder.

## FILES (13 of 25; each <= 250 pure lines)

| File | Holds |
|------|-------|
| `McpAutomationBridge_BlueprintBehaviour.h` | Public API (below) and the `Detail` internals shared by this folder |
| `...Behaviour.cpp` | `Author`: guards, plan, one batch, compile, commit or restore, save, reply text |
| `...Recipe.cpp` | `ParseRecipe`, `LoadRecipe`, top-level fields, `BuildPlan` |
| `...Members.cpp` | variables, dispatchers, custom events, functions: checks, flattening, variable flags and re-defaults |
| `...Dispatchers.cpp` | `DispatcherSignatureMismatch`: an existing dispatcher's parameters against the recipe's |
| `...HookEvents.cpp` | hook targets (overridable, component and custom events), id uniqueness, event creation with its parent call |
| `...Hooks.cpp` | hook checks, the shared Sequence hub, `"$id"` rewriting |
| `...Input.cpp` | the `input` section: host class, assets, key, node, registration, key mapping |
| `...InputAssets.cpp` | Enhanced Input module and default-class checks, the context resolver, asset creation, `EnsureInputAssets` |
| `...Snapshot.cpp` | `TakeSnapshot`, `Restore` |
| `...Ownership.cpp` | owner tags, `FindOwned`, `RemoveOwned`, `CheckRemovable`, `TagNew` |
| `...Nodes.h/.cpp` | create_node kinds CallDelegate, Message, AsyncTask (also reached by direct create_node and build_graph) |

Recipes: `plugins/McpAutomationBridge/Resources/Recipes/<Domain>/<Name>.json`.
Recipe checks: `tests/unit/plugin/behaviour_recipes.test.ts`.
The member step `add_event_dispatcher` lives in `Domains/Blueprint/Functions/...AddEventDispatcher.cpp`.

## API (namespace `McpBlueprintBehaviour`, include `Domains/BlueprintGraph/Behaviour/McpAutomationBridge_BlueprintBehaviour.h`)

```cpp
// Resources/Recipes/<Domain>/<Name>.json through ParseRecipe. Null with OutError when the file is
// missing or a placeholder has no value. Domain and Name: letters, digits, underscores.
// The file text is cached for the editor session: an edited recipe needs an editor restart.
TSharedPtr<FJsonObject> LoadRecipe(const FString& Domain, const FString& Name,
                                   const TMap<FString, TSharedPtr<FJsonValue>>& Values, FString& OutError);

// Recipe JSON text -> object. Every string VALUE that is exactly "{{Name}}" becomes Values[Name]
// with its own JSON type. Null with OutError on bad JSON, a placeholder with no value, or a string
// holding "{{" or "}}" that is not exactly one placeholder.
TSharedPtr<FJsonObject> ParseRecipe(const FString& RecipeJson,
                                    const TMap<FString, TSharedPtr<FJsonValue>>& Values, FString& OutError);

// Builds Recipe into the Blueprint, all or nothing. Game thread. Refused during PIE.
// Before: your own TakeSnapshot, taken BEFORE your own edits of the same Blueprint (see ROLLBACK).
FAuthorResult Author(UMcpAutomationBridgeSubsystem& Bridge, const FString& RequestId,
                     const TSharedPtr<FMcpBridgeWebSocket>& Socket, const FString& BlueprintPath,
                     const TSharedPtr<FJsonObject>& Recipe, const FSnapshot* Before = nullptr);

struct FAuthorResult
{
    bool bApplied;                   // in the Blueprint, and the Blueprint was saved
    FString Message;                 // one sentence; on failure names the recipe step that failed
    FString ErrorCode;               // set when !bApplied (ERROR CODES below, or the failing step's own)
    TSharedPtr<FJsonObject> Report;  // always set; send it as the reply's result (REPORT below)
};

FSnapshot TakeSnapshot(UBlueprint* Blueprint);          // compiles first when Dirty or Unknown
TArray<UEdGraphNode*> FindOwned(UBlueprint*, const FString& Tag);  // Num() > 0: installed
FString OwnerTag(const FString& Tag, const UEdGraphNode& Node);    // "MCP behaviour: <Tag> #<guid>"

// The one input-asset resolver (see INPUT). Creates what is missing and saves it; never rolled back.
bool EnsureInputAssets(UMcpAutomationBridgeSubsystem& Bridge, const FString& RequestId, UBlueprint* Blueprint,
                       const FString& Tag, FString& InOutActionPath, FString& InOutContextPath,

<!-- Content truncated to meet Windsurf 6KB limit -->

---
> Source: [ChiR24/Unreal_mcp](https://github.com/ChiR24/Unreal_mcp) — distributed by [TomeVault](https://tomevault.io).
<!-- tomevault:4.0:windsurf_rules:2026-10-06 -->
