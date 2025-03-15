# 全局战利品修改器（Global Loot Modifiers）

### 全局战利品修改器（Global Loot Modifiers）

全局战利品修改器（简称 GLMs）是一种基于数据的方式来修改掉落物，而无需覆盖大量原版战利品表，或者处理需要与其他模组的战利品表交互的情况（即使不知道加载了哪些模组）。

GLMs 的工作方式是先运行关联的战利品表，然后将 GLM 应用于运行结果。GLMs 是堆叠的，而不是“最后加载者生效”，允许多个模组修改同一个战利品表，这与标签（tags）的行为类似。

要注册一个 GLM，你需要以下四部分内容：

1. ​**​`global_loot_modifiers.json`​**​ **文件**：位于 `data/neoforge/loot_modifiers/global_loot_modifiers.json`​（不在你的模组命名空间中）。该文件告诉 NeoForge 应用哪些修改器以及应用顺序。
2. **表示战利品修改器的 JSON 文件**：该文件包含修改器的所有数据，允许数据包调整效果。文件位于 `data/<namespace>/loot_modifiers/<path>.json`​。
3. **实现** **​`IGlobalLootModifier`​**​ **或扩展** **​`LootModifier`​**​ **的类**：该类包含使修改器工作的代码。
4. **用于编码和解码战利品修改器类的** **​`MapCodec`​**​：通常作为战利品修改器类中的 `public static final`​ 字段实现。

---

### `global_loot_modifiers.json`​

​`global_loot_modifiers.json`​ 文件告诉 NeoForge 应该对哪些战利品表应用修改器。文件可能包含两个键：

* ​**​`entries`​**​：应加载的修改器列表。指定的 `ResourceLocation`​ 指向 `data/<namespace>/loot_modifiers/<path>.json`​ 中的关联条目。此列表是有序的，意味着修改器将按指定顺序应用，这在模组兼容性问题中有时很重要。
* ​**​`replace`​**​：表示修改器是否应替换旧的修改器（`true`​）或只是添加到现有列表中（`false`​）。这与标签中的 `replace`​ 键类似，但与标签不同，此键是必需的。通常，模组开发者应始终使用 `false`​；`true`​ 的能力主要面向模组包或数据包开发者。

示例用法：

```json
{
    "replace": false, // 必须存在
    "entries": [
        // 表示位于 data/examplemod/loot_modifiers/example_glm_1.json 的战利品修改器
        "examplemod:example_glm_1",
        "examplemod:example_glm_2"
        // ...
    ]
}
```

---

### 战利品修改器 JSON 文件

该文件包含与修改器相关的所有值，例如应用几率、要添加的物品等。建议尽可能避免硬编码值，以便数据包制作者可以调整平衡。战利品修改器必须至少包含两个字段，具体取决于情况：

* ​**​`type`​**​：战利品修改器的注册名称。
* ​**​`conditions`​**​：此修改器激活的战利品条件列表。
* 其他属性可能是必需的或可选的，具体取决于使用的编解码器。

**提示**：GLMs 的一个常见用例是向特定战利品表添加额外战利品。为此，可以使用 `neoforge:loot_table_id`​ 条件。

示例用法：

```json
{
    // 这是战利品修改器的注册名称
    "type": "examplemod:my_loot_modifier",
    "conditions": [
        // 战利品条件
    ],
    // 编解码器指定的额外属性
    "field1": "somestring",
    "field2": 10,
    "field3": "minecraft:dirt"
}
```

---

### `IGlobalLootModifier`​ 和 `LootModifier`​

为了将战利品修改器实际应用于战利品表，必须指定一个 `IGlobalLootModifier`​ 实现。在大多数情况下，你会希望使用 `LootModifier`​ 子类，它为你处理条件等。以下是一个扩展 `LootModifier`​ 的示例：

