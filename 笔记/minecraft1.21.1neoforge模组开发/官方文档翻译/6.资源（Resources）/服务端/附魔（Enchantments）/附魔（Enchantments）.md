# 附魔（Enchantments）

### 附魔（Enchantments）

附魔是可以应用于工具和其他物品的特殊效果。从 1.21 版本开始，附魔以数据组件（Data Components）的形式存储在物品上，并通过 JSON 文件定义，由所谓的附魔效果组件（enchantment effect components）组成。在游戏中，特定物品的附魔信息存储在 `DataComponentTypes.ENCHANTMENT`​ 组件中，具体表现为 `ItemEnchantment`​ 实例。

要添加一个新的附魔，可以在你的命名空间的附魔数据包子文件夹中创建一个 JSON 文件。例如，要创建一个名为 `examplemod:example_enchant`​ 的附魔，可以创建一个文件 `data/examplemod/enchantment/example_enchantment.json`​。

---

### 附魔 JSON 格式

```json
{
    // 用于游戏内附魔名称的文本组件。
    // 可以是翻译键或字面字符串。
    // 如果使用翻译键，请确保在语言文件中翻译它！
    "description": {
        "translate": "enchantment.examplemod.enchant_name"
    },
    
    // 该附魔可以应用于哪些物品。
    // 可以是单个物品 ID，例如 "minecraft:trident"，
    // 或物品 ID 列表，例如 ["examplemod:red_sword", "examplemod:blue_sword"]，
    // 或物品标签，例如 "#examplemod:enchantable/enchant_name"。
    // 注意：这不会使附魔出现在附魔台的这些物品上。
    "supported_items": "#examplemod:enchantable/enchant_name",

    // （可选）该附魔在附魔台中出现在哪些物品上。
    // 可以是单个物品、物品列表或物品标签。
    // 如果未指定，则与 `supported_items` 相同。
    "primary_items": [
        "examplemod:item_a",
        "examplemod:item_b"
    ],

    // （可选）与该附魔不兼容的其他附魔。
    // 可以是单个附魔 ID，例如 "minecraft:sharpness"，
    // 或附魔 ID 列表，例如 ["minecraft:sharpness", "minecraft:fire_aspect"]，
    // 或附魔标签，例如 "#examplemod:exclusive_to_enchant_name"。
    // 不兼容的附魔不会通过原版机制添加到同一物品上。
    "exclusive_set": "#examplemod:exclusive_to_enchant_name",
    
    // 该附魔出现在附魔台中的概率。
    // 范围为 [1, 1024]。
    "weight": 6,
    
    // 该附魔允许的最大等级。
    // 范围为 [1, 255]。
    "max_level": 3,
    
    // 该附魔的最大成本，以“附魔能量”为单位。
    // 这与玩家需要满足的附魔等级阈值相关，但并不等同。
    // 详见下文。
    // 实际成本将在此值与 min_cost 之间。
    "max_cost": {
        "base": 45,
        "per_level_above_first": 9
    },
    
    // 指定该附魔的最小成本；其他同上。
    "min_cost": {
        "base": 2,
        "per_level_above_first": 8
    },

    // 该附魔在铁砧中修复物品时增加的成本（以等级为单位）。成本乘以附魔等级。
    // 如果物品具有 DataComponentTypes.STORED_ENCHANTMENTS 组件，则成本减半。在原版中，这仅适用于附魔书。
    // 范围为 [1, ∞)。
    "anvil_cost": 2,
    
    // （可选）该附魔提供效果的槽位组列表。
    // 槽位组定义为 EquipmentSlotGroup 枚举的可能值之一。
    // 在原版中，这些值为：`any`、`hand`、`mainhand`、`offhand`、`armor`、`feet`、`legs`、`chest`、`head` 和 `body`。
    "slots": [
        "mainhand"
    ],

    // 该附魔提供的效果，作为附魔效果组件的映射（见下文）。
    "effects": {
        "examplemod:custom_effect": [
            {
                "effect": {
                    "type": "minecraft:add",
                    "value": {
                        "type": "minecraft:linear",
                        "base": 1,
                        "per_level_above_first": 1
                    }
                }
            }
        ]
    }
}
```

