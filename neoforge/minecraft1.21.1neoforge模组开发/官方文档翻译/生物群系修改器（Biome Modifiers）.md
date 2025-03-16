# 生物群系修改器（Biome Modifiers）

### 生物群系修改器（Biome Modifiers）

生物群系修改器是一个数据驱动的系统，允许修改生物群系的许多方面，包括注入或移除 `PlacedFeature`​、添加或移除生物生成、更改气候以及调整植被和水体颜色。NeoForge 提供了几种默认的生物群系修改器，涵盖了大多数玩家和模组开发者的使用场景。

---

#### 推荐阅读部分

* **玩家或资源包开发者**：

  * [应用生物群系修改器](#应用生物群系修改器)
  * [内置的 NeoForge 生物群系修改器](#内置的生物群系修改器)（Built-in Neoforge Biome Modifiers）
* **模组开发者进行简单的添加或移除生物群系修改**：

  * [应用生物群系修改器](#应用生物群系修改器) （Applying Biome Modifiers）
  * [内置的 NeoForge 生物群系修改器](#内置的生物群系修改器)
  * [数据生成生物群系修改器](#数据生成生物群系修改器)
* **模组开发者希望进行自定义或复杂的生物群系修改**：

  * [应用生物群系修改器](#应用生物群系修改器)
  * [创建自定义生物群系修改器](#创建自定义生物群系修改器)
  * [数据生成生物群系修改器](#数据生成生物群系修改器)

---

#### 应用生物群系修改器

为了让 NeoForge 加载生物群系修改器的 JSON 文件，文件需要位于模组资源的 `data/<modid>/neoforge/biome_modifier/<path>.json`​ 文件夹中，或者在数据包中。然后，当世界加载时，NeoForge 会读取生物群系修改器的指令，并将其描述的修改应用到所有目标生物群系。模组中已有的生物群系修改器可以通过数据包在相同位置和名称的 JSON 文件进行覆盖。

JSON 文件可以手动创建，遵循“内置的 NeoForge 生物群系修改器”部分中的示例，或者按照“数据生成生物群系修改器”部分中的方法生成。

---

#### 内置的生物群系修改器

这些生物群系修改器由 NeoForge 注册，供任何人使用。

---

##### 无操作（None）

此生物群系修改器没有任何操作，不会进行任何修改。资源包制作者和玩家可以在数据包中使用此修改器，通过覆盖模组的生物群系修改器 JSON 文件来禁用它们。

**JSON**：

```json
{
    "type": "neoforge:none"
}
```

---

##### 添加特征（Add Features）

此生物群系修改器类型将 `PlacedFeature`​（例如树木或矿石）添加到生物群系中，以便它们在世界生成期间生成。修改器接受生物群系的 ID 或标签、要添加到选定生物群系的 `PlacedFeature`​ ID 或标签，以及特征将在其中生成的 `GenerationStep.Decoration`​。

**JSON**：

```json
{
    "type": "neoforge:add_features",
    "biomes": "#namespace:your_biome_tag",
    "features": "namespace:your_feature",
    "step": "underground_ores"
}
```

**注意**：在向生物群系添加原版的 `PlacedFeature`​ 时应小心，因为这可能会导致特征循环冲突（两个生物群系在其特征列表中具有相同的两个特征，但在同一生成步骤中的顺序不同），从而导致崩溃。出于类似的原因，不应在多个生物群系修改器中使用相同的 `PlacedFeature`​。

---

##### 移除特征（Remove Features）

此生物群系修改器类型从生物群系中移除特征（例如树木或矿石），以便它们在世界生成期间不再生成。修改器接受生物群系的 ID 或标签、要从选定生物群系中移除的 `PlacedFeature`​ ID 或标签，以及特征将从其中移除的 `GenerationStep.Decoration`​。

**JSON**：

```json
{
    "type": "neoforge:remove_features",
    "biomes": "#namespace:your_biome_tag",
    "features": "namespace:problematic_feature",
    "steps": ["underground_ores", "underground_decoration"]
}
```

---

##### 添加生成（Add Spawns）

此生物群系修改器类型向生物群系添加实体生成。修改器接受生物群系的 ID 或标签，以及要添加的实体的 `SpawnerData`​。每个 `SpawnerData`​ 包含实体 ID、生成权重以及每次生成的最小/最大实体数量。

**JSON**：

```json
{
    "type": "neoforge:add_spawns",
    "biomes": "#namespace:biome_tag",
    "spawners": [
        {
            "type": "namespace:entity_type",
            "weight": 100,
            "minCount": 1,
            "maxCount": 4
        }
    ]
}
```

**注意**：如果你是模组开发者并添加了新实体，请确保实体已在 `RegisterSpawnPlacementsEvent`​ 中注册生成限制。生成限制用于确保实体安全地生成在地面或水中。如果未注册生成限制，你的实体可能会生成在半空中，掉落并死亡。

---

##### 移除生成（Remove Spawns）

此生物群系修改器类型从生物群系中移除实体生成。修改器接受生物群系的 ID 或标签，以及要移除的实体的 `EntityType`​ ID 或标签。

**JSON**：

```json
{
    "type": "neoforge:remove_spawns",
    "biomes": "#namespace:biome_tag",
    "entity_types": "#namespace:entitytype_tag"
}
```

---

##### 添加生成成本（Add Spawn Costs）

允许向生物群系添加新的生成成本。生成成本是一种较新的方式，用于使生物在生物群系中分散生成，以减少聚集。它通过让实体释放电荷来实现，这些电荷会累积并与周围其他实体的电荷相加。当生成新实体时，生成算法会寻找一个位置，该位置的总电荷场乘以生成实体的电荷值小于生成实体的能量预算。这是一种高级的生成方式，因此建议参考灵魂沙谷生物群系（这是该系统的主要使用者）以获取现有的值。

**JSON**：

```json
{
    "type": "neoforge:add_spawn_costs",
    "biomes": "#namespace:biome_tag",
    "entity_types": "#minecraft:skeletons",
    "spawn_cost": {
        "energy_budget": 1.0,
        "charge": 0.1
    }
}
```

---

##### 移除生成成本（Remove Spawn Costs）

允许从生物群系中移除生成成本。生成成本是一种较新的方式，用于使生物在生物群系中分散生成，以减少聚集。修改器接受生物群系的 ID 或标签，以及要移除生成成本的实体的 `EntityType`​ ID 或标签。

**JSON**：

```json
{
    "type": "neoforge:remove_spawn_costs",
    "biomes": "#namespace:biome_tag",
    "entity_types": "#minecraft:skeletons"
}
```

---

##### 添加传统雕刻器（Add Legacy Carvers）

此生物群系修改器类型允许向生物群系添加雕刻器洞穴和峡谷。这些是在洞穴与悬崖更新之前用于洞穴生成的内容。它**无法**向生物群系添加噪声洞穴，因为噪声洞穴是基于某些噪声区块生成器系统的一部分，并不实际与生物群系绑定。

**JSON**：

```json
{
    "type": "neoforge:add_carvers",
    "biomes": "minecraft:plains",
    "carvers": "examplemod:add_carvers_example",
    "step": "air"
}
```

---

##### 移除传统雕刻器（Remove Legacy Carvers）

此生物群系修改器类型允许从生物群系中移除雕刻器洞穴和峡谷。这些是在洞穴与悬崖更新之前用于洞穴生成的内容。它**无法**移除噪声洞穴，因为噪声洞穴是维度噪声设置系统的一部分，并不实际与生物群系绑定。

**JSON**：

```json
{
    "type": "neoforge:remove_carvers",
    "biomes": "minecraft:plains",
    "carvers": "examplemod:add_carvers_example",
    "steps": ["air", "liquid"]
}
```

---

#### 可用的装饰步骤值（Available Values for Decoration Steps）

许多上述 JSON 中的 `step`​ 或 `steps`​ 字段指的是 `GenerationStep.Decoration`​ 枚举。此枚举按以下顺序列出步骤，这也是游戏在世界生成期间使用的顺序。尝试将特征放在最符合它们的步骤中。

|步骤|描述|
| ------| -----------------------------------------------------------|
|​`raw_generation`​|首先运行。用于特殊地形特征，例如小型末地岛屿。|
|​`lakes`​|用于生成类似池塘的特征，例如熔岩湖。|
|​`local_modifications`​|用于地形修改，例如晶洞、冰山、巨石或滴水石。|
|​`underground_structures`​|用于小型地下结构特征，例如地牢或化石。|
|​`surface_structures`​|用于小型地表结构特征，例如沙漠井。|
|​`strongholds`​|专用于要塞结构。未修改的 Minecraft 中没有特征添加到这里。|
|​`underground_ores`​|用于所有矿石和矿脉的生成。包括金、泥土、花岗岩等。|
|​`underground_decoration`​|通常用于装饰洞穴。滴水石簇和幽匿脉络在此阶段生成。|
|​`fluid_springs`​|小型熔岩瀑布和水瀑布来自此阶段的特征。|
|​`vegetal_decoration`​|几乎所有植物（花、树、藤蔓等）都添加到此阶段。|
|​`top_layer_modification`​|最后运行。用于在寒冷生物群系的地表放置雪和冰。|

---

#### 创建自定义生物群系修改器

##### 生物群系修改器的实现

在底层，生物群系修改器由三部分组成：

1. 数据包注册的 `BiomeModifier`​，用于修改生物群系构建器。
2. 静态注册的 `MapCodec`​，用于编码和解码修改器。
3. JSON 文件，用于构建 `BiomeModifier`​，使用 `MapCodec`​ 的注册 ID 作为可索引类型。

​`BiomeModifier`​ 包含两个方法：`#modify`​ 和 `#codec`​。`modify`​ 接受当前生物群系的 `Holder`​、当前的 `BiomeModifier.Phase`​ 以及要修改的生物群系构建器。每个 `BiomeModifier`​ 在每个阶段被调用一次，以组织何时应对生物群系进行某些修改：

|阶段|描述|
| ------| ------------------------------------|
|​`BEFORE_EVERYTHING`​|用于在标准阶段之前运行的所有内容。|
|​`ADD`​|添加特征、生物生成等。|
|​`REMOVE`​|移除特征、生物生成等。|
|​`MODIFY`​|修改单个值（例如气候、颜色）。|
|​`AFTER_EVERYTHING`​|用于在标准阶段之后运行的所有内容。|

所有 `BiomeModifier`​ 都包含一个 `type`​ 键，该键引用用于 `BiomeModifier`​ 的 `MapCodec`​ 的 ID。`codec`​ 接受用于编码和解码修改器的 `MapCodec`​。此 `MapCodec`​ 是静态注册的，其 ID 用作 `BiomeModifier`​ 的类型。

```java
public record ExampleBiomeModifier(HolderSet<Biome> biomes, int value) implements BiomeModifier {
    
    @Override
    public void modify(Holder<Biome> biome, Phase phase, ModifiableBiomeInfo.BiomeInfo.Builder builder) {
        if (phase == /* 选择最适合你要修改的阶段的阶段 */) {
            // 修改 'builder'，检查生物群系的任何信息
        }
    }

    @Override
    public MapCodec<? extends BiomeModifier> codec() {
        return EXAMPLE_BIOME_MODIFIER.get();
    }
}

// 在某个注册类中
private static final DeferredRegister<MapCodec<? extends BiomeModifier>> BIOME_MODIFIERS =
    DeferredRegister.create(NeoForgeRegistries.Keys.BIOME_MODIFIER_SERIALIZERS, MOD_ID);

public static final Supplier<MapCodec<ExampleBiomeModifier>> EXAMPLE_BIOME_MODIFIER =
    BIOME_MODIFIERS.register("example_biome_modifier", () -> RecordCodecBuilder.mapCodec(instance ->
        instance.group(
            Biome.LIST_CODEC.fieldOf("biomes").forGetter(ExampleBiomeModifier::biomes),
            Codec.INT.fieldOf("value").forGetter(ExampleBiomeModifier::value)
        ).apply(instance, ExampleBiomeModifier::new)
    ));
```

---

#### 数据生成生物群系修改器

可以通过将 `RegistrySetBuilder`​ 传递给 `DatapackBuiltinEntriesProvider`​ 来生成生物群系修改器的 JSON 文件。JSON 文件将位于 `data/<modid>/neoforge/biome_modifier/<path>.json`​。

有关 `RegistrySetBuilder`​ 和 `DatapackBuiltinEntriesProvider`​ 的更多信息，请参阅[数据生成数据包注册表](#数据生成数据包注册表)的文章。

```java
// 定义我们的 BiomeModifier 的 ResourceKey。
public static final ResourceKey<BiomeModifier> EXAMPLE_MODIFIER = ResourceKey.create(
    NeoForgeRegistries.Keys.BIOME_MODIFIERS, // 此键所属的注册表
    ResourceLocation.fromNamespaceAndPath(MOD_ID, "example_modifier") // 注册表名称
);

// BUILDER 是传递给 DatapackBuiltinEntriesProvider 的 RegistrySetBuilder
// 在 GatherDataEvent 的监听器中。
BUILDER.add(NeoForgeRegistries.Keys.BIOME_MODIFIERS, bootstrap -> {
    // 查找任何必要的注册表。
    // 仅当需要获取标签数据时才需要查找静态注册表。
    HolderGetter<Biome> biomes = bootstrap.lookup(Registries.BIOME);

    // 注册生物群系修改器。
    bootstrap.register(EXAMPLE_MODIFIER,
        new ExampleBiomeModifier(
            biomes.getOrThrow(Tags.Biomes.IS_OVERWORLD),
            20
        )
    );
});
```

这将生成以下 JSON 文件：

```json
// 在 data/examplemod/neoforge/biome_modifier/example_modifier.json
{
    "type": "examplemod:example_biome_modifier",
    "biomes": "#c:is_overworld",
    "value": 20
}
```

---

希望这些内容对你的开发有所帮助！如果有其他问题，欢迎随时提问。
