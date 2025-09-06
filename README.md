Components Manager 插件 - 技术文档
============================




概述 (Overview)
-------------

**Components Manager** 是一个用于虚幻引擎的代码插件，它提供了增强版的 **Add Component By Class** 和 **Get Component By Class** 自定义蓝图节点。该插件解决了原生节点功能单一、使用繁琐的问题，通过内置的**全局组件缓存**、**智能目标查找**、**网络条件判断**和**精美的可视化界面**，极大提升了开发效率和蓝图的可读性。 





## 功能特性 (Features)

* **增强型自定义节点**: 提供功能更强大、更易用的 `CM Add Component By Class` 和 `CM Get Component By Class` 节点。

* **智能目标解析**: 在绝大多数场景下无需手动连接 `Target` 引脚，节点会自动从蓝图上下文中推断出正确的目标（`Self` 或 `Get Owner`）。

* **全局组件缓存系统**: 插件内部维护了一个全局缓存，`Get` 操作会优先从缓存中查找，`Add` 操作会自动更新缓存，显著提升频繁获取组件时的性能。

* **网络条件过滤**: 新增 `Condition` 引脚，可在添加组件前进行网络角色判断（如：仅Authority执行、仅本地控制者执行等）。

* **高级可视化**:
  
  * **分类与着色**: 根据组件类型（如SceneComponent、MovementComponent）对节点进行自动分类、着色并显示对应图标。
  
  * **可折叠引脚**: 支持将多个辅助引脚（如 `Target`, `bReplicate`）折叠起来，保持蓝图整洁。
  
  * **生成时暴露属性**: `Add` 节点支持自动为组件的 `ExposeOnSpawn` 属性创建输入引脚。

* **完整的错误处理**: 提供清晰的编译错误和警告信息，指导用户正确使用。
  
  



支持的引擎版本 (Supported Engine Versions)
-----------------------------------

* 虚幻引擎 5.6

* （可选）兼容 5.0 - 5.5
  
  



支持的平台 (Supported Platforms)
---------------------------

* Win64

* Mac
  
  



安装指南 (Installation)
-------------------

1. 从FAB商城下载 `ComponentsManager` 插件压缩包。

2. 将插件解压至您的项目根目录下的 `Plugins/` 文件夹中，或引擎的 `Engine/Plugins/Fab/` 目录下。
   
   * 项目路径: `YourProject/Plugins/ComponentsManager/`
   
   * 引擎路径: `YourEngine/Engine/Plugins/Marketplace/ComponentsManager/`

3. 启动虚幻编辑器，在弹出提示时选择**启用插件**。

4. 重启编辑器后，即可在蓝图编辑器的上下文菜单中搜索 `CM Get Component` 或 `CM Add Component` 节点。
   
   



快速入门 (Quick Start)
------------------

#### 获取组件 (Get Component)

1. 在您的蓝图中右键，搜索 `CM Get Component`。

2. 在 `Class` 引脚上选择或连接您要获取的组件类（例如 `ActorComponent`）。

3. `Target` 引脚通常可留空，节点会自动处理。

4. 节点的输出引脚：
   
   * `ReturnValue`: 获取到的组件引用。
   
   * `IsValid`: 布尔值，表示是否成功获取到组件。

#### 添加组件 (Add Component)

1. 在您的蓝图中右键，搜索 `CM Add Component`。

2. 在 `Class` 引脚上选择您要添加的组件类。

3. 配置可选参数：
   
   * `Enable`: 是否启用此操作。
   
   * `Condition`: 在何种网络条件下执行（默认 `None`）。
   
   * `bReplicate`: 是否复制该组件。

4. 节点的输出引脚与 `Get` 节点类似。
   
   



核心API参考 (Core API Reference)
----------------------------



### 运行时函数库 (`UCMCoreBPLibrary`)

#### `CM_GetComponentByClass`

```cpp
UFUNCTION(BlueprintCallable, Category = "Components|Manager")
static UActorComponent* CM_GetComponentByClass(AActor* Target, TSubclassOf<UActorComponent> ComponentClass, bool& bIsValid);
```

**参数**:

* `Target`: 要查找的目标Actor。

