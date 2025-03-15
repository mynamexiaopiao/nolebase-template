# 配方（Recipes）

### 配方（Recipes）

配方是 Minecraft 世界中将一组对象转换为其他对象的一种方式。尽管 Minecraft 仅将此系统用于物品转换，但该系统的构建方式允许任何类型的对象（方块、实体等）进行转换。几乎所有配方都使用配方数据文件；除非另有说明，本文中的“配方”均指数据驱动的配方。

配方数据文件位于 `data/<命名空间>/recipe/<路径>.json`​。例如，配方 `minecraft:diamond_block`​ 位于 `data/minecraft/recipe/diamond_block.json`​。

---

### 术语

* **配方 JSON 或配方文件**：由 `RecipeManager`​ 加载和存储的 JSON 文件。它包含配方类型、输入和输出等信息，以及其他附加信息（例如处理时间）。
* **配方（Recipe）** ：代码中表示所有 JSON 字段的类，包含匹配逻辑（“此输入是否匹配配方？”）和其他属性。
* **配方输入（RecipeInput）** ：提供配方输入的类型。有多个子类，例如 `CraftingInput`​ 或 `SingleRecipeInput`​（用于熔炉等）。
* **配方成分（Ingredient）** ：配方的单个输入（而 `RecipeInput`​ 通常表示一组输入以检查配方的成分）。成分是一个非常强大的系统，因此在单独的文章中进行了概述。
* **配方管理器（RecipeManager）** ：服务器上的单例字段，保存所有加载的配方。
* **配方序列化器（RecipeSerializer）** ：基本上是 `MapCodec`​ 和 `StreamCodec`​ 的包装器，用于序列化。
* **配方类型（RecipeType）** ：配方的注册类型等效项。主要用于按类型查找配方。一般来说，不同的合成容器应使用不同的 `RecipeType`​。例如，`minecraft:crafting`​ 配方类型涵盖了 `minecraft:crafting_shaped`​ 和 `minecraft:crafting_shapeless`​ 配方序列化器，以及特殊的合成序列化器。
* **配方进度（Recipe Advancement）** ：负责在配方书中解锁配方的进度。它们不是必需的，通常被玩家忽略，转而使用配方查看器模组，但配方数据提供器会为你生成它们，因此建议直接使用。
* **配方构建器（RecipeBuilder）** ：在数据生成期间用于创建 JSON 配方。
* **配方工厂（Recipe Factory）** ：用于从 `RecipeBuilder`​ 创建 `Recipe`​ 的方法引用。它可以是构造函数的引用、静态构建器方法，或者是专门为此目的创建的函数式接口（通常命名为 `Factory`​）。

---

### JSON 规范

配方文件的内容因所选类型而异。所有配方文件共有的属性是 `type`​ 和 `neoforge:conditions`​：

```json
{
    // 配方类型。映射到配方序列化器注册表中的条目。
    "type": "minecraft:crafting_shaped",
    // 数据加载条件列表。可选，NeoForge 新增。有关更多信息，请参阅上面的文章。
    "neoforge:conditions": [ /*...*/ ]
}
```

