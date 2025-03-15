# 战利品表（loot table）

### 战利品表（Loot Tables）

战利品表是用于定义随机掉落物品的数据文件。战利品表可以通过“掷骰”来生成一个（可能为空的）物品堆栈列表。生成的结果依赖于（伪）随机性。战利品表位于 `data/<mod_id>/loot_table/<name>.json`​。例如，泥土方块使用的战利品表 `minecraft:blocks/dirt`​ 位于 `data/minecraft/loot_table/blocks/dirt.json`​。

Minecraft 在游戏中的多个地方使用战利品表，包括方块掉落、实体掉落、宝箱战利品、钓鱼战利品等。战利品表的引用方式取决于上下文：

* **方块**：默认情况下，每个方块都会关联一个战利品表，位于 `<block_namespace>:blocks/<block_name>`​。可以通过在方块的 `Properties`​ 中调用 `#noLootTable`​ 来禁用战利品表，这样方块将不会掉落任何物品。这通常用于空气类或技术类方块。
* **实体**：默认情况下，所有不属于 `MobCategory.MISC`​ 类别的 `LivingEntity`​ 子类都会关联一个战利品表，位于 `<entity_namespace>:entities/<entity_name>`​。如果你直接继承 `LivingEntity`​，可以通过重写 `#getLootTable`​ 来更改战利品表；如果继承 `Mob`​ 或其子类，可以重写 `#getDefaultLootTable`​。例如，绵羊会根据羊毛颜色使用不同的战利品表。
* **宝箱**：结构中的宝箱在其方块实体数据中指定战利品表。Minecraft 将所有宝箱战利品表存储在 `minecraft:chests/<chest_name>`​ 中；建议模组遵循此惯例，但不是必须的。
* **村民礼物**：村民在袭击后可能向玩家投掷的礼物战利品表定义在 `neoforge:raid_hero_gifts`​ 数据映射中。
* **其他战利品表**：例如钓鱼战利品表，在需要时通过 `level.getServer().reloadableRegistries().getLootTable(lootTableId)`​ 获取。所有原版战利品表的位置可以在 `BuiltInLootTables`​ 中找到。

**警告**：通常只为属于你模组的内容创建战利品表。修改现有战利品表时，应使用全局战利品修改器（GLMs）。

由于战利品表系统的复杂性，战利品表由多个子系统组成，每个子系统都有不同的用途。

---

### 战利品条目（Loot Entry）

战利品条目（或战利品池条目）是单个战利品元素，代码中由抽象类 `LootPoolEntryContainer`​ 表示。它可以指定一个或多个掉落的物品。

原版提供了 8 种不同的战利品条目类型。通过共同的 `LootPoolEntryContainer`​ 超类，所有条目都具有以下属性：

* **weight**：权重值，默认为 1。用于某些物品比其他物品更常见的情况。例如，给定两个战利品条目，一个权重为 3，另一个为 1，则第一个条目被选中的概率为 75%，第二个为 25%。
* **quality**：品质值，默认为 0。如果非零，则此值会乘以幸运值（在战利品上下文中设置）并加到权重上。
* **conditions**：应用于此条目的战利品条件列表。如果任一条件失败，该条目将被视为不存在。
* **functions**：应用于此条目输出的战利品函数列表。

战利品条目通常分为两类：单例条目（由 `LootPoolSingletonContainer`​ 超类表示）和复合条目（由 `CompositeEntryBase`​ 超类表示），其中复合条目由多个单例条目组成。Minecraft 提供了以下单例条目类型：

* **minecraft:empty**：空条目，表示没有物品。通过 `EmptyLootItem#emptyItem`​ 创建。
* **minecraft:item**：单个物品条目，掉落指定物品。通过 `LootItem#lootTableItem`​ 创建。
* **minecraft:tag**：标签条目，掉落指定标签中的所有物品。根据 `expand`​ 属性的值，有两种变体。如果 `expand`​ 为 `true`​，则为标签中的每个物品生成一个条目；否则，使用一个条目掉落所有物品。通过 `TagEntry#tagContents`​ 或 `TagEntry#expandTag`​ 创建。
* **minecraft:dynamic**：动态掉落条目，引用无法预先指定的掉落。通过 `DynamicLoot#dynamicEntry`​ 创建。
* **minecraft:loot_table**：引用另一个战利品表的条目，将另一个战利品表的结果作为单个条目添加。通过 `NestedLootTable#lootTableReference`​ 或 `NestedLootTable#inlineLootTable`​ 创建。

以下复合条目类型由 Minecraft 提供：