* `ComponentClass`: 要查找的组件类。

* `bIsValid` (输出): 是否成功找到组件。

**返回值**: 找到的组件引用，失败则返回 `nullptr`。

#### `CM_AddComponentByClass`

```cpp
UFUNCTION(BlueprintCallable, Category = "Components|Manager")
static UActorComponent* CM_AddComponentByClass(
    AActor* Target, 
    TSubclassOf<UActorComponent> Class, 
    bool bReplicate = false, 
    bool bManualAttachment = false, 
    bool bDeferredFinish = false, 
    FTransform RelativeTransform = FTransform::Identity);
```

**参数**:

* `Target`: 要查找的目标Actor。

* `Class`: 要添加的组件类。

* `bReplicate `: 开启组件复制。

* `bManualAttachment`: 是否使用手动或自动附件。

* `bDeferredFinish` : 是否立即完成此组件的创建和注册过程。如果设置了expose-on-spawn属性，则为false。

* `RelativeTransform` : 新组件与其附着父级之间的相对变换（仅自动）。

**返回值**: 找到的组件引用，失败则返回 `nullptr`。



#### `ShouldAddComponent`

```cpp
UFUNCTION(BlueprintCallable, Category = "Components|Manager")
static bool ShouldAddComponent(AActor* Target, bool Enable, ECMNetworkCondition Condition);
```

用于根据网络条件判断是否应添加组件。



### 全局缓存 (`FCMGlobalCache`)

（此类为内部实现）  
提供静态方法 `FindCached` 和 `Store`，用于管理和查询缓存的组件引用。





## 模块架构 (Module Architecture)

插件采用标准的运行时与编辑器分离的架构，确保代码纯净且符合FAB规范。

* **`ComponentsManagerRuntime` (Runtime Module)**:
  
  * 类型: `Runtime`
  
  * 包含所有核心 gameplay 功能：组件缓存、查找逻辑、网络条件判断等。
  
  * 无任何编辑器依赖，可安全打包。

* **`ComponentsManagerEditor` (UncookedOnly Module)**:
  
  * 类型: `UncookedOnly`
  
  * 包含所有编辑器专用功能：自定义K2节点 (`UK2Node_CM...`)、Slate UI、引脚工厂等。
  
  * 负责将美观的节点界面编译成对Runtime模块函数的调用。
    
    



常见问题解答 (FAQ)
------------

**Q: 为什么我的蓝图编译后，之前放置的节点报错了？**  
**A**: 请确保插件已正确安装并启用。如果问题依旧，尝试在蓝图编辑器中删除并重新放置节点。

**Q: 缓存的数据是永久的吗？**  
**A**: 缓存的生命周期与Actor一致。当Actor被销毁时，其相关的组件缓存会被自动清理。

**Q: 插件支持在游戏运行时动态添加节点吗？**  
**A**: 不支持。所有自定义节点仅在编辑器中用于生成代码，它们本身不会打包到游戏中。

**Q: 出现“无法找到包含文件”错误怎么办？**  
**A**: 这通常是因为项目文件未更新。请右键点击 `.uproject` 文件，选择 “Generate Visual Studio project files”。
故障排除 (Troubleshooting)


* **编译错误**: 请确保您的引擎版本符合要求，并已安装所有必要的依赖。

* **节点找不到**: 检查插件是否已在编辑器中启用，并尝试重启编辑器。

* **功能异常**: 请访问我们的技术支持页面提交问题报告，并附上您的蓝图和日志文件。
  
  



## 技术支持 (Support)

* **文档链接**: [[Unreal Engina Plugin - ComponentsManager](https://github.com/sheatming/UE_ComponentsManager)]

* **Issues**: [[Issues](https://github.com/sheatming/UE_ComponentsManager/issues)]
  
  



版本历史 (Version History)
----------------------

| 版本号       | 发布日期    | 说明                                                              |
| --------- | ------- | --------------------------------------------------------------- |
| **1.0.0** | 2025-09 | 初始版本发布。包含 CM Add/Get Component By Class 核心节点、缓存系统、网络条件判断和高级可视化。 |
|           |         |                                                                 |

* * *

### **祝您使用愉快！**   **- 5heAtMin9**