```java
// 我们不能使用 record，因为 record 不能扩展其他类。
public class MyLootModifier extends LootModifier {
    // 编解码器见下文。
    public static final MapCodec<MyLootModifier> CODEC = ...;
    // 我们的额外属性。
    private final String field1;
    private final int field2;
    private final Item field3;

    // 第一个构造函数参数是条件列表，其余是我们的额外属性。
    public MyLootModifier(LootItemCondition[] conditions, String field1, int field2, Item field3) {
        super(conditions);
        this.field1 = field1;
        this.field2 = field2;
        this.field3 = field3;
    }

    // 返回我们的编解码器。
    @Override
    public MapCodec<? extends IGlobalLootModifier> codec() {
        return CODEC;
    }

    // 这是实际应用修改器的地方。如果需要，可以使用额外属性。
    // 参数是现有的战利品和战利品上下文。
    @Override
    protected ObjectArrayList<ItemStack> doApply(ObjectArrayList<ItemStack> generatedLoot, LootContext context) {
        // 在这里将你的物品添加到 generatedLoot。
        return generatedLoot;
    }
}
```

**注意**：从修改器返回的战利品列表会按注册顺序传递给其他修改器。因此，修改后的战利品可能会被其他战利品修改器进一步修改。

---

### 战利品修改器编解码器

为了告诉游戏我们的战利品修改器的存在，我们必须定义并注册一个编解码器。以下是一个包含三个字段的示例：

```java
public static final MapCodec<MyLootModifier> CODEC = RecordCodecBuilder.mapCodec(inst -> 
        // LootModifier#codecStart 添加 conditions 字段。
        LootModifier.codecStart(inst).and(inst.group(
                Codec.STRING.fieldOf("field1").forGetter(e -> e.field1),
                Codec.INT.fieldOf("field2").forGetter(e -> e.field2),
                BuiltInRegistries.ITEM.byNameCodec().fieldOf("field3").forGetter(e -> e.field3)
        )).apply(inst, MyLootModifier::new)
);
```

然后，我们将编解码器注册到注册表中：

```java
public static final DeferredRegister<MapCodec<? extends IGlobalLootModifier>> GLOBAL_LOOT_MODIFIER_SERIALIZERS =
        DeferredRegister.create(NeoForgeRegistries.Keys.GLOBAL_LOOT_MODIFIER_SERIALIZERS, ExampleMod.MOD_ID);

public static final Supplier<MapCodec<MyLootModifier>> MY_LOOT_MODIFIER =
        GLOBAL_LOOT_MODIFIER_SERIALIZERS.register("my_loot_modifier", () -> MyLootModifier.CODEC);
```

---

### 内置战利品修改器

NeoForge 提供了一个现成的战利品修改器供你使用：

#### `neoforge:add_table`​

此战利品修改器会运行第二个战利品表，并将其结果添加到应用修改器的战利品表中。

```json
{
    "type": "neoforge:add_table",
    "conditions": [], // 所需的战利品条件
    "table": "minecraft:chests/abandoned_mineshaft" // 要运行的第二个战利品表
}
```

---

### 数据生成（Datagen）

GLMs 可以通过子类化 `GlobalLootModifierProvider`​ 来生成数据：

```java
public class MyGlobalLootModifierProvider extends GlobalLootModifierProvider {
    // 从 GatherDataEvent 获取 PackOutput。
    public MyGlobalLootModifierProvider(PackOutput output) {
        super(output, ExampleMod.MOD_ID);
    }

    @Override
    protected void start() {
        // 调用 #add 添加新的 GLM。这还会在 global_loot_modifiers.json 中添加相应的条目。
        add(
                // 修改器的名称。这将是文件名。
                "my_loot_modifier_instance",
                // 要添加的战利品修改器。为了示例，我们添加一个天气战利品条件。
                new MyLootModifier(new LootItemCondition[] {
                        WeatherCheck.weather().setRaining(true).build()
                }, "somestring", 10, Items.DIRT);
                // 数据加载条件列表。注意，这些与修改器本身的战利品条件无关。
                // 为了示例，我们添加一个模组加载条件。
                // 还有一个接受可变参数条件的 #add 重载。
                List.of(new ModLoadedCondition("create"))
        );
    }
}
```

像所有数据提供器一样，你必须将提供器注册到 `GatherDataEvent`​：

```java
@SubscribeEvent
public static void onGatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(event.includeServer(), MyGlobalLootModifierProvider::new);
}
```
