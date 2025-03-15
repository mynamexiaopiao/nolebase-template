# 标签（Tags）

### 标签（Tags）

简单来说，标签是同一类型注册对象的列表。它们从数据文件中加载，并可用于成员资格检查。例如，制作木棍时会接受任何组合的木板（带有 `minecraft:planks`​ 标签的物品）。标签通常通过在它们前面加上 `#`​ 来与“常规”对象区分开来（例如 `#minecraft:planks`​，而 `minecraft:oak_planks`​ 则不是标签）。

任何注册表都可以有标签文件——虽然方块和物品是最常见的用例，但其他注册表（如流体、实体类型或伤害类型）也经常使用标签。如果你需要，也可以创建自己的标签。

标签文件位于 `data/<tag_namespace>/tags/<registry_path>/<tag_path>.json`​ 对于 Minecraft 注册表，以及 `data/<tag_namespace>/tags/<registry_namespace>/<registry_path>/<tag_path>.json`​ 对于非 Minecraft 注册表。例如，要修改 `minecraft:planks`​ 物品标签，你需要将标签文件放置在 `data/minecraft/tags/item/planks.json`​。

**注意**
与大多数其他 NeoForge 数据文件不同，NeoForge 添加的标签通常不使用 `neoforge`​ 命名空间，而是使用 `c`​ 命名空间（例如 `c:ingots/gold`​）。这是因为许多开发者同时在多个加载器上开发模组，因此 NeoForge 和 Fabric 模组加载器统一了标签。

有一些例外情况，某些与 NeoForge 系统紧密相关的标签仍然使用 `neoforge`​ 命名空间。例如，许多伤害类型标签就是如此。

覆盖标签文件通常是添加性的，而不是替换性的。这意味着如果两个数据包指定了相同 ID 的标签文件，这两个文件的内容将被合并（除非另有说明）。这种行为使标签与大多数其他数据文件不同，后者通常会替换所有现有值。

### 标签文件格式

标签文件的语法如下：

```json
{
    // 标签的值。
    "values": [
        // 一个值对象。必须指定要添加的对象的 ID，以及是否必需。
        // 如果条目是必需的，但对象不存在，则标签将不会加载。"required" 字段
        // 在技术上是可选的，但如果省略，则条目等同于下面的简写形式。
        {
            "id": "examplemod:example_ingot",
            "required": false
        },
        // 简写形式，等同于 {"id": "minecraft:gold_ingot", "required": true}，即必需的条目。
        "minecraft:gold_ingot",
        // 一个标签对象。通过前面的 # 与常规条目区分。在这种情况下，所有木板
        // 将被视为标签的条目。与普通条目一样，这也可以使用 "id"/"required" 格式。
        // 警告：循环标签依赖将导致数据包无法加载！
        "#minecraft:planks"
    ],
    // 是否在添加自己的条目之前移除所有预先存在的条目（true），还是仅添加自己的条目（false）。
    // 通常应为 false，将此选项设置为 true 主要针对包开发者。
    "replace": false,
    // 一种更精细的方式，用于从标签中移除条目（如果存在）。可选，NeoForge 添加。
    // 条目语法与 "values" 数组中的语法相同。
    "remove": [
        "minecraft:iron_ingot"
    ]
}
```

### 查找和命名标签

当你尝试查找现有标签时，通常建议遵循以下步骤：

1. 查看 Minecraft 的标签，看看你需要的标签是否已经存在。Minecraft 的标签可以在 `BlockTags`​、`ItemTags`​、`EntityTypeTags`​ 等中找到。
2. 如果没有，查看 NeoForge 的标签，看看你需要的标签是否已经存在。NeoForge 的标签可以在 `Tags.Blocks`​、`Tags.Items`​、`Tags.EntityTypes`​ 等中找到。
3. 否则，假设该标签在 Minecraft 或 NeoForge 中未指定，因此你需要创建自己的标签。

在创建自己的标签时，你应该问自己以下问题：

