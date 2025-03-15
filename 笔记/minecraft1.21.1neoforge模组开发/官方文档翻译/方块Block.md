# 方块Block

以下是您提供的文章的中文翻译：

---

### 方块

方块是 Minecraft 世界的核心。它们构成了所有的地形、结构和机器。如果您对制作 Mod 感兴趣，那么您可能会想要添加一些方块。本页将指导您如何创建方块，并介绍一些您可以对它们进行的操作。

### 一统天下的方块

在开始之前，重要的是要理解游戏中每个方块只有一个实例。一个世界由成千上万个对该方块的引用组成，这些引用位于不同的位置。换句话说，同一个方块只是被多次显示。

因此，方块应该只在注册期间实例化一次。一旦方块被注册，您就可以根据需要使用已注册的引用。

与大多数其他注册表不同，方块可以使用一种特殊版本的 `DeferredRegister`​，称为 `DeferredRegister.Blocks`​。`DeferredRegister.Blocks`​ 基本上类似于 `DeferredRegister<Block>`​，但有一些细微的区别：

* 它们通过 `DeferredRegister.createBlocks("yourmodid")`​ 创建，而不是常规的 `DeferredRegister.create(...)`​ 方法。
* ​`#register`​ 返回一个 `DeferredBlock<T extends Block>`​，它扩展了 `DeferredHolder<Block, T>`​。`T`​ 是我们正在注册的方块的类类型。
* 有一些辅助方法用于注册方块。有关更多详细信息，请参见下文。

现在，让我们注册我们的方块：

```java
// BLOCKS 是一个 DeferredRegister.Blocks
public static final DeferredBlock<Block> MY_BLOCK = BLOCKS.register("my_block", () -> new Block(...));
```

注册方块后，所有对新方块 `my_block`​ 的引用都应使用此常量。例如，如果您想检查给定位置的方块是否为 `my_block`​，代码将如下所示：

```java
level.getBlockState(position) // 返回给定世界（level）中给定位置的方块状态
        .is(MyBlockRegistrationClass.MY_BLOCK);
```

这种方法还有一个方便的效果，即 `block1 == block2`​ 可以正常工作，并且可以代替 Java 的 `equals`​ 方法使用（当然，使用 `equals`​ 仍然有效，但由于它通过引用进行比较，因此毫无意义）。

**警告**：不要在注册之外调用 `new Block()`​！一旦这样做，事情可能会并且将会崩溃：

* 方块必须在注册表未冻结时创建。NeoForge 会为您解冻注册表并在稍后冻结它们，因此注册是您创建方块的时间窗口。
* 如果您尝试在注册表再次冻结时创建和/或注册方块，游戏将崩溃并报告一个空方块，这可能会非常令人困惑。
* 如果您仍然设法拥有一个悬空的方块实例，游戏在同步和保存时将无法识别它，并将其替换为空气。

### 创建方块

如前所述，我们首先创建 `DeferredRegister.Blocks`​：

```java
public static final DeferredRegister.Blocks BLOCKS = DeferredRegister.createBlocks("yourmodid");
```

#### 基本方块

对于不需要特殊功能的简单方块（例如圆石、木板等），可以直接使用 `Block`​ 类。为此，在注册期间，使用 `BlockBehaviour.Properties`​ 参数实例化 `Block`​。此 `BlockBehaviour.Properties`​ 参数可以使用 `BlockBehaviour.Properties#of`​ 创建，并且可以通过调用其方法进行自定义。最重要的方法包括：

* ​`destroyTime`​ - 确定方块被破坏所需的时间。

  * 石头的破坏时间为 1.5，泥土为 0.5，黑曜石为 50，基岩为 -1（不可破坏）。
* ​`explosionResistance`​ - 确定方块的爆炸抗性。

  * 石头的爆炸抗性为 6.0，泥土为 0.5，黑曜石为 1,200，基岩为 3,600,000。
