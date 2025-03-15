# 能力系统（Capabilities）

### 能力系统（Capabilities）

能力系统（Capabilities）允许以动态和灵活的方式暴露功能，而无需直接实现多个接口。

总的来说，每个能力都以接口的形式提供一种功能。

NeoForge 为方块（Blocks）、实体（Entities）和物品堆栈（Item Stacks）添加了能力支持。以下部分将详细解释这些内容。

---

#### 为什么要使用能力？

能力的设计目的是将方块、实体或物品堆栈的**功能**与**实现方式**分离。如果你在考虑能力是否适合你的需求，可以问自己以下问题：

1. 你是否只关心方块、实体或物品堆栈能做什么，而不关心它如何实现？
2. 这种行为是否只适用于某些方块、实体或物品堆栈，而不是所有？
3. 这种行为的实现是否依赖于特定的方块、实体或物品堆栈？

以下是一些适合使用能力的例子：

* “我希望我的流体容器与其他模组的流体容器兼容，但我不了解每个流体容器的具体实现。” —— 是的，使用 `IFluidHandler`​ 能力。
* “我想计算某个实体中有多少物品，但我不了解实体如何存储它们。” —— 是的，使用 `IItemHandler`​ 能力。
* “我想为某个物品堆栈充能，但我不了解物品堆栈如何存储能量。” —— 是的，使用 `IEnergyStorage`​ 能力。
* “我想为玩家当前瞄准的方块上色，但我不了解方块如何被转换。” —— 是的。NeoForge 没有提供为方块上色的能力，但你可以自己实现一个。

以下是一个不推荐使用能力的例子：

* “我想检查一个实体是否在我的机器范围内。” —— 不，使用辅助方法代替。

---

#### NeoForge 提供的能力

NeoForge 为以下三个接口提供了能力支持：`IItemHandler`​、`IFluidHandler`​ 和 `IEnergyStorage`​。

1. ​**​`IItemHandler`​**​：暴露了处理物品栏槽位的接口。相关能力包括：

    * ​`Capabilities.ItemHandler.BLOCK`​：方块的自动化可访问物品栏（如箱子、机器等）。
    * ​`Capabilities.ItemHandler.ENTITY`​：实体的物品栏内容（如额外的玩家槽位、生物背包等）。
    * ​`Capabilities.ItemHandler.ENTITY_AUTOMATION`​：实体的自动化可访问物品栏（如船、矿车等）。
    * ​`Capabilities.ItemHandler.ITEM`​：物品堆栈的内容（如便携背包等）。
2. ​**​`IFluidHandler`​**​：暴露了处理流体物品栏的接口。相关能力包括：

    * ​`Capabilities.FluidHandler.BLOCK`​：方块的自动化可访问流体物品栏。
    * ​`Capabilities.FluidHandler.ENTITY`​：实体的流体物品栏。
    * ​`Capabilities.FluidHandler.ITEM`​：物品堆栈的流体物品栏（由于桶的特殊性，此能力为 `IFluidHandlerItem`​ 类型）。
3. ​**​`IEnergyStorage`​**​：暴露了处理能量容器的接口，基于 TeamCoFH 的 RedstoneFlux API。相关能力包括：

    * ​`Capabilities.EnergyStorage.BLOCK`​：方块内部的能量。
    * ​`Capabilities.EnergyStorage.ENTITY`​：实体内部的能量。
    * ​`Capabilities.EnergyStorage.ITEM`​：物品堆栈内部的能量。

---

#### 创建能力

NeoForge 支持为方块、实体和物品堆栈创建能力。

能力允许通过一些调度逻辑查找某些 API 的实现。NeoForge 中实现了以下几种能力：

* ​**​`BlockCapability`​**​：用于方块和方块实体的能力；行为依赖于特定的 `Block`​。
* ​**​`EntityCapability`​**​：用于实体的能力；行为依赖于特定的 `EntityType`​。
* ​**​`ItemCapability`​**​：用于物品堆栈的能力；行为依赖于特定的 `Item`​。

**提示**：为了与其他模组兼容，建议尽可能使用 NeoForge 提供的 `Capabilities`​ 类中的能力。否则，可以按照本节描述的方式创建自己的能力。

创建能力只需调用一个函数，结果对象应存储在静态最终字段中。必须提供以下参数：

* 能力的名称（`ResourceLocation`​）。

  * 使用相同名称多次创建能力将始终返回相同的对象。
  * 不同名称的能力是完全独立的，可以用于不同的用途。
* 被查询的行为类型（`T`​ 类型参数）。
* 查询中的额外上下文类型（`C`​ 类型参数）。

例如，以下是如何声明一个支持方向感知的方块 `IItemHandler`​ 能力：

```java
public static final BlockCapability<IItemHandler, @Nullable Direction> ITEM_HANDLER_BLOCK =
    BlockCapability.create(
        // 提供唯一标识能力的名称
        new ResourceLocation("mymod", "item_handler"),
        // 提供被查询的类型。这里我们要查找 `IItemHandler` 实例。
        IItemHandler.class,
        // 提供上下文类型。我们允许查询接收一个额外的 `Direction side` 参数。
        Direction.class
    );
```