* 这会修改我的模组的行为吗？如果是，标签应放在你的模组的命名空间中。（例如，`my-thing-can-spawn-on-this-block`​ 类型的标签。）
* 其他模组是否也会使用此标签？如果是，标签应放在 `c`​ 命名空间中。（例如，新的金属或宝石标签。）
* 否则，使用你的模组的命名空间。

命名标签本身也有一些约定需要遵循：

* 使用复数形式。例如：`minecraft:planks`​、`c:ingots`​。
* 对同一类型的多个对象使用文件夹，并为每个文件夹创建一个总标签。例如：`c:ingots/iron`​、`c:ingots/gold`​，以及包含两者的 `c:ingots`​。（注意：这是 NeoForge 的约定，Minecraft 对大多数标签不遵循此约定。）

### 使用标签

要在代码中引用标签，你必须创建一个 `TagKey<T>`​，其中 `T`​ 是标签的类型（`Block`​、`Item`​、`EntityType<?>`​ 等），使用注册表键和资源位置：

```java
public static final TagKey<Block> MY_TAG = TagKey.create(
        // 注册表键。注册表的类型必须与标签的泛型类型匹配。
        Registries.BLOCK,
        // 标签的位置。此示例将我们的标签放在 `data/examplemod/tags/blocks/example_tag.json`。
        ResourceLocation.fromNamespaceAndPath("examplemod", "example_tag")
);
```

**警告**
由于 `TagKey`​ 是一个记录（record），它的构造函数是公开的。然而，不应直接使用构造函数，因为这可能会导致各种问题，例如在查找标签条目时。

然后，我们可以使用我们的标签来执行各种操作。让我们从最明显的一个开始：检查一个对象是否在标签中。以下示例假设使用方块标签，但功能对于每种类型的标签都是相同的（除非另有说明）：

```java
// 检查泥土是否在我们的标签中。
boolean isInTag = BuiltInRegistries.BLOCK.getOrCreateTag(MY_TAG).stream().anyMatch(e -> e == Items.DIRT);
```

由于这是一个非常冗长的语句，尤其是在频繁使用时，`BlockState`​ 和 `ItemStack`​——标签系统的两个最常见用户——各自定义了一个 `#is`​ 辅助方法，用法如下：

```java
// 检查 blockState 的方块是否在我们的标签中。
boolean isInBlockTag = blockState.is(MY_TAG);
// 检查 itemStack 的物品是否在我们的标签中。假设存在 MY_ITEM_TAG 作为 TagKey<Item>。
boolean isInItemTag = itemStack.is(MY_ITEM_TAG);
```

如果需要，我们还可以获取标签条目的集合，如下所示：

```java
Set<Block> blocksInTag = BuiltInRegistries.BLOCK.getOrCreateTag(MY_TAG).stream().toSet();
```

出于性能原因，建议将这些集合缓存在字段中，并在标签重新加载时使缓存失效（可以使用 `TagsUpdatedEvent`​ 监听）。可以这样做：

```java
public class MyTagsCacheClass {
    private static Set<Block> blocksInTag = null;

    public static Set<Block> getBlockTagContents() {
        if (blocksInTag == null) {
            // 包装为不可修改的集合，因为我们不应该修改它
            blocksInTag = Collections.unmodifiableSet(BuiltInRegistries.BLOCK.getOrCreateTag(MY_TAG).stream().toSet());
        }
        return blocksInTag;
    }
    
    public static void invalidateCache() {
        blocksInTag = null;
    }
}

// 在事件处理类中
@SubscribeEvent
public static void onTagsUpdated(TagsUpdatedEvent event) {
    MyTagsCacheClass.invalidateCache();
}
```

### 数据生成（Datagen）

与许多其他 JSON 文件一样，标签可以通过数据生成器生成。每种标签都有自己的数据生成基类——一个用于方块标签，一个用于物品标签，等等——因此，我们还需要为每种标签创建一个类。所有这些类都继承自 `TagsProvider<T>`​ 基类，其中 `T`​ 是标签的类型（`Block`​、`Item`​ 等）。下表列出了不同对象的标签提供者类：

