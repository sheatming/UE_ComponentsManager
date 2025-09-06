# Components Manager Plugin - Technical Documentation

---

## Overview

**Components Manager** is a code plugin for Unreal Engine that provides enhanced versions of the **Add Component By Class** and **Get Component By Class** blueprint nodes. This plugin addresses the limitations of the native nodes—such as lack of functionality and cumbersome usage—by introducing a **global component cache**, **smart target resolution**, **network condition validation**, and **a polished visual interface**, greatly improving development efficiency and blueprint readability.

---

## Features

- **Enhanced Custom Nodes**: Offers more powerful and user-friendly `CM Add Component By Class` and `CM Get Component By Class` nodes.

- **Smart Target Resolution**: In most cases, there's no need to manually connect the `Target` pin. The node automatically infers the correct target (`Self` or `Get Owner`) from the blueprint context.

- **Global Component Cache System**: The plugin maintains an internal global cache. `Get` operations prioritize cache lookups, and `Add` operations automatically update the cache, significantly improving performance when frequently accessing components.

- **Network Condition Filtering**: Adds a `Condition` pin to determine network roles before adding components (e.g., Authority only, Local Controller only, etc.).

- **Advanced Visuals**:
  
  - **Categorization & Coloring**: Nodes are automatically categorized and colored based on component type (e.g., SceneComponent, MovementComponent), with appropriate icons.
  - **Collapsible Pins**: Auxiliary pins (e.g., `Target`, `bReplicate`) can be collapsed to keep blueprints tidy.
  - **Expose on Spawn**: The `Add` node can automatically generate input pins for component properties marked with `ExposeOnSpawn`.

- **Complete Error Handling**: Provides clear compile-time errors and warnings to guide correct usage.

---

## Supported Engine Versions

- Unreal Engine 5.6
- Optionally compatible with 5.0 – 5.5

---

## Supported Platforms

- Win64
- Mac

---

## Installation Guide

1. Download the `ComponentsManager` plugin from the FAB Marketplace.
2. Extract the plugin into your project's `Plugins/` folder or the engine's `Engine/Plugins/Fab/` directory.
   - Project path: `YourProject/Plugins/ComponentsManager/`
   - Engine path: `YourEngine/Engine/Plugins/Marketplace/ComponentsManager/`
3. Launch the Unreal Editor and **enable the plugin** when prompted.
4. Restart the editor. You can now search for `CM Get Component` or `CM Add Component` nodes in the blueprint context menu.

---

## Quick Start

### Get Component

1. Right-click in your blueprint and search for `CM Get Component`.
2. Select or connect the component class in the `Class` pin.
3. The `Target` pin can usually be left blank; the node will handle it automatically.
4. Output pins:
   - `ReturnValue`: Reference to the found component.
   - `IsValid`: Boolean indicating whether the component was successfully found.

### Add Component

1. Right-click in your blueprint and search for `CM Add Component`.
2. Select the component class in the `Class` pin.
3. Configure optional parameters:
   - `Enable`: Whether to perform the operation.
   - `Condition`: Network condition under which to execute (default: `None`).
   - `bReplicate`: Whether to replicate the component.
4. Output pins are similar to the `Get` node.

---

## Core API Reference

### Runtime Function Library (`UCMCoreBPLibrary`)

#### `CM_GetComponentByClass`

```cpp
UFUNCTION(BlueprintCallable, Category = "Components|Manager")
static UActorComponent* CM_GetComponentByClass(AActor* Target, TSubclassOf<UActorComponent> ComponentClass, bool& bIsValid);
```

**Parameters**:

* `Target`: The target Actor to search.

* `ComponentClass`: The class of the component to find.

* `bIsValid` (output): Whether the component was found.

**Return Value**: Reference to the found component, or `nullptr` if failed.

* * *

#### `CM_AddComponentByClass`

```cpp
UFUNCTION(BlueprintCallable, Category = "Components|Manager") 
static UActorComponent* CM_AddComponentByClass( 
    AActor* Target, 
    TSubclassOf<UActorComponent> Class, 
    bool bReplicate = false, 
    bool bManualAttachment = false,
    bool bDeferredFinish = false, 
    FTransform RelativeTransform = FTransform::Identity );
```



**Parameters**:

* `Target`: The target Actor.

* `Class`: The component class to add.

* `bReplicate`: Enable component replication.

* `bManualAttachment`: Whether to use manual or automatic attachment.

* `bDeferredFinish`: Whether to defer the completion of component creation.

* `RelativeTransform`: Relative transform to the parent (used in automatic attachment).

**Return Value**: Reference to the added component, or `nullptr` if failed.

* * *

#### `ShouldAddComponent`

```cpp
UFUNCTION(BlueprintCallable, Category = "Components|Manager") static bool ShouldAddComponent(AActor* Target, bool Enable, ECMNetworkCondition Condition);
```

Used to determine whether a component should be added based on network conditions.

* * *

### Global Cache (`FCMGlobalCache`)

(Internal implementation)  
Provides static methods `FindCached` and `Store` to manage and query cached component references.

* * *

Module Architecture
-------------------

The plugin follows a standard Runtime/Editor separation architecture, ensuring clean code and FAB compliance.

* **`ComponentsManagerRuntime`** (Runtime Module):
  
  * Type: `Runtime`
  
  * Contains all core gameplay logic: caching, lookup, network conditions, etc.
  
  * No editor dependencies; safe for packaging.

* **`ComponentsManagerEditor`** (UncookedOnly Module):
  
  * Type: `UncookedOnly`
  
  * Contains all editor-specific functionality: custom K2 nodes (`UK2Node_CM...`), Slate UI, pin factories, etc.
  
  * Responsible for compiling visual nodes into runtime function calls.

* * *

FAQ
---

**Q: Why are my previously placed nodes throwing errors after compiling?**  
**A**: Make sure the plugin is properly installed and enabled. If the issue persists, try deleting and re-adding the nodes.

**Q: Is the cache permanent?**  
**A**: The cache lifecycle is tied to the Actor. When the Actor is destroyed, its cached components are automatically cleared.

**Q: Can I dynamically add nodes at runtime?**  
**A**: No. All custom nodes are editor-only and used to generate code. They are not packaged into the game.

**Q: What should I do if I get a "missing include file" error?**  
**A**: This usually means the project files are outdated. Right-click the `.uproject` file and select "Generate Visual Studio project files".

* * *

Troubleshooting
---------------

* **Compile Errors**: Ensure your engine version is supported and all dependencies are installed.

* **Nodes Not Found**: Check if the plugin is enabled and restart the editor.

* **Functionality Issues**: Visit our support page and submit an issue with your blueprint and log files.

* * *

Support
-------

* **Documentation**: [Unreal Engine Plugin - ComponentsManager](https://github.com/sheatming/UE_ComponentsManager)

* **Issues**: [GitHub Issues](https://github.com/sheatming/UE_ComponentsManager)

* * *

Version History
---------------

| Version   | Date    | Description                                                                                                                            |
| --------- | ------- | -------------------------------------------------------------------------------------------------------------------------------------- |
| **1.0.0** | 2025-09 | Initial release. Includes CM Add/Get Component By Class nodes, caching system, network condition checks, and advanced visual features. |

* * *

### **Enjoy!**

**– 5heAtMin9**
