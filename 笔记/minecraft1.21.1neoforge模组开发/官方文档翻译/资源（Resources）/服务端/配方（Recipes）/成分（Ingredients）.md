# 成分（Ingredients）

### 成分（Ingredients）

成分用于配方中，以检查给定的 `ItemStack`​ 是否是配方的有效输入。为此，`Ingredient`​ 实现了 `Predicate<ItemStack>`​，可以调用 `#test`​ 来确认给定的 `ItemStack`​ 是否匹配该成分。

不幸的是，`Ingredient`​ 的许多内部实现较为混乱。NeoForge 通过尽可能忽略 `Ingredient`​ 类来解决这个问题，转而引入 `ICustomIngredient`​ 接口用于自定义成分。这并不是对常规 `Ingredient`​ 的直接替代，但我们可以分别使用 `ICustomIngredient#toVanilla`​ 和 `Ingredient#getCustomIngredient`​ 在两者之间进行转换。

### 内置成分类型

获取成分的最简单方法是使用 `Ingredient#of`​ 辅助方法。存在以下几种变体：

* ​`Ingredient.of()`​ 返回一个空成分。
* ​`Ingredient.of(Blocks.IRON_BLOCK, Items.GOLD_BLOCK)`​ 返回一个接受铁块或金块的成分。参数是 `ItemLike`​ 的可变参数，这意味着可以使用任意数量的方块和物品。
* ​`Ingredient.of(new ItemStack(Items.DIAMOND_SWORD))`​ 返回一个接受物品堆叠的成分。请注意，数量和数据组件会被忽略。
* ​`Ingredient.of(Stream.of(new ItemStack(Items.DIAMOND_SWORD)))`​ 返回一个接受物品堆叠的成分。与上一种方法类似，但使用 `Stream<ItemStack>`​，适用于你已经获得此类流的情况。
* ​`Ingredient.of(ItemTags.WOODEN_SLABS)`​ 返回一个接受指定标签中任何物品的成分，例如任何木质台阶。

此外，NeoForge 还添加了一些额外的成分：

* ​`BlockTagIngredient.of(BlockTags.CONVERTABLE_TO_MUD)`​ 返回一个类似于 `Ingredient.of()`​ 标签变体的成分，但使用方块标签。这适用于你希望使用物品标签但只有方块标签可用的情况（例如 `minecraft:convertable_to_mud`​）。
* ​`CompoundIngredient.of(Ingredient.of(Items.DIRT))`​ 返回一个包含子成分的成分，子成分通过构造函数传入（可变参数）。如果任何子成分匹配，则该成分匹配。
* ​`DataComponentIngredient.of(true, new ItemStack(Items.DIAMOND_SWORD))`​ 返回一个除了匹配物品外还匹配数据组件的成分。布尔参数表示严格匹配（`true`​）或部分匹配（`false`​）。严格匹配意味着数据组件必须完全匹配，而部分匹配意味着数据组件必须匹配，但其他数据组件也可以存在。`#of`​ 的其他重载允许指定多个物品或提供其他选项。
* ​`DifferenceIngredient.of(Ingredient.of(ItemTags.PLANKS), Ingredient.of(ItemTags.NON_FLAMMABLE_WOOD))`​ 返回一个匹配第一个成分中但不匹配第二个成分的所有内容的成分。给定的示例仅匹配可以燃烧的木板（即除绯红木板、诡异木板和模组下界木板外的所有木板）。
* ​`IntersectionIngredient.of(Ingredient.of(ItemTags.PLANKS), Ingredient.of(ItemTags.NON_FLAMMABLE_WOOD))`​ 返回一个匹配同时匹配两个子成分的所有内容的成分。给定的示例仅匹配不能燃烧的木板（即绯红木板、诡异木板和模组下界木板）。

请记住，NeoForge 提供的成分类型是 `ICustomIngredient`​，在将其用于原版上下文之前必须调用 `#toVanilla`​，如本文开头所述。

### 自定义成分类型

模组开发者可以通过 `ICustomIngredient`​ 系统添加自定义成分类型。为了举例说明，让我们创建一个接受物品标签和附魔到最低等级映射的附魔物品成分：