|类型|标签提供者类|
| --------------------------| --------------------------------------|
|BannerPattern|BannerPatternTagsProvider|
|Biome|BiomeTagsProvider|
|Block|BlockTagsProvider|
|CatVariant|CatVariantTagsProvider|
|DamageType|DamageTypeTagsProvider|
|Enchantment|EnchantmentTagsProvider|
|EntityType|EntityTypeTagsProvider|
|FlatLevelGeneratorPreset|FlatLevelGeneratorPresetTagsProvider|
|Fluid|FluidTagsProvider|
|GameEvent|GameEventTagsProvider|
|Instrument|InstrumentTagsProvider|
|Item|ItemTagsProvider|
|PaintingVariant|PaintingVariantTagsProvider|
|PoiType|PoiTypeTagsProvider|
|Structure|StructureTagsProvider|
|WorldPreset|WorldPresetTagsProvider|

值得注意的是 `IntrinsicHolderTagsProvider<T>`​ 类，它是 `TagsProvider<T>`​ 的子类，也是 `BlockTagsProvider`​、`ItemTagsProvider`​、`FluidTagsProvider`​、`EntityTypeTagsProvider`​ 和 `GameEventTagsProvider`​ 的公共超类。这些类（为简单起见，从现在起称为“内在提供者”）具有一些额外的生成功能，稍后将概述。

为了举例说明，假设我们要生成方块标签。（所有其他类的工作方式与其各自的标签类型相同。）

```java
public class MyBlockTagsProvider extends BlockTagsProvider {
    // 从 GatherDataEvent 获取参数。
    public MyBlockTagsProvider(PackOutput output, CompletableFuture<HolderLookup.Provider> lookupProvider, ExistingFileHelper existingFileHelper) {
        super(output, lookupProvider, ExampleMod.MOD_ID, existingFileHelper);
    }

    // 在此添加你的标签条目。
    @Override
    protected void addTags(HolderLookup.Provider lookupProvider) {
        // 为我们的标签创建一个标签构建器。这也可以是例如原版或 NeoForge 标签。
        tag(MY_TAG)
                // 添加条目。这是一个可变参数。
                // 非内在提供者必须在此处提供 ResourceKey 而不是实际对象。
                .add(Blocks.DIRT, Blocks.COBBLESTONE)
                // 添加可选的条目，如果不存在则忽略。此示例使用 Botania 的 Pure Daisy。
                // 与 #add 不同，这不是一个可变参数。
                .addOptional(ResourceLocation.fromNamespaceAndPath("botania", "pure_daisy"))
                // 添加一个标签条目。
                .addTag(BlockTags.PLANKS)
                // 添加多个标签条目。这是一个可变参数。
                // 可能会导致未检查的警告，可以安全地忽略。
                .addTags(BlockTags.LOGS, BlockTags.WOODEN_SLABS)
                // 添加一个可选的标签条目，如果不存在则忽略。
                .addOptionalTag(ResourceLocation.fromNamespaceAndPath("c", "ingots/tin"))
                // 添加多个可选的标签条目。这是一个可变参数。
                // 可能会导致未检查的警告，可以安全地忽略。
                .addOptionalTags(ResourceLocation.fromNamespaceAndPath("c", "nuggets/tin"), ResourceLocation.fromNamespaceAndPath("c", "storage_blocks/tin"))
                // 将 replace 属性设置为 true。
                .replace()
                // 将 replace 属性设置回 false。
                .replace(false)
                // 移除条目。这是一个可变参数。接受资源位置、资源键、
                // 标签键，或（仅限内在提供者）直接值。
                // 可能会导致未检查的警告，可以安全地忽略。
                .remove(ResourceLocation.fromNamespaceAndPath("minecraft", "crimson_slab"), ResourceLocation.fromNamespaceAndPath("minecraft", "warped_slab"));
    }
}
```

此示例生成以下标签 JSON：

