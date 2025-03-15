# 容器（Containers）

### 容器（Containers）

方块实体的一个常见用途是存储某种物品。Minecraft 中的一些重要方块，如熔炉或箱子，都使用方块实体来实现这一目的。为了在某个东西上存储物品，Minecraft 使用了**容器（Containers）** 。

`Container`接口定义了诸如`#getItem`、`#setItem`和`#removeItem`等方法，用于查询和更新容器。由于它是一个接口，它实际上并不包含一个后备列表或其他数据结构，这取决于实现系统。

因此，容器不仅可以实现在方块实体上，还可以实现在任何其他类上。显著的例子包括实体物品栏，以及常见的模组物品，如背包。

**警告**：NeoForge 提供了`ItemStackHandler`类作为`Container`的替代品，在许多地方应该优先使用它，因为它允许与其他容器/`ItemStackHandler`进行更清晰的交互。本文存在的主要原因是供参考原版代码，或者如果你在多个加载器上开发模组。如果可能，始终在你自己的代码中使用`ItemStackHandler`！相关文档正在编写中。

---

### 基本容器实现

容器可以以任何你喜欢的方式实现，只要你满足规定的方法（就像 Java 中的任何其他接口一样）。然而，通常使用固定长度的`NonNullList<ItemStack>`作为后备结构。单槽容器也可以简单地使用一个`ItemStack`字段。

例如，一个大小为 27 个槽（一个箱子）的`Container`的基本实现可能如下所示：

```java
public class MyContainer implements Container {
    private final NonNullList<ItemStack> items = NonNullList.withSize(
            // 列表的大小，即我们容器中的槽数。
            27,
            // 用于替代普通列表中 null 的默认值。
            ItemStack.EMPTY
    );

    // 我们容器中的槽数。
    @Override
    public int getContainerSize() {
        return 27;
    }

    // 容器是否被视为空。
    @Override
    public boolean isEmpty() {
        return this.items.stream().allMatch(ItemStack::isEmpty);
    }

    // 返回指定槽中的物品堆叠。
    @Override
    public ItemStack getItem(int slot) {
        return this.items.get(slot);
    }

    // 当对容器进行更改时调用此方法，例如添加、修改或移除物品堆叠时。
    // 例如，你可以在这里调用 BlockEntity#setChanged。
    @Override
    public void setChanged() {

    }

    // 从给定槽中移除指定数量的物品，返回刚刚移除的堆叠。
    // 我们在这里委托给 ContainerHelper，它会按预期为我们执行此操作。
    // 但是，我们必须手动调用 #setChanged。
    @Override
    public ItemStack removeItem(int slot, int amount) {
        ItemStack stack = ContainerHelper.removeItem(this.items, slot, amount);
        this.setChanged();
        return stack;
    }

    // 从指定槽中移除所有物品，返回刚刚移除的堆叠。
    // 我们再次委托给 ContainerHelper，并且我们再次需要手动调用 #setChanged。
    @Override
    public ItemStack removeItemNoUpdate(int slot) {
        ItemStack stack = ContainerHelper.takeItem(this.items, slot);
        this.setChanged();
        return stack;
    }

    // 在给定槽中设置给定的物品堆叠。首先限制为容器的最大堆叠大小。
    @Override
    public void setItem(int slot, ItemStack stack) {
        stack.limitSize(this.getMaxStackSize(stack));
        this.items.set(slot, stack);
        this.setChanged();
    }

    // 容器是否对给定玩家仍然“有效”。例如，箱子和类似方块会检查玩家是否仍然在方块的一定距离内。
    @Override
    public boolean stillValid(Player player) {
        return true;
    }

    // 清除内部存储，将所有槽再次设置为空。
    @Override
    public void clearContent() {
        items.clear();
        this.setChanged();
    }
}
```

---

### SimpleContainer

`SimpleContainer`类是容器的一个基本实现，带有一些额外的功能，例如能够添加`ContainerListener`。如果你需要一个没有特殊要求的容器实现，可以使用它。

---

### BaseContainerBlockEntity

`BaseContainerBlockEntity`类是 Minecraft 中许多重要方块实体的基类，例如箱子和类似箱子的方块、各种熔炉类型、漏斗、发射器、投掷器和酿造台等。

除了`Container`，它还实现了`MenuProvider`和`Nameable`接口：

- `Nameable`定义了一些与设置（自定义）名称相关的方法，除了许多方块实体外，`Entity`等类也实现了它。这使用了`Component`系统。
- `MenuProvider`定义了`#createMenu`方法，允许从容器构造`AbstractContainerMenu`。这意味着如果你想要一个没有关联 GUI 的容器（例如唱片机），使用此类是不可取的。