对于不需要上下文的能力，可以使用 `Void`​ 类型，并有一个专用的辅助方法：

```java
public static final BlockCapability<IItemHandler, Void> ITEM_HANDLER_NO_CONTEXT =
    BlockCapability.createVoid(
        new ResourceLocation("mymod", "item_handler_no_context"),
        IItemHandler.class
    );
```

对于实体和物品堆栈，`EntityCapability`​ 和 `ItemCapability`​ 也有类似的方法。

---

#### 查询能力

一旦我们在静态字段中有了 `BlockCapability`​、`EntityCapability`​ 或 `ItemCapability`​ 对象，就可以查询能力。

对于实体和物品堆栈，可以使用 `getCapability`​ 尝试查找能力的实现。如果结果为 `null`​，则表示没有可用的实现。

例如：

```java
var object = entity.getCapability(CAP, context);
if (object != null) {
    // 使用 object
}

var object = stack.getCapability(CAP, context);
if (object != null) {
    // 使用 object
}
```

方块能力的查询方式略有不同，因为没有方块实体的方块也可以具有能力。查询现在在世界上执行，并额外提供目标位置：

```java
var object = level.getCapability(CAP, pos, context);
if (object != null) {
    // 使用 object
}
```

如果已知方块实体和/或方块状态，可以传递它们以节省查询时间：

```java
var object = level.getCapability(CAP, pos, blockState, blockEntity, context);
if (object != null) {
    // 使用 object
}
```

以下是一个更具体的例子，展示如何从 `Direction.NORTH`​ 方向查询方块的 `IItemHandler`​ 能力：

```java
IItemHandler handler = level.getCapability(Capabilities.ItemHandler.BLOCK, pos, Direction.NORTH);
if (handler != null) {
    // 使用 handler 进行某些物品相关操作。
}
```

---

#### 方块能力缓存

当查询能力时，系统会在幕后执行以下步骤：

1. 如果没有提供方块实体和方块状态，则获取它们。
2. 获取已注册的能力提供者（见下文）。
3. 遍历提供者并询问它们是否能提供能力。
4. 其中一个提供者将返回能力实例，可能会分配一个新对象。

虽然实现相当高效，但对于频繁执行的查询（例如每游戏刻一次），这些步骤可能会占用大量服务器时间。`BlockCapabilityCache`​ 系统为在给定位置频繁查询的能力提供了显著的加速。

**提示**：通常，`BlockCapabilityCache`​ 会创建一次，然后存储在执行频繁查询的对象的字段中。具体何时何地存储缓存由你决定。

要创建缓存，请调用 `BlockCapabilityCache.create`​，并提供要查询的能力、世界、位置和查询上下文。

```java
// 声明字段：
private BlockCapabilityCache<IItemHandler, @Nullable Direction> capCache;

// 稍后，例如在方块实体的 `onLoad` 中：
this.capCache = BlockCapabilityCache.create(
    Capabilities.ItemHandler.BLOCK, // 要缓存的能力
    level, // 世界
    pos, // 目标位置
    Direction.NORTH // 上下文
);

// 查询缓存：
IItemHandler handler = this.capCache.getCapability();
if (handler != null) {
    // 使用 handler 进行某些物品相关操作。
}
```

缓存会自动被垃圾回收器清除，无需手动注销。

---

#### 注册能力

能力提供者是最终提供能力的函数。能力提供者可以返回能力实例，如果无法提供能力则返回 `null`​。提供者特定于：

* 它们为之提供的能力。
* 它们为之提供的方块实例、方块实体类型、实体类型或物品实例。

需要在 `RegisterCapabilitiesEvent`​ 中注册提供者。

例如，以下是如何为方块注册能力：

```java
private static void registerCapabilities(RegisterCapabilitiesEvent event) {
    event.registerBlock(
        Capabilities.ItemHandler.BLOCK, // 要注册的能力
        (level, pos, state, be, side) -> <返回 IItemHandler>,
        // 要注册的方块
        MY_ITEM_HANDLER_BLOCK,
        MY_OTHER_ITEM_HANDLER_BLOCK
    );
}
```

通常，注册会针对某些方块实体类型，因此提供了 `registerBlockEntity`​ 辅助方法：

```java
event.registerBlockEntity(
    Capabilities.ItemHandler.BLOCK, // 要注册的能力
    MY_BLOCK_ENTITY_TYPE, // 要注册的方块实体类型
    (myBlockEntity, side) -> myBlockEntity.myIItemHandlerForTheGivenSide
);
```

**注意**：如果方块或方块实体提供者先前返回的能力不再有效，则必须通过调用 `level.invalidateCapabilities(pos)`​ 来使缓存失效。有关更多信息，请参阅上面的失效部分。
