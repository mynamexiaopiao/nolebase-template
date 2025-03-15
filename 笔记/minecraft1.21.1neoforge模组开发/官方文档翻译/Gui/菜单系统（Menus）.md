# 菜单系统（Menus）

### 菜单系统（Menus）

菜单系统是图形用户界面（GUI）的一种后端实现，负责处理与某些数据持有者交互的逻辑。菜单本身并不是数据持有者，而是允许用户间接修改内部数据持有者状态的视图。因此，数据持有者不应直接与任何菜单耦合，而是应传递数据引用以便调用和修改。

---

#### MenuType（菜单类型）

菜单是动态创建和销毁的，因此它们不是注册对象。相反，注册的是另一个工厂对象，以便轻松创建和引用菜单的类型。对于菜单来说，这些工厂对象是 `MenuType`​。

​`MenuType`​ 必须被注册。

---

#### MenuSupplier（菜单提供者）

​`MenuType`​ 通过将 `MenuSupplier`​ 和 `FeatureFlagSet`​ 传递给其构造函数来创建。`MenuSupplier`​ 表示一个函数，该函数接受容器的 ID 和查看菜单的玩家库存，并返回一个新创建的 `AbstractContainerMenu`​。

```java
// 对于某个 DeferredRegister<MenuType<?>> REGISTER
public static final Supplier<MenuType<MyMenu>> MY_MENU = REGISTER.register(
    "my_menu", 
    () -> new MenuType(MyMenu::new, FeatureFlags.DEFAULT_FLAGS)
);

// 在 MyMenu 中，一个 AbstractContainerMenu 的子类
public MyMenu(int containerId, Inventory playerInv) {
    super(MY_MENU.get(), containerId);
    // ...
}
```

**注意**：容器 ID 对于单个玩家是唯一的。这意味着两个不同玩家上的相同容器 ID 将代表两个不同的菜单，即使它们查看的是相同的数据持有者。

​`MenuSupplier`​ 通常负责在客户端上创建一个菜单，并使用虚拟数据引用来存储和与服务器数据持有者同步的信息进行交互。

---

#### IContainerFactory（容器工厂）

如果客户端需要额外的信息（例如数据持有者在世界中的位置），则可以使用子类 `IContainerFactory`​。除了容器 ID 和玩家库存外，它还提供了一个 `RegistryFriendlyByteBuf`​，可以存储从服务器发送的额外信息。可以通过 `IMenuTypeExtension#create`​ 使用 `IContainerFactory`​ 创建 `MenuType`​。

```java
// 对于某个 DeferredRegister<MenuType<?>> REGISTER
public static final Supplier<MenuType<MyMenuExtra>> MY_MENU_EXTRA = REGISTER.register(
    "my_menu_extra", 
    () -> IMenuTypeExtension.create(MyMenu::new)
);

// 在 MyMenuExtra 中，一个 AbstractContainerMenu 的子类
public MyMenuExtra(int containerId, Inventory playerInv, FriendlyByteBuf extraData) {
    super(MY_MENU_EXTRA.get(), containerId);
    // 从缓冲区存储额外数据
    // ...
}
```

---

#### AbstractContainerMenu（抽象容器菜单）

所有菜单都继承自 `AbstractContainerMenu`​。菜单接受两个参数：`MenuType`​（表示菜单本身的类型）和容器 ID（表示当前访问者的菜单的唯一标识符）。

**注意**：玩家一次只能打开 100 个唯一的菜单。

每个菜单应包含两个构造函数：一个用于在服务器上初始化菜单，另一个用于在客户端上初始化菜单。用于在客户端上初始化菜单的构造函数是传递给 `MenuType`​ 的构造函数。服务器菜单构造函数中包含的任何字段都应在客户端菜单构造函数中具有默认值。

```java
// 客户端菜单构造函数
public MyMenu(int containerId, Inventory playerInventory) { // 如果需要从服务器读取数据，可选 FriendlyByteBuf 参数
    this(containerId, playerInventory, /* 这里提供任何默认参数 */);
}

// 服务器菜单构造函数
public MyMenu(int containerId, Inventory playerInventory, /* 这里提供任何额外参数 */) {
    // ...
}
```

---