`BaseContainerBlockEntity`将所有我们通常对`NonNullList<ItemStack>`的调用通过`#getItems`和`#setItems`两个方法捆绑在一起，大大减少了我们需要编写的样板代码。`BaseContainerBlockEntity`的示例实现可能如下所示：

```java
public class MyBlockEntity extends BaseContainerBlockEntity {
    // 容器大小。当然，这可以是任何你想要的值。
    public static final int SIZE = 9;
    // 我们的物品堆叠列表。由于 #setItems 的存在，这不是 final 的。
    private NonNullList<ItemStack> items = NonNullList.withSize(SIZE, ItemStack.EMPTY);

    // 构造函数，和以前一样。
    public MyBlockEntity(BlockPos pos, BlockState blockState) {
        super(MY_BLOCK_ENTITY.get(), pos, blockState);
    }

    // 容器大小，和以前一样。
    @Override
    public int getContainerSize() {
        return SIZE;
    }

    // 我们的物品堆叠列表的 getter。
    @Override
    protected NonNullList<ItemStack> getItems() {
        return items;
    }

    // 我们的物品堆叠列表的 setter。
    @Override
    protected void setItems(NonNullList<ItemStack> items) {
        this.items = items;
    }

    // 菜单的显示名称。别忘了添加翻译！
    @Override
    protected Component getDefaultName() {
        return Component.translatable("container.examplemod.myblockentity");
    }

    // 从此容器创建的菜单。有关返回内容，请参阅下文。
    @Override
    protected AbstractContainerMenu createMenu(int containerId, Inventory inventory) {
        return null;
    }
}
```

请记住，这个类同时是一个`BlockEntity`和一个`Container`。这意味着你可以使用此类作为方块实体的超类，以获得一个具有预实现容器的功能方块实体。

---

### WorldlyContainer

`WorldlyContainer`是`Container`的子接口，允许通过`Direction`访问给定容器的槽。它主要用于方块实体，这些方块实体只将其容器的一部分暴露给特定的一侧。例如，这可以用于一个机器，它从一侧输出并从所有其他侧输入，或者反之亦然。该接口的简单实现可能如下所示：

```java
// 参见上面的 BaseContainerBlockEntity 方法。如果需要，你当然可以直接扩展 BlockEntity 并自己实现 Container。
public class MyBlockEntity extends BaseContainerBlockEntity implements WorldlyContainer {
    // 其他内容
    
    // 假设槽 0 是我们的输出槽，槽 1-8 是我们的输入槽。
    // 进一步假设我们向顶部输出并从所有其他侧输入。
    private static final int[] OUTPUTS = new int[]{0};
    private static final int[] INPUTS = new int[]{1, 2, 3, 4, 5, 6, 7, 8};

    // 根据传递的 Direction 返回暴露的槽索引数组。
    @Override
    public int[] getSlotsForFace(Direction side) {
        return side == Direction.UP ? OUTPUTS : INPUTS;
    }

    // 是否可以通过给定的一侧在给定的槽中放置物品。
    // 对于我们的示例，我们仅在不是从上方输入且索引在 [1, 8] 范围内时返回 true。
    @Override
    public boolean canPlaceItemThroughFace(int index, ItemStack itemStack, @Nullable Direction direction) {
        return direction != Direction.UP && index > 0 && index < 9;
    }

    // 是否可以从给定的一侧和给定的槽中取出物品。
    // 对于我们的示例，我们仅在从上方拉取且槽索引为 0 时返回 true。
    @Override
    public boolean canTakeItemThroughFace(int index, ItemStack stack, Direction direction) {
        return direction == Direction.UP && index == 0;
    }
}
```

---

### 使用容器

现在我们已经创建了容器，让我们来使用它们！

由于`Container`和`BlockEntity`之间有相当大的重叠，如果可能，最好通过将方块实体强制转换为`Container`来检索容器：

```java
if (blockEntity instanceof Container container) {
    // 对容器进行操作
}
```

然后，容器可以使用我们之前提到的方法，例如：

```java
// 获取容器中的第一个物品。
ItemStack stack = container.getItem(0);

// 将容器中的第一个物品设置为泥土。
container.setItem(0, new ItemStack(Items.DIRT));

// 从第三个槽中移除最多 16 个物品。
container.removeItem(2, 16);
```

**警告**：如果尝试访问超出容器大小的槽，容器可能会抛出异常。或者，它们可能会返回`ItemStack.EMPTY`，例如`SimpleContainer`。

---

### 物品堆叠上的容器

到目前为止，我们主要讨论了`BlockEntity`上的容器。然而，它们也可以使用`minecraft:container`数据组件应用于`ItemStack`：