---

### 附魔成本和等级

​`max_cost`​ 和 `min_cost`​ 字段指定了创建该附魔所需的附魔能量的边界。然而，实际使用这些值的过程有些复杂。

首先，附魔台会考虑周围方块的 `IBlockExtension#getEnchantPowerBonus()`​ 返回值。然后，调用 `EnchantmentHelper#getEnchantmentCost`​ 来为每个槽位推导出一个“基础等级”。这个等级在游戏中显示为附魔菜单中附魔旁边的绿色数字。对于每个附魔，基础等级会根据物品的附魔能力值（`IItemExtension#getEnchantmentValue()`​ 的返回值）进行两次随机调整，公式如下：

```
(调整后的等级) = (基础等级) + random.nextInt(e / 4 + 1) + random.nextInt(e / 4 + 1)
```

其中 `e`​ 是附魔能力值。

调整后的等级会再随机上下浮动 15%，最终用于选择附魔。该等级必须落在附魔的成本范围内才能被选中。

实际上，这意味着附魔定义中的成本值可能高于 30，有时甚至远高于 30。例如，对于附魔能力值为 10 的物品，附魔台可能生成成本高达 `1.15 * (30 + 2 * (10 / 4) + 1) = 40`​ 的附魔。

---

### 附魔效果组件

附魔效果组件是特殊注册的数据组件，用于决定附魔的功能。组件的类型定义了其效果，而其包含的数据用于调整或修改该效果。例如，`minecraft:damage`​ 组件根据其数据修改武器造成的伤害。

原版定义了多种内置的附魔效果组件，用于实现所有原版附魔。

---

### 自定义附魔效果组件

自定义附魔效果组件的逻辑必须完全由其创建者实现。首先，你需要定义一个类或记录来保存实现效果所需的信息。例如，我们创建一个示例记录类 `Increment`​：

```java
// 定义一个示例数据记录。
public record Increment(int value) {
    public static final Codec<Increment> CODEC = RecordCodecBuilder.create(instance ->
            instance.group(
                    Codec.INT.fieldOf("value").forGetter(Increment::value)
            ).apply(instance, Increment::new)
    );

    public int add(int x) {
        return value() + x;
    }
}
```

附魔效果组件类型必须注册到 `BuiltInRegistries.ENCHANTMENT_EFFECT_COMPONENT_TYPE`​，它接受一个 `DataComponentType<?>`​。例如，你可以注册一个可以存储 `Increment`​ 对象的附魔效果组件，如下所示：

```java
// 在某个注册类中
public static final DeferredRegister<DataComponentType<?>> ENCHANTMENT_COMPONENT_TYPES = DeferredRegister.create(BuiltInRegistries.ENCHANTMENT_EFFECT_COMPONENT_TYPE, "examplemod");

public static final DeferredHolder<DataComponentType<?>, DataComponentType<Increment>>> INCREMENT =
    ENCHANTMENT_COMPONENT_TYPES.register("increment",
        () -> DataComponentType.<Increment>builder()
            .persistent(Increment.CODEC)
            .build());
```

现在，我们可以实现一些游戏逻辑，利用该组件来修改整数值：

```java
// 在某个游戏逻辑中，`itemStack` 可用。
// `INCREMENT` 是上面定义的附魔组件类型持有者。
// `value` 是一个整数。
AtomicInteger atomicValue = new AtomicInteger(value);

EnchantmentHelper.runIterationOnItem(stack, (enchantmentHolder, enchantLevel) -> {
    // 从附魔持有者中获取 Increment 实例（如果是其他附魔，则为 null）。
    Increment increment = enchantmentHolder.value().effects().get(INCREMENT.get());

    // 如果该附魔有 Increment 组件，则使用它。
    if(increment != null){
        atomicValue.set(increment.add(atomicValue.get()));
    }
});

int modifiedValue = atomicValue.get();
// 在游戏逻辑中使用修改后的值。
```

首先，我们调用 `EnchantmentHelper#runIterationOnItem`​ 的一个重载。该函数接受一个 `EnchantmentHelper.EnchantmentVisitor`​，这是一个函数式接口，接受附魔及其等级，并在给定物品的所有附魔上调用（本质上是一个 `BiConsumer<Enchantment, Integer>`​）。