#### `#stillValid`​ 和 `ContainerLevelAccess`​

​`#stillValid`​ 方法确定菜单是否应为给定玩家保持打开状态。这通常指向静态的 `#stillValid`​ 方法，该方法接受 `ContainerLevelAccess`​、玩家和菜单所附加的方块。客户端菜单必须始终为此方法返回 `true`​，而静态的 `#stillValid`​ 默认会这样做。此实现检查玩家是否位于数据存储对象所在位置的八格范围内。

​`ContainerLevelAccess`​ 提供了当前世界和封闭范围内的方块位置。在服务器上构造菜单时，可以通过调用 `ContainerLevelAccess#create`​ 创建一个新的访问器。客户端菜单构造函数可以传递 `ContainerLevelAccess#NULL`​，这将不执行任何操作。

```java
// 客户端菜单构造函数
public MyMenuAccess(int containerId, Inventory playerInventory) {
    this(containerId, playerInventory, ContainerLevelAccess.NULL);
}

// 服务器菜单构造函数
public MyMenuAccess(int containerId, Inventory playerInventory, ContainerLevelAccess access) {
    // ...
}

// 假设此菜单附加到 Supplier<Block> MY_BLOCK
@Override
public boolean stillValid(Player player) {
    return AbstractContainerMenu.stillValid(this.access, player, MY_BLOCK.get());
}
```

---

#### 数据同步

某些数据需要同时存在于服务器和客户端上以显示给玩家。为此，菜单实现了一层基本的数据同步，以便在当前数据与上次同步到客户端的数据不匹配时进行同步。对于玩家，此检查每刻执行一次。

Minecraft 默认支持两种形式的数据同步：通过 `Slot`​ 同步 `ItemStack`​ 和通过 `DataSlot`​ 同步整数。`Slot`​ 和 `DataSlot`​ 是视图，它们持有对数据存储的引用，玩家可以在屏幕上修改这些数据（假设操作有效）。这些可以通过 `#addSlot`​ 和 `#addDataSlot`​ 在菜单的构造函数中添加。

**注意**：由于 NeoForge 弃用了 `Container`​ 而改用 `IItemHandler`​ 能力，因此以下解释将围绕使用能力变体 `SlotItemHandler`​ 展开。

​`SlotItemHandler`​ 包含四个参数：表示堆栈所在库存的 `IItemHandler`​、此插槽特定表示的堆栈索引，以及插槽在屏幕上渲染的左上角位置相对于 `AbstractContainerScreen#leftPos`​ 和 `#topPos`​ 的 x 和 y 坐标。客户端菜单构造函数应始终提供相同大小的空库存实例。

在大多数情况下，菜单中包含的任何插槽首先被添加，然后是玩家的库存，最后是玩家的快捷栏。要访问菜单中的任何单个 `Slot`​，必须根据添加插槽的顺序计算索引。

​`DataSlot`​ 是一个抽象类，应实现一个 getter 和 setter 以引用存储在数据存储对象中的数据。客户端菜单构造函数应始终通过 `DataSlot#standalone`​ 提供新实例。

这些插槽和数据槽应在每次初始化新菜单时重新创建。

**注意**：尽管 `DataSlot`​ 存储一个整数，但由于网络传输的方式，它实际上被限制为短整型（-32768 到 32767）。整数的高 16 位被忽略。NeoForge 修补了数据包以向客户端提供完整的整数。

```java
// 假设我们有一个大小为 5 的数据对象的库存
// 假设我们在每次初始化服务器菜单时构造了一个 DataSlot

// 客户端菜单构造函数
public MyMenuAccess(int containerId, Inventory playerInventory) {
    this(containerId, playerInventory, new ItemStackHandler(5), DataSlot.standalone());
}

// 服务器菜单构造函数
public MyMenuAccess(int containerId, Inventory playerInventory, IItemHandler dataInventory, DataSlot dataSingle) {
    // 检查数据库存大小是否为某个固定值
    // 然后，为数据库存添加插槽
    this.addSlot(new SlotItemHandler(dataInventory, /*...*/));

    // 为玩家库存添加插槽
    this.addSlot(new Slot(playerInventory, /*...*/));

    // 为处理的整数添加数据槽
    this.addDataSlot(dataSingle);

    // ...
}
```