```java
// 我们在这里使用 SimpleContainer 作为超类，这样我们就不必自己重新实现物品处理逻辑。
// 由于 SimpleContainer 的实现细节，如果多个方可以同时访问容器，这可能会导致竞争条件，
// 所以我们假设我们的模组不允许这种情况。
// 如果需要，你当然可以使用不同的容器实现（或自己实现 Container）。
public class MyBackpackContainer extends SimpleContainer {
    // 此容器对应的物品堆叠。在构造函数中传入并设置。
    private final ItemStack stack;
    
    public MyBackpackContainer(ItemStack stack) {
        // 我们调用 super 并传入我们想要的容器大小。
        super(27);
        // 设置 stack 字段。
        this.stack = stack;
        // 我们从数据组件（如果存在）加载容器内容，数据组件由 ItemContainerContents 类表示。
        // 如果不存在，我们使用 ItemContainerContents.EMPTY。
        ItemContainerContents contents = stack.getOrDefault(DataComponents.CONTAINER, ItemContainerContents.EMPTY);
        // 将数据组件内容复制到我们的物品堆叠列表中。
        contents.copyInto(this.getItems());
    }

    // 当内容更改时，我们在堆叠上保存数据组件。
    @Override
    public void setChanged() {
        super.setChanged();
        this.stack.set(DataComponents.CONTAINER, ItemContainerContents.fromItems(this.getItems()));
    }
}
```

瞧，你已经创建了一个基于物品的容器！调用`new MyBackpackContainer(stack)`来为菜单或其他用例创建容器。

**警告**：请注意，直接与容器交互的菜单在修改`ItemStack`时必须`#copy()`它们，否则会破坏数据组件的不变性契约。为此，NeoForge 为你提供了`StackCopySlot`类。

---

### 实体上的容器

实体上的容器很棘手：无法普遍确定一个实体是否有容器。这完全取决于你处理的实体，因此可能需要大量的特殊处理。

如果你自己创建一个实体，没有什么能阻止你直接在它上面实现`Container`，但请注意你将无法使用`SimpleContainer`等超类（因为`Entity`是超类）。

---

### 生物上的容器

生物不实现`Container`，但它们实现了`EquipmentUser`接口（以及其他接口）。该接口定义了`#setItemSlot(EquipmentSlot, ItemStack)`、`#getItemBySlot(EquipmentSlot)`和`#setDropChance(EquipmentSlot, float)`方法。虽然与`Container`在代码上没有关联，但功能非常相似：我们将槽（在这种情况下是装备槽）与`ItemStack`关联。

与`Container`最显著的区别是没有类似列表的顺序（尽管`Mob`在后台使用`NonNullList<ItemStack>`）。访问不是通过槽索引进行的，而是通过七个`EquipmentSlot`枚举值进行的：`MAINHAND`、`OFFHAND`、`FEET`、`LEGS`、`CHEST`、`HEAD`和`BODY`（其中`BODY`用于马和狗的盔甲）。

与生物的“槽”交互的示例如下所示：

```java
// 获取 HEAD（头盔）槽中的物品堆叠。
ItemStack helmet = mob.getItemBySlot(EquipmentSlot.HEAD);

// 将基岩放入生物的 FEET（靴子）槽中。
mob.setItemSlot(EquipmentSlot.FEET, new ItemStack(Items.BEDROCK));

// 启用基岩在被杀死时始终掉落。
mob.setDropChance(EquipmentSlot.FEET, 1f);
```

---

### InventoryCarrier

`InventoryCarrier`是一些生物（如村民）实现的接口。它声明了一个`#getInventory`方法，返回一个`SimpleContainer`。此接口用于需要实际物品栏而不是`EquipmentUser`提供的装备槽的非玩家实体。

---

### 玩家上的容器（玩家物品栏）

玩家的物品栏通过`Inventory`类实现，该类实现了`Container`以及前面提到的`Nameable`接口。然后，该`Inventory`的实例作为名为`inventory`的字段存储在`Player`上，可通过`Player#getInventory`访问。可以像任何其他容器一样与物品栏交互。

物品栏内容存储在三个`public final NonNullList<ItemStack>`中：

- `items`列表覆盖了 36 个主物品栏槽，包括九个快捷栏槽（索引 0-8）。
- `armor`列表是一个长度为 4 的列表，包含`FEET`、`LEGS`、`CHEST`和`HEAD`的盔甲，按此顺序。此列表使用`EquipmentSlot`访问器，类似于生物（见上文）。
- `offhand`列表仅包含副手槽，即长度为 1。

当遍历物品栏内容时，建议先遍历`items`，然后遍历`armor`，最后遍历`offhand`，以与原版行为一致。