为了实际执行调整，使用提供的 `Increment#add`​ 方法。由于这是在 lambda 表达式中，我们需要使用可以原子更新的类型（如 `AtomicInteger`​）来修改该值。这也允许多个 `INCREMENT`​ 组件在同一物品上运行并叠加效果，就像原版中的情况一样。

---

### 条件效果（ConditionalEffect）

将类型包装在 `ConditionalEffect<?>`​ 中，可以使附魔效果组件根据给定的 `LootContext`​ 选择性地生效。

​`ConditionalEffect`​ 提供了 `ConditionalEffect#matches(LootContext context)`​，用于根据其内部的 `Optional<LootItemCondition>`​ 返回是否应允许效果运行，并处理其 `LootItemCondition`​ 的序列化和反序列化。

原版还提供了一个辅助方法 `Enchantment#applyEffects()`​，用于进一步简化检查这些条件的过程。该方法接受一个 `List<ConditionalEffect<T>>`​，评估条件，并对每个满足条件的 `ConditionalEffect`​ 中的 `T`​ 运行 `Consumer<T>`​。由于许多原版附魔效果组件被定义为 `List<ConditionalEffect<?>>`​，因此可以直接插入辅助方法，如下所示：

```java
// `enchant` 是一个 Enchantment 实例。
// `lootContext` 是一个 LootContext 实例。
Enchantment.applyEffects(
    enchant.getEffects(EnchantmentEffectComponents.KNOCKBACK), // 或其他 List<ConditionalEffect<T>>
    lootContext,
    (effectData) -> // 使用 effectData（在此示例中为 EnchantmentValueEffect）。
);
```

注册一个自定义的 `ConditionalEffect`​ 包装的附魔效果组件类型可以如下进行：

```java
public static final DeferredHolder<DataComponentType<?>, DataComponentType<ConditionalEffect<Increment>>> CONDITIONAL_INCREMENT =
    ENCHANTMENT_COMPONENT_TYPES.register("conditional_increment",
        () -> DataComponentType.ConditionalEffect<Increment>builder()
            // 所需的 LootContextParamSet 取决于附魔的预期功能。
            // 可能是 ENCHANTED_DAMAGE、ENCHANTED_ITEM、ENCHANTED_LOCATION、ENCHANTED_ENTITY 或 HIT_BLOCK 之一，
            // 因为这些都将附魔等级带入上下文（以及指示的其他信息）。
            .persistent(ConditionalEffect.codec(Increment.CODEC, LootContextParamSets.ENCHANTED_DAMAGE))
            .build());
```

​`ConditionalEffect.codec`​ 的参数是 `ConditionalEffect<T>`​ 的编解码器，后跟某个 `LootContextParamSets`​ 条目。

---

### 附魔数据生成

附魔 JSON 文件可以通过数据生成系统自动创建，方法是将 `RegistrySetBuilder`​ 传递给 `DatapackBuiltInEntriesProvider`​。JSON 文件将放置在 `<project root>/src/generated/data/<modid>/enchantment/<path>.json`​。

有关 `RegistrySetBuilder`​ 和 `DatapackBuiltinEntriesProvider`​ 的更多信息，请参阅数据生成相关文章。

---

### 数据生成示例

```json
{
    // 附魔的铁砧成本。
    "anvil_cost": 2,

    // 指定附魔名称的文本组件。
    "description": "Example Enchantment",

    // 与该附魔关联的效果组件及其值的映射。
    "effects": {
        // <效果组件>
    },

    // 附魔的最大成本。
    "max_cost": {
        "base": 4,
        "per_level_above_first": 2
    },

    // 该附魔的最大等级。
    "max_level": 3,

    // 附魔的最小成本。
    "min_cost": {
        "base": 3,
        "per_level_above_first": 1
    },

    // 该附魔提供效果的 EquipmentSlotGroup 别名列表。
    "slots": [
        "any"
    ],

    // 该附魔可以使用铁砧应用于的物品集合。
    "supported_items": /* <支持的物品列表> */,

    // 该附魔的权重。
    "weight": 30
}
```