---

#### `#quickMoveStack`​

​`#quickMoveStack`​ 是任何菜单必须实现的第二个方法。每当堆栈被 Shift 点击或快速移动出当前插槽时，都会调用此方法，直到堆栈完全移出其先前的插槽或没有其他地方可以放置堆栈为止。该方法返回正在快速移动的插槽中的堆栈副本。

通常使用 `#moveItemStackTo`​ 在插槽之间移动堆栈，该方法将堆栈移动到第一个可用的插槽。它接受要移动的堆栈、尝试移动堆栈的第一个插槽索引（包含）、最后一个插槽索引（不包含），以及是否从第一个插槽到最后一个插槽检查（当为 `false`​ 时）或从最后一个插槽到第一个插槽检查（当为 `true`​ 时）。

在 Minecraft 的实现中，此方法的逻辑相当一致：

```java
// 假设我们有一个大小为 5 的数据库存
// 库存有 4 个输入插槽（索引 1 - 4），输出到结果插槽（索引 0）
// 我们还有 27 个玩家库存插槽和 9 个快捷栏插槽
// 因此，实际插槽索引如下：
//   - 数据库存：结果（0），输入（1 - 4）
//   - 玩家库存（5 - 31）
//   - 玩家快捷栏（32 - 40）
@Override
public ItemStack quickMoveStack(Player player, int quickMovedSlotIndex) {
    // 快速移动的插槽堆栈
    ItemStack quickMovedStack = ItemStack.EMPTY;
    // 快速移动的插槽
    Slot quickMovedSlot = this.slots.get(quickMovedSlotIndex);

    // 如果插槽在有效范围内且插槽不为空
    if (quickMovedSlot != null && quickMovedSlot.hasItem()) {
        // 获取要移动的原始堆栈
        ItemStack rawStack = quickMovedSlot.getItem();
        // 将插槽堆栈设置为原始堆栈的副本
        quickMovedStack = rawStack.copy();

        /*
        以下快速移动逻辑可以简化为：如果在数据库存中，
        尝试移动到玩家库存/快捷栏，反之亦然，适用于无法转换数据的容器（例如箱子）。
        */

        // 如果快速移动是在数据库存的结果插槽上执行的
        if (quickMovedSlotIndex == 0) {
            // 尝试将结果插槽移动到玩家库存/快捷栏
            if (!this.moveItemStackTo(rawStack, 5, 41, true)) {
                // 如果无法移动，则不再快速移动
                return ItemStack.EMPTY;
            }

            // 在结果插槽快速移动后执行逻辑
            slot.onQuickCraft(rawStack, quickMovedStack);
        }
        // 如果快速移动是在玩家库存或快捷栏插槽上执行的
        else if (quickMovedSlotIndex >= 5 && quickMovedSlotIndex < 41) {
            // 尝试将库存/快捷栏插槽移动到数据库存的输入插槽
            if (!this.moveItemStackTo(rawStack, 1, 5, false)) {
                // 如果无法移动且在玩家库存插槽中，则尝试移动到快捷栏
                if (quickMovedSlotIndex < 32) {
                    if (!this.moveItemStackTo(rawStack, 32, 41, false)) {
                        // 如果无法移动，则不再快速移动
                        return ItemStack.EMPTY;
                    }
                }
                // 否则尝试将快捷栏移动到玩家库存插槽
                else if (!this.moveItemStackTo(rawStack, 5, 32, false)) {
                    // 如果无法移动，则不再快速移动
                    return ItemStack.EMPTY;
                }
            }
        }
        // 如果快速移动是在数据库存的输入插槽上执行的，则尝试移动到玩家库存/快捷栏
        else if (!this.moveItemStackTo(rawStack, 5, 41, false)) {
            // 如果无法移动，则不再快速移动
            return ItemStack.EMPTY;
        }

        if (rawStack.isEmpty()) {
            // 如果原始堆栈已完全移出插槽，则将插槽设置为空堆栈
            quickMovedSlot.set(ItemStack.EMPTY);
        } else {
            // 否则，通知插槽堆栈计数已更改
            quickMovedSlot.setChanged();
        }

        /*
        如果菜单不代表可以转换堆栈的容器（例如箱子），则可以删除以下 if 语句和 Slot#onTake 调用。
        */
        if (rawStack.getCount() == quickMovedStack.getCount()) {
            // 如果原始堆栈无法移动到另一个插槽，则不再快速移动
            return ItemStack.EMPTY;
        }
        // 在移动后执行逻辑，处理剩余的堆栈
        quickMovedSlot.onTake(player, rawStack);
    }

    return quickMovedStack; // 返回插槽堆栈
}
```