```json
{
    "values": [
        "minecraft:dirt",
        "minecraft:cobblestone",
        {
            "id": "botania:pure_daisy",
            "required": false
        },
        "#minecraft:planks",
        "#minecraft:logs",
        "#minecraft:wooden_slabs",
        {
            "id": "c:ingots/tin",
            "required": false
        },
        {
            "id": "c:nuggets/tin",
            "required": false
        },
        {
            "id": "c:storage_blocks/tin",
            "required": false
        }
    ],
    "remove": [
        "minecraft:crimson_slab",
        "minecraft:warped_slab"
    ]
}
```

与所有数据提供者一样，将每个标签提供者添加到 `GatherDataEvent`​：

```java
@SubscribeEvent
public static void gatherData(GatherDataEvent event) {
    PackOutput output = generator.getPackOutput();
    CompletableFuture<HolderLookup.Provider> lookupProvider = event.getLookupProvider();
    ExistingFileHelper existingFileHelper = event.getExistingFileHelper();

    // 其他提供者在这里
    event.getGenerator().addProvider(
        event.includeServer(),
        new MyBlockTagsProvider(output, lookupProvider, existingFileHelper)
    );
}
```

​`ItemTagsProvider`​ 有一个额外的辅助方法 `#copy`​。它用于常见的用例，即物品标签镜像方块标签：

```java
// 在 ItemTagsProvider 的 #addTags 方法中，假设两个参数的类型为 TagKey<Block> 和 TagKey<Item>。
copy(EXAMPLE_BLOCK_TAG, EXAMPLE_ITEM_TAG);
```

### 自定义标签提供者

要为自定义注册表创建自定义标签提供者，或者为默认没有标签提供者的原版或 NeoForge 注册表创建自定义标签提供者，你也可以像这样创建自定义标签提供者（以配方类型标签为例）：

```java
public class MyRecipeTypeTagsProvider extends TagsProvider<RecipeType<?>> {
    // 从 GatherDataEvent 获取参数。
    public MyRecipeTypeTagsProvider(PackOutput output, CompletableFuture<HolderLookup.Provider> lookupProvider, ExistingFileHelper existingFileHelper) {
        // 第二个参数是我们为其生成标签的注册表键。
        super(output, Registries.RECIPE_TYPE, lookupProvider, ExampleMod.MOD_ID, existingFileHelper);
    }
    
    @Override
    protected void addTags(HolderLookup.Provider lookupProvider) { /*...*/ }
}
```

如果适用且需要，你还可以扩展 `IntrinsicHolderTagsProvider<T>`​ 而不是 `TagsProvider<T>`​，这允许你直接传递对象而不是它们的资源键。这还需要一个函数参数，该参数返回给定对象的资源键。以属性标签为例：

```java
public class MyAttributeTagsProvider extends TagsProvider<Attribute> {
    // 从 GatherDataEvent 获取参数。
    public MyAttributeTagsProvider(PackOutput output, CompletableFuture<HolderLookup.Provider> lookupProvider, ExistingFileHelper existingFileHelper) {
        super(output,
                Registries.ATTRIBUTE,
                lookupProvider,
                // 一个函数，给定一个 Attribute，返回一个 ResourceKey<Attribute>。
                attribute -> BuiltInRegistries.ATTRIBUTE.getResourceKey(attribute).orElseThrow(),
                ExampleMod.MOD_ID,
                existingFileHelper);
    }

    // 现在可以直接在此处使用属性，而不仅仅是它们的资源键。
    @Override
    protected void addTags(HolderLookup.Provider lookupProvider) { /*...*/ }
}
```

**注意**
`TagsProvider`​ 还暴露了 `#getOrCreateRawBuilder`​ 方法，返回一个 `TagBuilder`​。`TagBuilder`​ 允许将原始 `ResourceLocation`​ 添加到标签中，这在某些场景中可能有用。`TagsProvider.TagAppender<T>`​ 类（由 `TagsProvider#tag`​ 返回）只是 `TagBuilder`​ 的包装器。