* ​`sound`​ - 设置方块在被敲击、破坏或放置时发出的声音。

  * 默认值为 `SoundType.STONE`​。有关更多详细信息，请参见 [声音页面](#)。
* ​`lightLevel`​ - 设置方块的发光等级。接受一个带有 `BlockState`​ 参数的函数，返回一个介于 0 和 15 之间的值。

  * 例如，萤石使用 `state -> 15`​，火把使用 `state -> 14`​。
* ​`friction`​ - 设置方块的摩擦系数（滑度）。

  * 默认值为 0.6。冰使用 0.98。

因此，一个简单的实现将如下所示：

```java
// BLOCKS 是一个 DeferredRegister.Blocks
public static final DeferredBlock<Block> MY_BETTER_BLOCK = BLOCKS.register(
        "my_better_block", 
        () -> new Block(BlockBehaviour.Properties.of()
                .destroyTime(2.0f)
                .explosionResistance(10.0f)
                .sound(SoundType.GRAVEL)
                .lightLevel(state -> 7)
        ));
```

有关更多文档，请参见 `BlockBehaviour.Properties`​ 的源代码。有关更多示例，或查看 Minecraft 使用的值，请查看 `Blocks`​ 类。

**注意**：重要的是要理解，世界中的方块与物品栏中的方块不是同一个东西。物品栏中看起来像方块的东西实际上是一个 `BlockItem`​，这是一种特殊类型的物品，使用时可以放置一个方块。这也意味着诸如创造标签或最大堆叠大小之类的事情由相应的 `BlockItem`​ 处理。

​`BlockItem`​ 必须与方块分开注册。这是因为方块不一定需要物品，例如如果它不打算被收集（例如火）。

#### 更多功能

直接使用 `Block`​ 只允许创建非常基本的方块。如果您想添加功能，例如玩家交互或不同的碰撞箱，则需要一个扩展 `Block`​ 的自定义类。`Block`​ 类有许多可以重写的方法，用于执行不同的操作；有关更多信息，请参见 `Block`​、`BlockBehaviour`​ 和 `IBlockExtension`​ 类。有关方块的一些最常见用例，请参见下面的 [使用方块](#) 部分。

如果您想制作一个具有不同变体的方块（例如具有底部、顶部和双变体的台阶），您应该使用方块状态。最后，如果您想要一个存储额外数据的方块（例如存储其物品栏的箱子），则应使用方块实体。这里的经验法则是，如果您有有限且合理数量的状态（最多几百个状态），请使用方块状态；如果您有无限或接近无限数量的状态，请使用方块实体。

### 方块类型

方块类型是用于序列化和反序列化方块对象的 `MapCodec`​。此 `MapCodec`​ 通过 `BlockBehaviour#codec`​ 设置并注册到方块类型注册表。目前，它的唯一用途是在生成方块列表报告时使用。应为每个 `Block`​ 的子类创建一个方块类型。例如，`FlowerBlock#CODEC`​ 表示大多数花的方块类型，而其子类 `WitherRoseBlock`​ 具有单独的方块类型。

如果方块子类仅接受 `BlockBehaviour.Properties`​，则可以使用 `BlockBehaviour#simpleCodec`​ 创建 `MapCodec`​。

```java
// 对于某个方块子类
public class SimpleBlock extends Block {

    public SimpleBlock(BlockBehavior.Properties properties) {
        // ...
    }

    @Override
    public MapCodec<SimpleBlock> codec() {
        return SIMPLE_CODEC.value();
    }
}

// 在某个注册类中
public static final DeferredRegister<MapCodec<? extends Block>> REGISTRAR = DeferredRegister.create(BuiltInRegistries.BLOCK_TYPE, "yourmodid");

public static final DeferredHolder<MapCodec<? extends Block>, MapCodec<SimpleBlock>> SIMPLE_CODEC = REGISTRAR.register(
    "simple",
    () -> simpleCodec(SimpleBlock::new)
);
```

如果方块子类包含更多参数，则应使用 `RecordCodecBuilder#mapCodec`​ 创建 `MapCodec`​，并传递 `BlockBehaviour#propertiesCodec`​ 作为 `BlockBehaviour.Properties`​ 参数。

```java
// 对于某个方块子类
public class ComplexBlock extends Block {

    public ComplexBlock(int value, BlockBehavior.Properties properties) {
        // ...
    }

    @Override
    public MapCodec<ComplexBlock> codec() {
        return COMPLEX_CODEC.value();
    }

    public int getValue() {
        return this.value;
    }
}

// 在某个注册类中
public static final DeferredRegister<MapCodec<? extends Block>> REGISTRAR = DeferredRegister.create(BuiltInRegistries.BLOCK_TYPE, "yourmodid");

public static final DeferredHolder<MapCodec<? extends Block>, MapCodec<ComplexBlock>> COMPLEX_CODEC = REGISTRAR.register(
    "simple",
    () -> RecordCodecBuilder.mapCodec(instance ->
        instance.group(
            Codec.INT.fieldOf("value").forGetter(ComplexBlock::getValue),
            BlockBehaviour.propertiesCodec() // 表示 BlockBehavior.Properties 参数
        ).apply(instance, ComplexBlock::new)
    );
);
```

**注意**：尽管方块类型目前基本上未被使用，但随着 Mojang 继续朝着以编解码器为中心的结构发展，预计它将变得更加重要。

### `DeferredRegister.Blocks`​ 辅助方法

我们已经讨论了如何创建 `DeferredRegister.Blocks`​，以及它返回 `DeferredBlocks`​。现在，让我们看看这个专门的 `DeferredRegister`​ 还提供了哪些其他实用工具。让我们从 `#registerBlock`​ 开始：

```java
public static final DeferredRegister.Blocks BLOCKS = DeferredRegister.createBlocks("yourmodid");

public static final DeferredBlock<Block> EXAMPLE_BLOCK = BLOCKS.registerBlock(
        "example_block",
        Block::new, // 属性将传递到的工厂。
        BlockBehaviour.Properties.of() // 要使用的属性。
);
```

在内部，这将通过将属性参数应用于提供的方块工厂（通常是构造函数）来简单地调用 `BLOCKS.register("example_block", () -> new Block(BlockBehaviour.Properties.of()))`​。

如果您想使用 `Block::new`​，您可以完全省略工厂：

```java
public static final DeferredBlock<Block> EXAMPLE_BLOCK = BLOCKS.registerSimpleBlock(
        "example_block",
        BlockBehaviour.Properties.of() // 要使用的属性。
);
```

这与前面的示例完全相同，但稍微短一些。当然，如果您想使用 `Block`​ 的子类而不是 `Block`​ 本身，您将不得不使用前面的方法。

### 资源

如果您注册了您的方块并将其放置在世界中，您会发现它缺少诸如纹理之类的东西。这是因为纹理等由 Minecraft 的资源系统处理。要将纹理应用于方块，您必须提供一个模型和一个方块状态文件，将方块与纹理和形状关联起来。请阅读链接的文章以获取更多信息。

### 使用方块

方块很少直接用于执行操作。事实上，Minecraft 中最常见的两个操作——获取某个位置的方块和在某个位置设置方块——使用的是方块状态，而不是方块。一般的设计方法是让方块定义行为，但通过方块状态实际运行行为。因此，`BlockState`​ 经常作为参数传递给 `Block`​ 的方法。有关如何使用方块状态以及如何从方块中获取方块状态的更多信息，请参见 [使用方块状态](#)。

在几种情况下，`Block`​ 的多个方法在不同时间被调用。以下小节列出了最常见的与方块相关的流程。除非另有说明，否则所有方法都在逻辑两端调用，并且应在两端返回相同的结果。

#### 放置方块

方块放置逻辑从 `BlockItem#useOn`​ 调用（或某些子类的实现，例如用于睡莲的 `PlaceOnWaterBlockItem`​）。有关游戏如何到达此处的更多信息，请参见 [交互流程](#)。在实践中，这意味着一旦右键点击 `BlockItem`​（例如圆石物品），就会调用此行为。

* 检查几个先决条件，例如您是否处于旁观模式、主手中的 `ItemStack`​ 是否启用了所有必需的功能标志，或者目标位置是否不在世界边界之外。如果至少有一个检查失败，流程结束。
* 为当前尝试放置方块的位置的方块调用 `BlockBehaviour#canBeReplaced`​。如果返回 `false`​，流程结束。返回 `true`​ 的突出情况是高草或雪层。
* 调用 `Block#getStateForPlacement`​。在这里，根据上下文（包括位置、旋转和放置方块的侧面等信息），可以返回不同的方块状态。这对于可以朝不同方向放置的方块非常有用。
* 使用上一步中获得的方块状态调用 `BlockBehaviour#canSurvive`​。如果返回 `false`​，流程结束。
* 通过 `Level#setBlock`​ 调用将方块状态设置到世界中。
* 在该 `Level#setBlock`​ 调用中，调用 `BlockBehaviour#onPlace`​。
* 调用 `Block#setPlacedBy`​。

#### 破坏方块

破坏方块稍微复杂一些，因为它需要时间。该过程大致可以分为三个阶段：“启动”、“挖掘”和“实际破坏”。

* 当按下左键时，进入“启动”阶段。
* 现在，需要按住左键，进入“挖掘”阶段。此阶段的方法每刻调用一次。
* 如果“继续”阶段未被中断（通过释放左键）并且方块被破坏，则进入“实际破坏”阶段。

或者，对于喜欢伪代码的人：

```java
leftClick();
initiatingStage();
while (leftClickIsBeingHeld()) {
    miningStage();
    if (blockIsBroken()) {
        actuallyBreakingStage();
        break;
    }
}
```

以下小节进一步将这些阶段分解为实际的方法调用。

##### “启动”阶段

* 仅客户端：触发 `InputEvent.InteractionKeyMappingTriggered`​，带有左键和主手。如果事件被取消，流程结束。
* 检查几个先决条件，例如您是否处于旁观模式、主手中的 `ItemStack`​ 是否启用了所有必需的功能标志，或者目标方块是否不在世界边界之外。如果至少有一个检查失败，流程结束。
* 触发 `PlayerInteractEvent.LeftClickBlock`​。如果事件被取消，流程结束。

  * 请注意，当事件在客户端被取消时，不会向服务器发送数据包，因此服务器上不会运行任何逻辑。
  * 但是，在服务器上取消此事件仍会导致客户端代码运行，这可能导致不同步！
* 调用 `Block#attack`​。

##### “挖掘”阶段

* 触发 `PlayerInteractEvent.LeftClickBlock`​。如果事件被取消，流程进入“完成”阶段。

  * 请注意，当事件在客户端被取消时，不会向服务器发送数据包，因此服务器上不会运行任何逻辑。
  * 但是，在服务器上取消此事件仍会导致客户端代码运行，这可能导致不同步！
* 调用 `Block#getDestroyProgress`​ 并将其添加到内部破坏进度计数器。

  * ​`Block#getDestroyProgress`​ 返回一个介于 0 和 1 之间的浮点值，表示每刻破坏进度计数器应增加多少。
* 相应地更新进度覆盖（裂纹纹理）。
* 如果破坏进度大于 1.0（即完成，即应破坏方块），则退出“挖掘”阶段并进入“实际破坏”阶段。

##### “实际破坏”阶段

* 调用 `Item#canAttackBlock`​。如果返回 `false`​（确定不应破坏方块），流程进入“完成”阶段。
* 如果方块是 `GameMasterBlock`​ 的实例，则调用 `Player#canUseGameMasterBlocks`​。这确定玩家是否有能力破坏仅限创造模式的方块。如果为 `false`​，流程进入“完成”阶段。
* 仅服务器：调用 `Player#blockActionRestricted`​。这确定当前玩家是否可以破坏方块。如果为 `true`​，流程进入“完成”阶段。
* 仅服务器：触发 `BlockEvent.BreakEvent`​。如果被取消或 `getExpToDrop`​ 返回 -1，流程进入“完成”阶段。初始取消状态由上述三个方法确定。
* 仅服务器：触发 `PlayerEvent.HarvestCheck`​。如果 `canHarvest`​ 返回 `false`​ 或传递给破坏事件的 `BlockState`​ 为 `null`​，则事件的初始经验值为 0。
* 仅服务器：如果 `PlayerEvent.HarvestCheck#canHarvest`​ 返回 `true`​，则调用 `IBlockExtension#getExpDrop`​。此值传递给 `BlockEvent.BreakEvent#getExpToDrop`​，以便在流程的后面使用。
* 仅服务器：调用 `IBlockExtension#canHarvestBlock`​。这确定方块是否可以收获，即是否可以破坏并掉落物品。
* 调用 `IBlockExtension#onDestroyedByPlayer`​。如果返回 `false`​，流程进入“完成”阶段。在该 `IBlockExtension#onDestroyedByPlayer`​ 调用中：

  * 调用 `Block#playerWillDestroy`​。
  * 通过 `Level#setBlock`​ 调用将方块状态从世界中移除，使用 `Blocks.AIR.defaultBlockState()`​ 作为方块状态参数。
  * 在该 `Level#setBlock`​ 调用中，调用 `Block#onRemove`​。
  * 调用 `Block#destroy`​。
* 仅服务器：如果之前的 `IBlockExtension#canHarvestBlock`​ 调用返回 `true`​，则调用 `Block#playerDestroy`​。
* 仅服务器：调用 `Block#dropResources`​。这确定方块被挖掘时掉落的物品。
* 仅服务器：触发 `BlockDropsEvent`​。如果事件被取消，则方块破坏时不会掉落任何物品。否则，将 `BlockDropsEvent#getDrops`​ 中的每个 `ItemEntity`​ 添加到当前世界中。
* 仅服务器：如果之前的 `IBlockExtension#getExpDrop`​ 调用返回的值大于 0，则使用该值调用 `Block#popExperience`​。

### 刻

刻是一种机制，每 1/20 秒或 50 毫秒（“一刻”）更新（刻）游戏的一部分。方块提供了不同的刻方法，这些方法以不同的方式调用。

#### 服务器刻和刻调度

​`BlockBehaviour#tick`​ 在两种情况下调用：通过默认的随机刻（见下文）或通过调度的刻。调度的刻可以通过 `Level#scheduleTick(BlockPos, Block, int)`​ 创建，其中 `int`​ 表示延迟。Vanilla 在各种地方使用此系统，例如，大型滴叶的倾斜机制严重依赖此系统。其他突出的用户是各种红石组件。

#### 客户端刻

​`Block#animateTick`​ 仅在客户端调用，每帧调用一次。这是客户端行为发生的地方，例如火把粒子生成。

#### 天气刻

天气刻由 `Block#handlePrecipitation`​ 处理，独立于常规刻运行。它仅在服务器上调用，仅在某种形式的降雨时调用，且有 1/16 的概率。例如，下雨或下雪时填充的炼药锅使用此机制。

#### 随机刻

随机刻系统独立于常规刻运行。随机刻必须通过方块的 `BlockBehaviour.Properties`​ 启用，方法是调用 `BlockBehaviour.Properties#randomTicks()`​ 方法。这使方块能够成为随机刻机制的一部分。

随机刻每刻在区块中的一定数量的方块上发生。该数量由 `randomTickSpeed`​ 游戏规则定义。默认值为 3，每刻从区块中选择 3 个随机方块。如果这些方块启用了随机刻，则调用它们各自的 `BlockBehaviour#randomTick`​ 方法。

随机刻在 Minecraft 中用于广泛的机制，例如植物生长、冰和雪的融化或铜的氧化。