---

#### 打开菜单

一旦注册了菜单类型、完成了菜单的实现并附加了屏幕，玩家就可以打开菜单。可以通过在逻辑服务器上调用 `IPlayerExtension#openMenu`​ 来打开菜单。该方法接受服务器端菜单的 `MenuProvider`​，如果需要将额外数据同步到客户端，还可以选择接受 `Consumer<RegistryFriendlyByteBuf>`​。

**注意**：只有在使用 `IContainerFactory`​ 创建菜单类型时，才应使用带有 `Consumer<RegistryFriendlyByteBuf>`​ 参数的 `IPlayerExtension#openMenu`​。

---

#### MenuProvider（菜单提供者）

​`MenuProvider`​ 是一个接口，包含两个方法：`#createMenu`​（创建菜单的服务器实例）和 `#getDisplayName`​（返回包含菜单标题的组件以传递给屏幕）。`#createMenu`​ 方法包含三个参数：菜单的容器 ID、打开菜单的玩家的库存以及打开菜单的玩家。

可以使用 `SimpleMenuProvider`​ 轻松创建 `MenuProvider`​，它接受创建服务器菜单的方法引用和菜单的标题。

```java
// 在某个可以访问逻辑服务器上的玩家的实现中（例如 ServerPlayer 实例）
// 假设我们有 ServerPlayer serverPlayer
serverPlayer.openMenu(new SimpleMenuProvider(
    (containerId, playerInventory, player) -> new MyMenu(containerId, playerInventory),
    Component.translatable("menu.title.examplemod.mymenu")
));
```

---

#### 常见实现

菜单通常在某些玩家交互时打开（例如右键点击方块或实体时）。

---

#### 方块实现

方块通常通过重写 `BlockBehaviour#useWithoutItem`​ 来实现菜单。如果在逻辑客户端上，交互返回 `InteractionResult#SUCCESS`​。否则，它打开菜单并返回 `InteractionResult#CONSUME`​。

​`MenuProvider`​ 应通过重写 `BlockBehaviour#getMenuProvider`​ 来实现。Vanilla 方法使用此方法在旁观模式下查看菜单。

```java
// 在某个 Block 子类中
@Override
public MenuProvider getMenuProvider(BlockState state, Level level, BlockPos pos) {
    return new SimpleMenuProvider(/* ... */);
}

@Override
public InteractionResult useWithoutItem(BlockState state, Level level, BlockPos pos, Player player, InteractionHand hand, BlockHitResult result) {
    if (!level.isClientSide && player instanceof ServerPlayer serverPlayer) {
        serverPlayer.openMenu(state.getMenuProvider(level, pos));
    }
    return InteractionResult.sidedSuccess(level.isClientSide);
}
```

**注意**：这是实现逻辑的最简单方式，而不是唯一方式。如果你希望方块仅在特定条件下打开菜单，则需要事先将某些数据同步到客户端，以便在条件不满足时返回 `InteractionResult#PASS`​ 或 `#FAIL`​。

---

#### 生物实现

生物通常通过重写 `Mob#mobInteract`​ 来实现菜单。这与方块实现类似，唯一的区别是生物本身应实现 `MenuProvider`​ 以支持旁观模式查看。

```java
public class MyMob extends Mob implements MenuProvider {
    // ...

    @Override
    public InteractionResult mobInteract(Player player, InteractionHand hand) {
        if (!this.level.isClientSide && player instanceof ServerPlayer serverPlayer) {
            serverPlayer.openMenu(this);
        }
        return InteractionResult.sidedSuccess(this.level.isClientSide);
    }
}
```

**注意**：再次强调，这是实现逻辑的最简单方式，而不是唯一方式。

---

希望这些内容对你的开发有所帮助！如果有其他问题，欢迎随时提问。