```java
public class MinEnchantedIngredient implements ICustomIngredient {
    private final TagKey<Item> tag;
    private final Map<Holder<Enchantment>, Integer> enchantments;
    // 用于序列化成分的编解码器。
    public static final MapCodec<MinEnchantedIngredient> CODEC = RecordCodecBuilder.mapCodec(inst -> inst.group(
            TagKey.codec(Registries.ITEM).fieldOf("tag").forGetter(e -> e.tag),
            Codec.unboundedMap(Enchantment.CODEC, Codec.INT)
                    .optionalFieldOf("enchantments", Map.of())
                    .forGetter(e -> e.enchantments)
    ).apply(inst, MinEnchantedIngredient::new));
    // 从常规编解码器创建流编解码器。在某些情况下，从头定义一个新的流编解码器可能更有意义。
    public static final StreamCodec<RegistryFriendlyByteBuf, MinEnchantedIngredient> STREAM_CODEC =
            ByteBufCodecs.fromCodecWithRegistries(CODEC.codec());

    // 允许传入预先存在的附魔到等级的映射。
    public MinEnchantedIngredient(TagKey<Item> tag, Map<Holder<Enchantment>, Integer> enchantments) {
        this.tag = tag;
        this.enchantments = enchantments;
    }

    public MinEnchantedIngredient(TagKey<Item> tag) {
        this(tag, new HashMap<>());
    }

    // 使此方法可链式调用以便于使用。
    public MinEnchantedIngredient with(Holder<Enchantment> enchantment, int level) {
        enchantments.put(enchantment, level);
        return this;
    }

    // 检查传入的 ItemStack 是否匹配我们的成分，通过验证物品是否在标签中并测试所有所需附魔是否至少达到所需等级。
    @Override
    public boolean test(ItemStack stack) {
        return stack.is(tag) && enchantments.keySet()
                .stream()
                .allMatch(ench -> stack.getEnchantmentLevel(ench) >= enchantments.get(ench));
    }

    // 确定此成分是否执行 NBT 或数据组件匹配（false）或不执行（true）。
    // 还确定是否使用流编解码器进行同步，稍后会详细介绍。
    // 我们在堆叠上查询附魔，因此我们的成分不是简单的。
    @Override
    public boolean isSimple() {
        return false;
    }

    // 返回匹配此成分的物品流。主要用于显示目的。
    // 这里有一些好的实践需要遵循：
    // - 始终至少包含一个物品堆叠，以防止意外识别为空。
    // - 至少包含每个接受的物品一次。
    // - 如果 #isSimple 为 true，则此方法应精确并包含每个匹配的物品堆叠。
    //   如果不是，则应尽可能精确，但不需要非常准确。
    // 在我们的例子中，我们使用标签中的所有物品，每个物品都带有所需的附魔。
    @Override
    public Stream<ItemStack> getItems() {
        // 获取物品堆叠列表，标签中的每个物品一个堆叠。
        List<ItemStack> stacks = BuiltInRegistries.ITEM
                .getOrCreateTag(tag)
                .stream()
                .map(ItemStack::new)
                .toList();
        // 为所有堆叠附上所有附魔。
        for (ItemStack stack : stacks) {
            enchantments
                    .keySet()
                    .forEach(ench -> stack.enchant(ench, enchantments.get(ench)));
        }
        // 返回列表的流形式。
        return stacks.stream();
    }
}
```

自定义成分是一个注册表，因此我们必须注册我们的成分。我们使用 NeoForge 提供的 `IngredientType`​ 类来注册，该类基本上是 `MapCodec`​ 和可选的 `StreamCodec`​ 的包装器。

```java
public static final DeferredRegister<IngredientType<?>> INGREDIENT_TYPES =
        DeferredRegister.create(NeoForgeRegistries.Keys.INGREDIENT_TYPE, ExampleMod.MOD_ID);

public static final Supplier<IngredientType<MinEnchantedIngredient>> MIN_ENCHANTED =
        INGREDIENT_TYPES.register("min_enchanted",
                // 流编解码器参数是可选的，如果未指定流编解码器，则会使用 ByteBufCodecs#fromCodec 或 #fromCodecWithRegistries 从编解码器创建流编解码器。
                () -> new IngredientType<>(MinEnchantedIngredient.CODEC, MinEnchantedIngredient.STREAM_CODEC));
```

当我们完成这些后，我们还需要在我们的成分类中重写 `#getType`​：

```java
public class MinEnchantedIngredient implements ICustomIngredient {
    // 其他内容

    @Override    
    public IngredientType<?> getType() {
        return MIN_ENCHANTED;
    }
}
```

就这样！我们的成分类型已经准备好使用了。

### JSON 表示

由于原版成分非常有限，而 NeoForge 引入了全新的注册表来处理它们，因此也值得看看内置成分和我们自己的成分在 JSON 中的样子。

指定类型的成分通常被认为是非原版的。例如：

```json
{
    "type": "neoforge:block_tag",
    "tag": "minecraft:convertable_to_mud"
}
```

或者使用我们自己的成分的另一个示例：

```json
{
    "type": "examplemod:min_enchanted",
    "tag": "c:swords",
    "enchantments": {
        "minecraft:sharpness": 4
    }
}
```

如果未指定类型，则我们有一个原版成分。原版成分可以指定两个属性之一：`item`​ 或 `tag`​。

原版物品成分的示例：

```json
{
    "item": "minecraft:dirt"
}
```

原版标签成分的示例：

```json
{
    "tag": "c:ingots"
}
```