Minecraft 提供的所有类型的完整列表可以在 [内置配方类型](#built-in-recipe-types) 文章中找到。模组也可以定义自己的配方类型。

---

### 使用配方

配方通过 `RecipeManager`​ 类加载、存储和获取，而 `RecipeManager`​ 又通过 `ServerLevel#getRecipeManager`​ 或（如果没有可用的 `ServerLevel`​）`ServerLifecycleHooks.getCurrentServer()#getRecipeManager`​ 获取。请注意，虽然客户端拥有完整的 `RecipeManager`​ 副本用于显示目的，但配方逻辑应始终在服务器上运行以避免同步问题。

获取配方的最简单方法是通过 ID：

```java
RecipeManager recipes = serverLevel.getRecipeManager();
// RecipeHolder<?> 是配方 ID 和配方本身的记录。
Optional<RecipeHolder<?>> optional = recipes.byId(ResourceLocation.withDefaultNamespace("diamond_block"));
optional.map(RecipeHolder::value).ifPresent(recipe -> {
    // 在这里对配方执行任何操作。请注意，配方可能是任何类型。
});
```

更实用的方法是构建 `RecipeInput`​ 并尝试获取匹配的配方。在此示例中，我们将创建一个包含一个钻石块的 `CraftingInput`​，使用 `CraftingInput#of`​。这将创建一个无形状的输入，而有形状的输入将使用 `CraftingInput#ofPositioned`​，其他输入将使用其他 `RecipeInput`​（例如，熔炉配方通常使用 `new SingleRecipeInput`​）。

```java
RecipeManager recipes = serverLevel.getRecipeManager();
// 构建 RecipeInput，如配方所需。例如，为合成配方构建 CraftingInput。
// 参数分别为宽度、高度和物品。
CraftingInput input = CraftingInput.of(1, 1, List.of(Items.DIAMOND_BLOCK));
// 配方持有者的泛型通配符应扩展 Recipe<CraftingInput>。
// 这允许在以后进行更多的类型安全。
Optional<RecipeHolder<? extends Recipe<CraftingInput>>> optional = recipes.getRecipeFor(
    // 要获取的配方类型。在我们的例子中，我们使用合成类型。
    RecipeTypes.CRAFTING,
    // 我们的配方输入。
    input,
    // 我们的世界上下文。
    serverLevel
);
// 这将返回钻石块 -> 9 钻石的配方（除非数据包更改了该配方）。
optional.map(RecipeHolder::value).ifPresent(recipe -> {
    // 在这里执行任何操作。请注意，配方现在是 Recipe<CraftingInput> 而不是 Recipe<?>。
});
```

或者，你也可以获取一个可能为空的匹配配方的列表，这在可以合理假设多个配方匹配的情况下特别有用：

```java
RecipeManager recipes = serverLevel.getRecipeManager();
CraftingInput input = CraftingInput.of(1, 1, List.of(Items.DIAMOND_BLOCK));
// 这些不是 Optionals，可以直接使用。但是，列表可能为空，表示没有匹配的配方。
List<RecipeHolder<? extends Recipe<CraftingInput>>> list = recipes.getRecipesFor(
    // 与上述相同的参数。
    RecipeTypes.CRAFTING, input, serverLevel
);
```

一旦我们有了正确的配方输入，我们还希望获取配方输出。这是通过调用 `Recipe#assemble`​ 完成的：

```java
RecipeManager recipes = serverLevel.getRecipeManager();
CraftingInput input = CraftingInput.of(...);
Optional<RecipeHolder<? extends Recipe<CraftingInput>>> optional = recipes.getRecipeFor(...);
// 使用 ItemStack.EMPTY 作为回退。
ItemStack result = optional
    .map(RecipeHolder::value)
    .map(e -> e.assemble(input, serverLevel.registryAccess()))
    .orElse(ItemStack.EMPTY);
```

如果需要，还可以迭代某个类型的所有配方。这是这样完成的：

```java
RecipeManager recipes = serverLevel.getRecipeManager();
// 像以前一样，传递所需的配方类型。
List<RecipeHolder<?>> list = recipes.getAllRecipesFor(RecipeTypes.CRAFTING);
```

---

### 其他配方机制

一些机制在原版中通常被认为是配方，但在代码中以不同的方式实现。这通常是由于遗留原因，或者因为“配方”是从其他数据（例如标签）构建的。

**警告**：配方查看器模组通常不会识别这些配方。必须手动添加对这些模组的支持，请参阅相应模组的文档以获取更多信息。

#### 铁砧配方

铁砧有两个输入槽和一个输出槽。唯一的原版用例是工具修复、组合和重命名，由于每个用例都需要特殊处理，因此没有提供配方文件。但是，可以使用 `AnvilUpdateEvent`​ 构建该系统。此事件允许获取输入（左侧输入槽）和材料（右侧输入槽），并允许设置输出物品堆栈，以及经验成本和消耗的材料数量。还可以通过取消事件来完全阻止该过程。

```java
// 此示例允许用一整堆泥土修复石镐，消耗一半的堆叠，花费 3 级经验。
@SubscribeEvent
public static void onAnvilUpdate(AnvilUpdateEvent event) {
    ItemStack left = event.getLeft();
    ItemStack right = event.getRight();
    if (left.is(Items.STONE_PICKAXE) && right.is(Items.DIRT) && right.getCount() >= 64) {
        event.setOutput(Items.STONE_PICKAXE);
        event.setMaterialCost(32);
        event.setCost(3);
    }
}
```

#### 酿造

请参阅 [Mob Effects &amp; Potions](#mob-effects-and-potions) 文章中的酿造章节。

---

### 自定义配方

要添加自定义配方，我们至少需要三样东西：一个 `Recipe`​、一个 `RecipeType`​ 和一个 `RecipeSerializer`​。根据你实现的内容，如果无法重用现有的子类，则可能还需要自定义的 `RecipeInput`​。

为了举例说明并突出许多不同的功能，我们将实现一个配方驱动的机制，要求你右键单击世界中的某个 `BlockState`​ 并使用某个物品，破坏 `BlockState`​ 并掉落结果物品。

#### 配方输入

首先定义我们要放入配方的内容。重要的是要理解，配方输入表示玩家当前正在使用的实际输入。因此，我们在这里不使用标签或成分，而是使用我们可用的实际物品堆栈和方块状态。

```java
// 我们的输入是一个 BlockState 和一个 ItemStack。
public record RightClickBlockInput(BlockState state, ItemStack stack) implements RecipeInput {
    // 从特定槽位获取物品的方法。我们有一个堆栈，没有槽位的概念，因此我们假设
    // 槽位 0 持有我们的物品，并在任何其他槽位上抛出异常。（取自 SingleRecipeInput#getItem。）
    @Override
    public ItemStack getItem(int slot) {
        if (slot != 0) throw new IllegalArgumentException("No item for index " + slot);
        return this.stack();
    }

    // 我们的输入所需的槽位大小。同样，我们没有槽位的概念，因此我们只返回 1，
    // 因为我们涉及一个物品堆栈。涉及多个物品的输入应在此处返回实际计数。
    @Override
    public int size() {
        return 1;
    }
}
```

配方输入不需要以任何方式注册或序列化，因为它们是按需创建的。并非总是需要创建自己的输入，原版的输入（`CraftingInput`​、`SingleRecipeInput`​ 和 `SmithingRecipeInput`​）适用于许多用例。

此外，NeoForge 提供了 `RecipeWrapper`​ 输入，它包装了 `#getItem`​ 和 `#size`​ 调用，以处理传递给构造函数的 `IItemHandler`​。基本上，这意味着任何基于网格的库存（例如箱子）都可以通过将其包装在 `RecipeWrapper`​ 中用作配方输入。

#### 配方类

现在我们有了输入，让我们来看看配方本身。这是保存我们的配方数据的地方，还处理匹配和返回配方结果。因此，它通常是自定义配方中最长的类。

```java
// Recipe<T> 的泛型参数是我们上面的 RightClickBlockInput。
public class RightClickBlockRecipe implements Recipe<RightClickBlockInput> {
    // 我们的配方数据的代码表示。这基本上可以是任何你想要的东西。
    // 常见的包括某种处理时间整数或经验奖励。
    // 请注意，我们现在使用成分而不是物品堆栈作为输入。
    private final BlockState inputState;
    private final Ingredient inputItem;
    private final ItemStack result;

    // 添加一个设置所有属性的构造函数。
    public RightClickBlockRecipe(BlockState inputState, Ingredient inputItem, ItemStack result) {
        this.inputState = inputState;
        this.inputItem = inputItem;
        this.result = result;
    }

    // 我们的成分列表。如果没有成分，则不需要重写
    // （默认实现在此处返回一个空列表）。缓存较大的列表是有意义的。
    @Override
    public NonNullList<Ingredient> getIngredients() {
        NonNullList<Ingredient> list = NonNullList.create();
        list.add(this.inputItem);
        return list;
    }

    // 基于网格的配方应返回其配方是否可以适应给定的尺寸。
    // 我们没有网格，因此我们只返回是否可以放置任何物品。
    @Override
    public boolean canCraftInDimensions(int width, int height) {
        return width * height >= 1;
    }

    // 检查给定的输入是否匹配此配方。第一个参数匹配泛型。
    // 我们检查我们的方块状态和物品堆栈，只有在两者都匹配时才返回 true。
    @Override
    public boolean matches(RightClickBlockInput input, Level level) {
        return this.inputState == input.state() && this.inputItem.test(input.stack());
    }

    // 在此处返回结果的不可修改版本。此方法的结果主要用于
    // 配方书，通常也被 JEI 和其他配方查看器使用。
    @Override
    public ItemStack getResultItem(HolderLookup.Provider registries) {
        return this.result;
    }

    // 在此处返回配方结果，基于给定的输入。第一个参数匹配泛型。
    // 重要提示：如果使用现有结果，请始终调用 .copy()！如果不这样做，事情可能会并且将会出错，
    // 因为结果每个配方存在一次，但每次合成配方时都会创建组装的堆栈。
    @Override
    public ItemStack assemble(RightClickBlockInput input, HolderLookup.Provider registries) {
        return this.result.copy();
    }

    // 此示例概述了最重要的方法。还有许多其他方法需要重写。
    // 查看 Recipe 的类定义以查看所有方法。
}
```

#### 配方类型

接下来是我们的配方类型。这相当简单，因为除了名称之外，配方类型没有其他数据。它们是配方系统中两个注册部分之一，因此与所有其他注册表一样，我们创建一个 `DeferredRegister`​ 并注册到其中：

```java
public static final DeferredRegister<RecipeType<?>> RECIPE_TYPES =
        DeferredRegister.create(Registries.RECIPE_TYPE, ExampleMod.MOD_ID);

public static final Supplier<RecipeType<RightClickBlockRecipe>> RIGHT_CLICK_BLOCK =
        RECIPE_TYPES.register(
                "right_click_block",
                // 由于泛型是泛型，我们需要限定泛型。
                () -> RecipeType.<RightClickBlockRecipe>simple(ResourceLocation.fromNamespaceAndPath(ExampleMod.MOD_ID, "right_click_block"))
        );
```

注册配方类型后，我们必须在配方中重写 `#getType`​，如下所示：

```java
public class RightClickBlockRecipe implements Recipe<RightClickBlockInput> {
    // 其他内容

    @Override
    public RecipeType<?> getType() {
        return RIGHT_CLICK_BLOCK.get();
    }
}
```

#### 配方序列化器

配方序列化器提供两个编解码器，一个 `MapCodec`​ 和一个 `StreamCodec`​，分别用于从/到 JSON 和从/到网络的序列化。本节不会深入讨论编解码器的工作原理，有关更多信息，请参阅 [Map Codecs](#map-codecs) 和 [Stream Codecs](#stream-codecs)。

由于配方序列化器可能变得相当大，原版将它们移动到单独的类中。建议但不要求遵循此做法——较小的序列化器通常在配方类的字段中定义为匿名类。为了遵循良好的做法，我们将创建一个单独的类来保存我们的编解码器：

```java
// 泛型参数是我们的配方类。
// 注意：这假设简单的 RightClickBlockRecipe#getInputState、#getInputItem 和 #getResult getters
// 可用，这些在上面的代码中被省略了。
public class RightClickBlockRecipeSerializer implements RecipeSerializer<RightClickBlockRecipe> {
    public static final MapCodec<RightClickBlockRecipe> CODEC = RecordCodecBuilder.mapCodec(inst -> inst.group(
            BlockState.CODEC.fieldOf("state").forGetter(RightClickBlockRecipe::getInputState),
            Ingredient.CODEC.fieldOf("ingredient").forGetter(RightClickBlockRecipe::getInputItem),
            ItemStack.CODEC.fieldOf("result").forGetter(RightClickBlockRecipe::getResult)
    ).apply(inst, RightClickBlockRecipe::new));
    public static final StreamCodec<RegistryFriendlyByteBuf, RightClickBlockRecipe> STREAM_CODEC =
            StreamCodec.composite(
                    ByteBufCodecs.idMapper(Block.BLOCK_STATE_REGISTRY), RightClickBlockRecipe::getInputState,
                    Ingredient.CONTENTS_STREAM_CODEC, RightClickBlockRecipe::getInputItem,
                    ItemStack.STREAM_CODEC, RightClickBlockRecipe::getResult,
                    RightClickBlockRecipe::new
            );

    // 返回我们的 MapCodec。
    @Override
    public MapCodec<RightClickBlockRecipe> codec() {
        return CODEC;
    }

    // 返回我们的 StreamCodec。
    @Override
    public StreamCodec<RegistryFriendlyByteBuf, RightClickBlockRecipe> streamCodec() {
        return STREAM_CODEC;
    }
}
```

与类型一样，我们注册我们的序列化器：

```java
public static final DeferredRegister<RecipeSerializer<?>> RECIPE_SERIALIZERS =
        DeferredRegister.create(Registries.RECIPE_SERIALIZER, ExampleMod.MOD_ID);

public static final Supplier<RecipeSerializer<RightClickBlockRecipe>> RIGHT_CLICK_BLOCK =
        RECIPE_SERIALIZERS.register("right_click_block", RightClickBlockRecipeSerializer::new);
```

同样，我们还必须在配方中重写 `#getSerializer`​，如下所示：

```java
public class RightClickBlockRecipe implements Recipe<RightClickBlockInput> {
    // 其他内容

    @Override
    public RecipeSerializer<?> getSerializer() {
        return RIGHT_CLICK_BLOCK.get();
    }
}
```

#### 合成机制

现在配方的所有部分都已完成，你可以制作一些配方 JSON（请参阅数据生成部分），然后查询配方管理器以获取你的配方，如上所述。然后你对配方执行的操作取决于你。一个常见的用例是一个可以处理你的配方的机器，将活动配方存储为字段。

在我们的例子中，我们希望在右键单击方块时应用配方。我们将使用事件处理程序来实现这一点。请记住，这是一个示例实现，你可以以任何方式修改它（只要在服务器上运行）。

```java
@SubscribeEvent
public static void useItemOnBlock(UseItemOnBlockEvent event) {
    // 如果我们不在事件的方块阶段，则跳过。有关详细信息，请参阅事件的 javadocs。
    if (event.getUsePhase() != UseItemOnBlockEvent.UsePhase.BLOCK) return;
    // 获取我们需要的参数。
    UseOnContext context = event.getUseOnContext();
    Level level = context.getLevel();
    BlockPos pos = context.getClickedPos();
    BlockState blockState = level.getBlockState(pos);
    ItemStack itemStack = context.getItemInHand();
    RecipeManager recipes = level.getRecipeManager();
    // 创建一个输入并查询配方。
    RightClickBlockInput input = new RightClickBlockInput(blockState, itemStack);
    Optional<RecipeHolder<? extends Recipe<CraftingInput>>> optional = recipes.getRecipeFor(
            // 配方类型。
            RIGHT_CLICK_BLOCK,
            input,
            level
    );
    ItemStack result = optional
            .map(RecipeHolder::value)
            .map(e -> e.assemble(input, level.registryAccess()))
            .orElse(ItemStack.EMPTY);
    // 如果有结果，则破坏方块并在世界中掉落结果。
    if (!result.isEmpty()) {
        level.removeBlock(pos, false);
        // 如果世界不是服务器世界，则不生成实体。
        if (!level.isClientSide()) {
            ItemEntity entity = new ItemEntity(level,
                    // pos 的中心。
                    pos.getX() + 0.5, pos.getY() + 0.5, pos.getZ() + 0.5,
                    result);
            level.addFreshEntity(entity);
        }
        // 取消事件以停止交互管道。
        event.cancelWithResult(ItemInteractionResult.sidedSuccess(level.isClientSide));
    }
}
```

#### 扩展合成网格大小

​`ShapedRecipePattern`​ 类负责保存形状合成配方的内存表示，其硬编码限制为 3x3 槽位，阻碍了想要添加更大的合成表同时重用原版形状合成配方类型的模组。为了解决这个问题，NeoForge 修补了一个静态方法 `ShapedRecipePattern#setCraftingSize(int width, int height)`​，允许增加限制。它应该在 `FMLCommonSetupEvent`​ 期间调用。最大的值在这里获胜，因此例如，如果一个模组添加了 4x6 的合成表，另一个模组添加了 6x5 的合成表，则结果值为 6x6。

**危险**：`ShapedRecipePattern#setCraftingSize`​ 不是线程安全的。它必须包装在 `event#enqueueWork`​ 调用中。

---

### 数据生成

像大多数其他 JSON 文件一样，配方可以生成数据。对于配方，我们希望扩展 `RecipeProvider`​ 类并重写 `#buildRecipes`​：

```java
public class MyRecipeProvider extends RecipeProvider {
    // 从 GatherDataEvent 获取参数。
    public MyRecipeProvider(PackOutput output, CompletableFuture<HolderLookup.Provider> lookupProvider) {
        super(output, registries);
    }

    @Override
    protected void buildRecipes(RecipeOutput output) {
        // 在这里添加你的配方。
    }
}
```

值得注意的是 `#buildRecipes`​ 的 `RecipeOutput`​ 参数。Minecraft 使用此对象自动为你生成配方进度。除此之外，NeoForge 将条件支持注入到 `RecipeOutput`​ 中，可以通过 `#withConditions`​ 调用。

配方本身通常通过 `RecipeBuilder`​ 的子类添加。列出所有原版配方构建器超出了本文的范围（它们在 [内置配方类型](#built-in-recipe-types) 文章中进行了说明），但创建自己的构建器如下所述。

像所有其他数据提供器一样，配方提供器必须像这样注册到 `GatherDataEvent`​：

```java
@SubscribeEvent
public static void gatherData(GatherDataEvent event) {
    DataGenerator generator = event.getGenerator();
    PackOutput output = generator.getPackOutput();
    CompletableFuture<HolderLookup.Provider> lookupProvider = event.getLookupProvider();

    // 其他提供器
    generator.addProvider(
            event.includeServer(),
            new MyRecipeProvider(output, lookupProvider)
    );
}
```

配方提供器还添加了常见场景的帮助程序，例如 `twoByTwoPacker`​（用于 2x2 方块配方）、`threeByThreePacker`​（用于 3x3 方块配方）或 `nineBlockStorageRecipes`​（用于 3x3 方块配方和 1 方块到 9 物品的配方）。

#### 自定义配方的数据生成

要为你的自定义配方序列化器创建配方构建器，你需要实现 `RecipeBuilder`​ 及其方法。一个常见的实现，部分复制自原版，如下所示：

```java
// 此类是抽象的，因为有很多每个配方序列化器的逻辑。
// 它用于展示所有（原版）配方构建器的共同部分。
public abstract class SimpleRecipeBuilder implements RecipeBuilder {
    // 使字段受保护，以便我们的子类可以使用它们。
    protected final ItemStack result;
    protected final Map<String, Criterion<?>> criteria = new LinkedHashMap<>();
    @Nullable
    protected String group;

    // 构造函数通常接受结果物品堆栈。
    // 或者，也可以使用静态构建器方法。
    public SimpleRecipeBuilder(ItemStack result) {
        this.result = result;
    }

    // 此方法为配方进度添加一个条件。
    @Override
    public SimpleRecipeBuilder unlockedBy(String name, Criterion<?> criterion) {
        this.criteria.put(name, criterion);
        return this;
    }

    // 此方法添加一个配方书组。如果你不想使用配方书组，
    // 删除 this.group 字段并使此方法无操作（即返回 this）。
    @Override
    public SimpleRecipeBuilder group(@Nullable String group) {
        this.group = group;
        return this;
    }

    // 原版需要一个 Item 而不是 ItemStack。你仍然可以并且应该使用 ItemStack
    // 来序列化配方。
    @Override
    public Item getResult() {
        return this.result.getItem();
    }
}
```

因此，我们有了配方构建器的基础。现在，在我们继续配方序列化器依赖的部分之前，我们应该首先考虑如何制作我们的配方工厂。在我们的例子中，直接使用构造函数是有意义的。在其他情况下，使用静态助手或小型函数式接口是可行的方法。如果你使用一个构建器来处理多个配方类，这一点尤其重要。

利用 `RightClickBlockRecipe::new`​ 作为我们的配方工厂，并重用上面的 `SimpleRecipeBuilder`​ 类，我们可以为 `RightClickBlockRecipes`​ 创建以下配方构建器：

```java
public class RightClickBlockRecipeBuilder extends SimpleRecipeBuilder {
    private final BlockState inputState;
    private final Ingredient inputItem;

    // 由于我们每个输入都有一个，我们将它们传递给构造函数。
    // 具有某种成分列表的配方序列化器的构建器通常会
    // 初始化一个空列表，并具有 #addIngredient 或类似方法。
    public RightClickBlockRecipeBuilder(ItemStack result, BlockState inputState, Ingredient inputItem) {
        super(result);
        this.inputState = inputState;
        this.inputItem = inputItem;
    }

    // 使用给定的 RecipeOutput 和 id 保存配方。此方法在 RecipeBuilder 接口中定义。
    @Override
    public void save(RecipeOutput output, ResourceLocation id) {
        // 构建进度。
        Advancement.Builder advancement = output.advancement()
                .addCriterion("has_the_recipe", RecipeUnlockedTrigger.unlocked(id))
                .rewards(AdvancementRewards.Builder.recipe(id))
                .requirements(AdvancementRequirements.Strategy.OR);
        this.criteria.forEach(advancement::addCriterion);
        // 我们的工厂参数是结果、方块状态和成分。
        RightClickBlockRecipe recipe = new RightClickBlockRecipe(this.inputState, this.inputItem, this.result);
        // 将 id、配方和配方进度传递给 RecipeOutput。
        output.accept(id, recipe, advancement.build(id.withPrefix("recipes/")));
    }
}
```

现在，在数据生成期间，你可以像任何其他配方构建器一样调用你的配方构建器：

```java
@Override
protected void buildRecipes(RecipeOutput output) {
    new RightClickRecipeBuilder(
            // 我们的构造函数参数。此示例添加了流行的泥土 -> 钻石转换。
            new ItemStack(Items.DIAMOND),
            Blocks.DIRT.defaultBlockState(),
            Ingredient.of(Items.APPLE)
    )
            .unlockedBy("has_apple", has(Items.APPLE))
            .save(output);
    // 其他配方构建器
}
```

**注意**：也可以将 `SimpleRecipeBuilder`​ 合并到 `RightClickBlockRecipeBuilder`​（或你自己的配方构建器）中，特别是如果你只有一个或两个配方构建器。这里的抽象用于展示构建器的哪些部分是配方依赖的，哪些不是。