* **minecraft:group**：包含其他战利品条目的列表，按顺序运行。通过 `EntryGroup#list`​ 或 `LootPoolSingletonContainer.Builder#append`​ 创建。
* **minecraft:sequence**：类似于 `minecraft:group`​，但当一个子条目失败时停止运行。通过 `SequentialEntry#sequential`​ 或 `LootPoolSingletonContainer.Builder#then`​ 创建。
* **minecraft:alternatives**：与 `minecraft:sequence`​ 相反，当一个子条目成功时停止运行。通过 `AlternativesEntry#alternatives`​ 或 `LootPoolSingletonContainer.Builder#otherwise`​ 创建。

模组开发者还可以定义自定义战利品条目类型。

---

### 战利品池（Loot Pool）

战利品池本质上是一个战利品条目的列表。战利品表可以包含多个战利品池，每个池独立运行。

战利品池可能包含以下内容：

* **entries**：战利品条目列表。
* **conditions**：应用于此池的战利品条件列表。如果任一条件失败，池中的条目将不会被运行。
* **functions**：应用于此池所有条目输出的战利品函数列表。
* **rolls 和 bonus_rolls**：两个数字提供器，用于确定此池的运行次数。公式为 `rolls + bonus_rolls * luck`​，其中 `luck`​ 值在战利品参数中设置。
* **name**：战利品池的名称（NeoForge 新增）。GLMs 可以使用此名称。如果未指定，则为战利品池的哈希码，前缀为 `custom#`​。

---

### 数字提供器（Number Provider）

数字提供器是一种在数据包上下文中获取（伪）随机数的方式。主要用于战利品表，但也用于其他上下文，例如世界生成。原版提供了以下六种数字提供器：

* **minecraft:constant**：常量值。通过 `ConstantValue#exactly`​ 创建。
* **minecraft:uniform**：均匀分布的随机整数或浮点数。通过 `UniformGenerator#between`​ 创建。
* **minecraft:binomial**：二项分布的随机整数。通过 `BinomialDistributionGenerator#binomial`​ 创建。
* **minecraft:score**：根据实体目标和分数名称获取分数值。通过 `ScoreboardValue#fromScoreboard`​ 创建。
* **minecraft:storage**：从命令存储中获取值。通过 `new StorageValue`​ 创建。
* **minecraft:enchantment_level**：根据附魔等级提供值。通过 `EnchantmentLevelProvider#forEnchantmentLevel`​ 创建。

模组开发者还可以注册自定义数字提供器。

---

### 战利品参数（Loot Parameters）

战利品参数是战利品表运行时提供的参数，代码中由 `LootContextParam<T>`​ 表示。它们可以被战利品条件和函数使用。例如，`minecraft:killed_by_player`​ 条件检查是否存在 `minecraft:player`​ 参数。

Minecraft 提供了以下战利品参数：

* **minecraft:origin**：与战利品表关联的位置，例如宝箱的位置。
* **minecraft:tool**：与战利品表关联的物品堆栈，例如用于破坏方块的物品。
* **minecraft:block_state**：与战利品表关联的方块状态，例如被破坏的方块状态。
* **minecraft:this_entity**：与战利品表关联的实体，通常是被杀死的实体。
* **minecraft:damage_source**：与战利品表关联的伤害来源，通常是杀死实体的伤害来源。

自定义战利品参数可以通过 `new LootContextParam<T>`​ 创建。

---

### 战利品表（Loot Table）

结合所有上述元素，我们最终得到战利品表。战利品表 JSON 可以指定以下值：

* **pools**：战利品池列表。
* **neoforge:conditions**：数据加载条件列表（注意：这是数据加载条件，不是战利品条件！）。
* **functions**：应用于此战利品表所有条目输出的战利品函数列表。
* **type**：战利品参数集，用于验证战利品参数的正确使用。可选；如果省略，将跳过验证。
* **random_sequence**：此战利品表的随机序列，形式为资源位置。

---

### 数据生成（Datagen）

战利品表可以通过注册 `LootTableProvider`​ 并提供 `LootTableSubProvider`​ 列表来生成数据。例如：

```java
@SubscribeEvent
public static void onGatherData(GatherDataEvent event) {
    event.getGenerator().addProvider(
        event.includeServer(),
        output -> new MyLootTableProvider(
            output,
            Set.of(),
            List.of(...)
        )
    );
}
```

​`LootTableSubProvider`​ 是实际生成战利品表的地方。通过实现 `LootTableSubProvider`​ 并重写 `#generate`​ 方法，可以生成自定义战利品表。
