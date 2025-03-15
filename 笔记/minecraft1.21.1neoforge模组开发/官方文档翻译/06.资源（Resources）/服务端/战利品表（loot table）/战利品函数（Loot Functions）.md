# 战利品函数（Loot Functions）

### 战利品函数（Loot Functions）

战利品函数用于修改战利品条目的结果，或战利品池或战利品表的多个结果。在这两种情况下，都会定义一个函数列表，并按顺序运行。在数据生成期间，可以通过调用 `#apply`​ 将战利品函数应用于 `LootPoolSingletonContainer.Builder<?>`​、`LootPool.Builder`​ 和 `LootTable.Builder`​。本文将概述可用的战利品函数。要创建自定义战利品函数，请参阅 [自定义战利品函数](#custom-loot-functions)。

**注意**：战利品函数不能应用于复合战利品条目（`CompositeEntryBase`​ 的子类及其关联的构建器类）。必须手动添加到每个单例条目中。

除了 `minecraft:sequence`​ 之外，所有原版战利品函数都可以在 `conditions`​ 块中指定战利品条件。如果其中一个条件失败，则不会应用该函数。在代码端，这是由 `LootItemConditionalFunction`​ 控制的，所有战利品函数（除了 `SequenceFunction`​）都扩展了它。

---

### `minecraft:set_item`​

设置结果物品堆栈中使用的不同物品。

```json
{
    "function": "minecraft:set_item",
    // 要使用的物品
    "item": "minecraft:dirt"
}
```

目前无法在数据生成期间创建此函数。

---

### `minecraft:set_count`​

设置结果物品堆栈中使用的物品数量。使用数字提供器。

```json
{
    "function": "minecraft:set_count",
    // 要使用的数量
    "count": {
        "type": "minecraft:uniform",
        "min": 1,
        "max": 3
    },
    // 是否添加到现有值而不是设置它。可选，默认为 false
    "add": true
}
```

在数据生成期间，调用 `SetItemCountFunction#setCount`​ 并传入所需的数字提供器和可选的 `add`​ 布尔值，以构建此函数的构建器。

---

### `minecraft:explosion_decay`​

应用爆炸衰减。物品有 1 / `explosion_radius`​ 的几率“幸存”。根据数量多次运行。需要 `minecraft:explosion_radius`​ 战利品参数，如果该参数不存在，则不进行修改。

```json
{
    "function": "minecraft:explosion_decay"
}
```

在数据生成期间，调用 `ApplyExplosionDecay#explosionDecay`​ 以构建此函数的构建器。

---

### `minecraft:limit_count`​

将物品堆栈的数量限制在给定的 `IntRange`​ 内。

```json
{
    "function": "minecraft:limit_count",
    // 要使用的限制。可以具有最小值、最大值或两者。
    "limit": {
        "max": 32
    }
}
```

在数据生成期间，调用 `LimitCount#limitCount`​ 并传入所需的 `IntRange`​，以构建此函数的构建器。

---

### `minecraft:set_custom_data`​

在物品堆栈上设置自定义 NBT 数据。

```json
{
    "function": "minecraft:set_custom_data",
    "tag": {
        "exampleproperty": 0
    }
}
```

在数据生成期间，调用 `SetCustomDataFunction#setCustomData`​ 并传入所需的 `CompoundTag`​，以构建此函数的构建器。

**警告**：此函数通常应被视为已弃用。请改用 `minecraft:set_components`​。

---

### `minecraft:copy_custom_data`​

将自定义 NBT 数据从方块实体或实体源复制到物品堆栈。对于方块实体，不鼓励使用此函数，请改用 `minecraft:copy_components`​ 或 `minecraft:set_contents`​。对于实体，需要设置实体目标。需要与指定源（实体目标或方块实体）对应的战利品参数，如果该参数不存在，则不进行修改。

```json
{
    "function": "minecraft:copy_custom_data",
    // 要使用的源。有效值可以是实体目标，或 "block_entity" 以使用战利品上下文的方块实体参数，或 "storage" 以使用命令存储。
    // 如果为 "storage"，则可以是一个 JSON 对象，额外指定要使用的命令存储路径。
    "source": "this",
    // 使用 "storage" 的示例
    "source": {
        "type": "storage",
        "source": "examplepath"
    },
    // 复制操作
    "ops": [
        {
            // 源和目标路径。在此示例中，我们从源中的 "src" 复制到目标中的 "dest"。
            "source": "src",
            "target": "dest",
            // 合并策略。有效值为 "replace"、"append" 和 "merge"。
            "op": "merge"
        }
    ]
}
```

在数据生成期间，调用 `CopyCustomDataFunction#copy`​ 并传入所需的源和目标值以及合并策略（可选，默认为 `replace`​），以构建此函数的构建器。

---

### `minecraft:set_components`​

在物品堆栈上设置数据组件值。大多数原版用例都有专门的函数，如下所述。

```json
{
    "function": "minecraft:set_components",
    // 可以使用任何组件。在此示例中，我们将物品的染色颜色设置为红色。
    "components": {
        "dyed_color": {
            "rgb": 16711680
        }
    }
}
```

在数据生成期间，调用 `SetComponentsFunction#setComponent`​ 并传入所需的数据组件和值，以构建此函数的构建器。

---

### `minecraft:copy_components`​

将数据组件值从方块实体复制到物品堆栈。需要 `minecraft:block_entity`​ 战利品参数，如果该参数不存在，则不进行修改。

```json
{
    "function": "minecraft:copy_components",
    // 系统设计为允许多个源，但目前仅支持方块实体。
    "source": "block_entity",
    // 默认情况下，复制所有组件。"exclude" 列表允许排除某些组件，"include" 列表允许显式重新包含组件。两个字段都是可选的。
    "exclude": [],
    "include": []
}
```

在数据生成期间，调用 `CopyComponentsFunction#copyComponents`​ 并传入所需的数据源（通常为 `CopyComponentsFunction.Source.BLOCK_ENTITY`​），以构建此函数的构建器。

---

### `minecraft:copy_state`​

将方块状态属性复制到物品堆栈的 `block_state`​ 数据组件中，用于尝试放置方块时。必须显式指定要复制的方块状态属性。需要 `minecraft:block_state`​ 战利品参数，如果该参数不存在，则不进行修改。

```json
{
    "function": "minecraft:copy_state",
    // 预期的方块。如果与实际破坏的方块不匹配，则不运行该函数。
    "block": "minecraft:oak_slab",
    // 要保存的方块状态属性
    "properties": {
        "type": "top"
    }
}
```

在数据生成期间，调用 `CopyBlockState#copyState`​ 并传入方块，以构建此条件的构建器。然后可以使用 `#copy`​ 在构建器上设置所需的方块状态属性值。

---

### `minecraft:set_contents`​

设置物品堆栈的内容。

```json
{
    "function": "minecraft:set_contents",
    // 要使用的内容组件。有效值为 "container"、"bundle_contents" 和 "charged_projectiles"。
    "component": "container",
    // 要添加到内容中的战利品条目列表
    "entries": [
        {
            "type": "minecraft:empty",
            "weight": 3
        },
        {
            "type": "minecraft:item",
            "item": "minecraft:arrow"
        }
    ]
}
```

在数据生成期间，调用 `SetContainerContents#setContents`​ 并传入所需的内容组件，以构建此函数的构建器。然后调用 `#withEntry`​ 在构建器上添加条目。

---

### `minecraft:modify_contents`​

对物品堆栈的内容应用函数。

```json
{
    "function": "minecraft:modify_contents",
    // 要使用的内容组件。有效值为 "container"、"bundle_contents" 和 "charged_projectiles"。
    "component": "container",
    // 要使用的函数
    "modifier": "apply_explosion_decay"
}
```

目前无法在数据生成期间创建此函数。

---

### `minecraft:set_loot_table`​

在结果物品堆栈上设置容器战利品表。用于箱子和其他在放置时保留此属性的战利品容器。

```json
{
    "function": "minecraft:set_loot_table",
    // 要使用的战利品表的 ID
    "name": "minecraft:entities/enderman",
    // 目标方块实体的方块实体类型的 ID
    "type": "minecraft:chest",
    // 生成战利品表的随机种子。可选，默认为 0
    "seed": 42
}
```

在数据生成期间，调用 `SetContainerLootTable#withLootTable`​ 并传入所需的方块实体类型、战利品表资源键和可选的种子，以构建此函数的构建器。

---

### `minecraft:set_name`​

为结果物品堆栈设置名称。名称可以是 `Component`​ 而不是字面字符串。它也可以从实体目标解析。如果适用，需要相应的实体战利品参数，如果该参数不存在，则不进行修改。

```json
{
    "function": "minecraft:set_name",
    "name": "Funny Item",
    // 要使用的实体目标
    "entity": "this",
    // 是设置自定义名称（"custom_name"）还是物品名称本身（"item_name"）。
    // 自定义名称以斜体显示，可以在铁砧中更改，而物品名称则不能。
    "target": "custom_name"
}
```

在数据生成期间，调用 `SetNameFunction#setName`​ 并传入所需的名称组件、名称目标和可选的实体目标，以构建此函数的构建器。

---

### `minecraft:copy_name`​

将实体目标或方块实体的名称复制到结果物品堆栈中。需要与指定源（实体目标或方块实体）对应的战利品参数，如果该参数不存在，则不进行修改。

```json
{
    "function": "minecraft:copy_name",
    // 实体目标，或 "block_entity" 如果要复制方块实体的名称
    "source": "this"
}
```

在数据生成期间，调用 `CopyNameFunction#copyName`​ 并传入所需的实体源，以构建此函数的构建器。

---

### `minecraft:set_lore`​

为结果物品堆栈设置 lore（工具提示行）。行可以是 `Component`​ 而不是字面字符串。它也可以从实体目标解析。如果适用，需要相应的实体战利品参数，如果该参数不存在，则不进行修改。

```json
{
    "function": "minecraft:set_lore",
    "lore": [
        "Funny Lore",
        "Funny Lore 2"
    ],
    // 使用的合并模式。有效值为：
    // - "append"：将条目附加到任何现有的 lore 条目。
    // - "insert"：在特定位置插入条目。位置由名为 "offset" 的附加字段表示。"offset" 是可选的，默认为 0。
    // - "replace_all"：删除所有先前的条目，然后附加条目。
    // - "replace_section"：删除一部分条目，然后在该位置添加条目。
    //   删除的部分由 "offset" 和可选的 "size" 字段表示。
    //   如果省略 "size"，则使用 "lore" 中的行数。
    "mode": {
        "type": "insert",
        "offset": 0
    },
    // 要使用的实体目标
    "entity": "this"
}
```

在数据生成期间，调用 `SetLoreFunction#setLore`​ 以构建此函数的构建器。然后根据需要调用 `#addLine`​、`#setMode`​ 和 `#setResolutionContext`​。

---

### `minecraft:toggle_tooltips`​

启用或禁用某些组件工具提示。

```json
{
    "function": "minecraft:toggle_tooltips",
    "toggles": {
        // 所有值都是可选的。如果省略，这些值将使用堆栈上的预先存在的值。
        // 预先存在的值通常为 true，除非它们已被另一个函数修改。
        "minecraft:attribute_modifiers": false,
        "minecraft:can_break": false,
        "minecraft:can_place_on": false,
        "minecraft:dyed_color": false,
        "minecraft:enchantments": false,
        "minecraft:jukebox_playable": false,
        "minecraft:stored_enchantments": false,
        "minecraft:trim": false,
        "minecraft:unbreakable": false
    }
}
```

目前无法在数据生成期间创建此函数。

---

### `minecraft:enchant_with_levels`​

使用给定的等级随机附魔物品堆栈。使用数字提供器。

```json
{
    "function": "minecraft:enchant_with_levels",
    // 要使用的等级数量
    "levels": {
        "type": "minecraft:uniform",
        "min": 10,
        "max": 30
    },
    // 可能的附魔列表。可选，默认为物品的所有适用附魔。
    "options": [
        "minecraft:sharpness",
        "minecraft:fire_aspect"
    ]
}
```

在数据生成期间，调用 `EnchantWithLevelsFunction#enchantWithLevels`​ 并传入所需的数字提供器，以构建此函数的构建器。然后，如果需要，使用 `#fromOptions`​ 在构建器上设置附魔列表。

---

### `minecraft:enchant_randomly`​

使用一个随机附魔附魔物品。

```json
{
    "function": "minecraft:enchant_randomly",
    // 可能的附魔列表。可选，默认为所有附魔。
    "options": [
        "minecraft:sharpness",
        "minecraft:fire_aspect"
    ],
    // 是否仅允许兼容的附魔，或任何附魔。可选，默认为 true。
    "only_compatible": true
}
```

在数据生成期间，调用 `EnchantRandomlyFunction#randomEnchantment`​ 或 `EnchantRandomlyFunction#randomApplicableEnchantment`​ 以构建此函数的构建器。然后，如果需要，调用 `#withEnchantment`​ 或 `#withOneOf`​。

---

### `minecraft:set_enchantments`​

在结果物品堆栈上设置附魔。

```json
{
    "function": "minecraft:set_enchantments",
    // 附魔到数字提供器的映射
    "enchantments": {
        "minecraft:fire_aspect": 2,
        "minecraft:sharpness": {
            "type": "minecraft:uniform",
            "min": 3,
            "max": 5
        }
    },
    // 是否将附魔等级添加到现有等级而不是覆盖它们。可选，默认为 false。
    "add": true
}
```

在数据生成期间，调用 `new SetEnchantmentsFunction.Builder`​ 并传入 `add`​ 布尔值（可选），以构建此函数的构建器。然后调用 `#withEnchantment`​ 添加要设置的附魔。

---

### `minecraft:enchanted_count_increase`​

根据附魔值增加物品堆栈的数量。使用数字提供器。需要 `minecraft:attacking_entity`​ 战利品参数，如果该参数不存在，则不进行修改。

```json
{
    "function": "minecraft:enchanted_count_increase",
    // 要使用的附魔
    "enchantment": "minecraft:fortune",
    // 每级增加的数量。数字提供器每函数运行一次，而不是每级运行一次。
    "count": {
        "type": "minecraft:uniform",
        "min": 1,
        "max": 3
    },
    // 堆栈大小限制，无论附魔等级如何，都不会超过此限制。可选。
    "limit": 5
}
```

在数据生成期间，调用 `EnchantedCountIncreaseFunction#lootingMultiplier`​ 并传入所需的数字提供器，以构建此函数的构建器。然后可选地调用 `#setLimit`​。

---

### `minecraft:apply_bonus`​

根据附魔值和各种公式增加物品堆栈的数量。需要 `minecraft:tool`​ 战利品参数，如果该参数不存在，则不进行修改。

```json
{
    "function": "minecraft:apply_bonus",
    // 要查询的附魔值
    "enchantment": "minecraft:fortune",
    // 要使用的公式。有效值为：
    // - "minecraft:binomial_with_bonus_count"：使用二项分布应用加成，n = 附魔等级 + 额外值，p = 概率。
    // - "minecraft:ore_drops"：使用矿石掉落的特殊公式应用加成，包括随机性。
    // - "minecraft:uniform_bonus_count"：根据附魔等级乘以常数乘数增加加成。
    "formula": "ore_drops",
    // 参数值，取决于公式。
    // 如果公式为 "minecraft:binomial_with_bonus_count"，则需要 "extra" 和 "probability"。
    // 如果公式为 "minecraft:ore_drops"，则不需要参数。
    // 如果公式为 "minecraft:uniform_bonus_count"，则需要 "bonusMultiplier"。
    "parameters": {}
}
```

在数据生成期间，调用 `ApplyBonusCount#addBonusBinomialDistributionCount`​、`ApplyBonusCount#addOreBonusCount`​ 或 `ApplyBonusCount#addUniformBonusCount`​ 并传入附魔和其他所需参数（取决于公式），以构建此函数的构建器。

---

### `minecraft:furnace_smelt`​

尝试将物品熔炼为熔炉中的物品，如果无法熔炼，则返回未修改的物品堆栈。

```json
{
    "function": "minecraft:furnace_smelt"
}
```

在数据生成期间，调用 `SmeltItemFunction#smelted`​ 以构建此函数的构建器。

---

### `minecraft:set_damage`​

在结果物品堆栈上设置耐久度伤害值。使用数字提供器。

```json
{
    "function": "minecraft:set_damage",
    // 要设置的伤害
    "damage": {
        "type": "minecraft:uniform",
        "min": 10,
        "max": 300
    },
    // 是否添加到现有伤害而不是设置它。可选，默认为 false。
    "add": true
}
```

在数据生成期间，调用 `SetItemDamageFunction#setDamage`​ 并传入所需的数字提供器和可选的 `add`​ 布尔值，以构建此函数的构建器。

---

### `minecraft:set_attributes`​

在结果物品堆栈上添加属性修饰符列表。

```json
{
    "function": "minecraft:set_attributes",
    // 属性修饰符列表
    "modifiers": [
        {
            // 修饰符的资源位置 ID。应前缀为你的模组 ID。
            "id": "examplemod:example_modifier",
            // 修饰符所针对的属性的 ID。
            "attribute": "minecraft:generic.attack_damage",
            // 属性修饰符操作。
            // 有效值为 "add_value"、"add_multiplied_base" 和 "add_multiplied_total"。
            "operation": "add_value",
            // 修饰符的数量。这也可以是数字提供器。
            "amount": 5,
            // 修饰符适用的槽位。有效值为 "any"（任何库存槽位）、
            // "mainhand"、"offhand"、"hand"（主手/副手/双手）、
            // "feet"、"legs"、"chest"、"head"、"armor"（靴子/护腿/胸甲/头盔/任何护甲槽位）
            // 和 "body"（马铠和类似槽位）。
            "slot": "armor"
        }
    ],
    // 是否替换现有值而不是添加到它们。可选，默认为 true。
    "replace": false
}
```

在数据生成期间，调用 `SetAttributesFunction#setAttributes`​ 以构建此函数的构建器。然后使用 `#withModifier`​ 在构建器上添加修饰符。使用 `SetAttributesFunction#modifier`​ 获取修饰符。

---

### `minecraft:set_potion`​

在结果物品堆栈上设置药水。

```json
{
    "function": "minecraft:set_potion",
    // 药水的 ID
    "id": "minecraft:strength"
}
```

在数据生成期间，调用 `SetPotionFunction#setPotion`​ 并传入所需的药水，以构建此函数的构建器。

---

### `minecraft:set_stew_effect`​

在结果物品堆栈上设置炖煮效果列表。

```json
{
    "function": "minecraft:set_stew_effect",
    // 要设置的效果
    "effects": [
        {
            // 效果 ID
            "type": "minecraft:fire_resistance",
            // 效果持续时间，以刻为单位。这也可以是数字提供器。
            "duration": 100
        }
    ]
}
```

在数据生成期间，调用 `SetStewEffectFunction#stewEffect`​ 以构建此函数的构建器。然后调用 `#withModifier`​。

---

### `minecraft:set_ominous_bottle_amplifier`​

在结果物品堆栈上设置不祥瓶放大器。使用数字提供器。

```json
{
    "function": "minecraft:set_ominous_bottle_amplifier",
    // 要使用的放大器
    "amplifier": {
        "type": "minecraft:uniform",
        "min": 1,
        "max": 3
    }
}
```

在数据生成期间，调用 `SetOminousBottleAmplifierFunction#amplifier`​ 并传入所需的数字提供器，以构建此函数的构建器。

---

### `minecraft:exploration_map`​

如果且仅当物品是地图时，将结果物品堆栈转换为探索地图。需要 `minecraft:origin`​ 战利品参数，如果该参数不存在，则不进行修改。

```json
{
    "function": "minecraft:exploration_map",
    // 结构标签，包含探索地图可以引导到的结构。
    // 可选，默认为 "minecraft:on_treasure_maps"，默认情况下仅包含埋藏的宝藏。
    "destination": "minecraft:eye_of_ender_located",
    // 要使用的地图装饰类型。请参阅 `MapDecorationTypes` 类以获取可用值。
    // 可选，默认为 "minecraft:mansion"。
    "decoration": "minecraft:target_x",
    // 要使用的缩放级别。可选，默认为 2。
    "zoom": 4,
    // 要使用的搜索半径。可选，默认为 50。
    "search_radius": 25,
    // 搜索结构时是否跳过现有区块。可选，默认为 true。
    "skip_existing_chunks": true
}
```

在数据生成期间，调用 `ExplorationMapFunction#makeExplorationMap`​ 以构建此函数的构建器。然后根据需要调用构建器上的各种设置器。

---

### `minecraft:fill_player_head`​

根据给定的实体目标在结果物品堆栈上设置玩家头颅所有者。需要相应的战利品参数，如果该参数不存在，则不进行修改。

```json
{
    "function": "minecraft:fill_player_head",
    // 要使用的实体目标。如果未解析为玩家，则不修改堆栈。
    "entity": "this_entity"
}
```

在数据生成期间，调用 `FillPlayerHead#fillPlayerHead`​ 并传入所需的实体目标，以构建此函数的构建器。

---

### `minecraft:set_banner_pattern`​

在结果物品堆栈上设置旗帜图案。这是用于旗帜，而不是旗帜图案物品。

```json
{
    "function": "minecraft:set_banner_patterns",
    // 旗帜图案图层列表
    "patterns": [
        {
            // 要使用的旗帜图案的 ID
            "pattern": "minecraft:globe",
            // 图层的染料颜色
            "color": "light_blue"
        }
    ],
    // 是否附加到现有图层而不是替换它们
    "append": true
}
```

在数据生成期间，调用 `SetBannerPatternFunction#setBannerPattern`​ 并传入 `append`​ 布尔值，以构建此函数的构建器。然后调用 `#addPattern`​ 添加图案。

---

### `minecraft:set_instrument`​

在结果物品堆栈上设置乐器标签。

```json
{
    "function": "minecraft:set_instrument",
    // 要使用的乐器标签
    "options": "minecraft:goat_horns"
}
```

在数据生成期间，调用 `SetInstrumentFunction#setInstrumentOptions`​ 并传入所需的乐器标签，以构建此函数的构建器。

---

### `minecraft:set_fireworks`​

```json
{
    "function": "minecraft:set_fireworks",
    // 要使用的爆炸。可选，如果省略，则使用现有的数据组件值。
    "explosions": [
        {
            // 要使用的烟花爆炸形状。有效的原版值为 "small_ball"、"large_ball"、
            // "star"、"creeper" 和 "burst"。可选，默认为 "small_ball"。
            "shape": "star",
            // 要使用的颜色。可选，默认为空列表。
            "colors": [
                16711680,
                65280
            ],
            // 要使用的褪色颜色。可选，默认为空列表。
            "fade_colors": [
                65280,
                255
            ],
            // 爆炸是否有尾迹。可选，默认为 false。
            "has_trail": true,
            // 爆炸是否有闪烁效果。可选，默认为 false。
            "has_twinkle": true
        }
    ],
    // 烟花的飞行持续时间。可选，如果省略，则使用现有的数据组件值。
    "flight_duration": 5
}
```

目前无法在数据生成期间创建此函数。

---

### `minecraft:set_firework_explosion`​

在结果物品堆栈上设置烟花爆炸。

```json
{
    "function": "minecraft:set_firework_explosion",
    // 要使用的烟花爆炸形状。有效的原版值为 "small_ball"、"large_ball"、
    // "star"、"creeper" 和 "burst"。可选，默认为 "small_ball"。
    "shape": "star",
    // 要使用的颜色。可选，默认为空列表。
    "colors": [
        16711680,
        65280
    ],
    // 要使用的褪色颜色。可选，默认为空列表。
    "fade_colors": [
        65280,
        255
    ],
    // 爆炸是否有尾迹。可选，默认为 false。
    "has_trail": true,
    // 爆炸是否有闪烁效果。可选，默认为 false。
    "has_twinkle": true
}
```

在数据生成期间，调用 `SetItemCountFunction#setCount`​ 并传入所需的数字提供器和可选的 `add`​ 布尔值，以构建此函数的构建器。

---

### `minecraft:set_book_cover`​

设置成书的非页面特定内容。

```json
{
    "function": "minecraft:set_book_cover",
    // 书名。可选，如果省略，书名保持不变。
    "title": "Hello World!",
    // 书作者。可选，如果省略，书作者保持不变。
    "author": "Steve",
    // 书生成，即它被复制的次数。限制在 0 到 3 之间。
    // 可选，如果省略，书生成保持不变。
    "generation": 2
}
```

在数据生成期间，调用 `new SetBookCoverFunction`​ 并传入所需的参数，以构建此函数的构建器。

---

### `minecraft:set_written_book_pages`​

设置成书的页面。

```json
{
    "function": "minecraft:set_written_book_pages",
    // 要设置的页面，作为字符串列表。
    "pages": [
        "Hello World!",
        "Hello World on page 2!",
        "Never Gonna Give You Up!"
    ],
    // 使用的合并模式。有效值为：
    // - "append"：将条目附加到任何现有的 lore 条目。
    // - "insert"：在特定位置插入条目。位置由名为 "offset" 的附加字段表示。"offset" 是可选的，默认为 0。
    // - "replace_all"：删除所有先前的条目，然后附加条目。
    // - "replace_section"：删除一部分条目，然后在该位置添加条目。
    //   删除的部分由 "offset" 和可选的 "size" 字段表示。
    //   如果省略 "size"，则使用 "lore" 中的行数。
    "mode": {
        "type": "insert",
        "offset": 0
    }
}
```

目前无法在数据生成期间创建此函数。

---

### `minecraft:set_writable_book_pages`​

设置可写书（书与笔）的页面。

```json
{
    "function": "minecraft:set_writable_book_pages",
    // 要设置的页面，作为字符串列表。
    "pages": [
        "Hello World!",
        "Hello World on page 2!",
        "Never Gonna Give You Up!"
    ],
    // 使用的合并模式。有效值为：
    // - "append"：将条目附加到任何现有的 lore 条目。
    // - "insert"：在特定位置插入条目。位置由名为 "offset" 的附加字段表示。"offset" 是可选的，默认为 0。
    // - "replace_all"：删除所有先前的条目，然后附加条目。
    // - "replace_section"：删除一部分条目，然后在该位置添加条目。
    //   删除的部分由 "offset" 和可选的 "size" 字段表示。
    //   如果省略 "size"，则使用 "lore" 中的行数。
    "mode": {
        "type": "insert",
        "offset": 0
    }
}
```

目前无法在数据生成期间创建此函数。

---

### `minecraft:set_custom_model_data`​

在结果物品堆栈上设置自定义模型数据。

```json
{
    "function": "minecraft:set_custom_model_data",
    // 要使用的自定义模型数据值。这也可以是数字提供器。
    "value": 4
}
```

目前无法在数据生成期间创建此函数。

---

### `minecraft:filtered`​

此函数接受一个 `ItemPredicate`​，用于检查 `tool`​ 战利品参数；如果检查成功，则运行另一个函数。`ItemPredicate`​ 可以指定有效的物品 ID 列表（`items`​）、物品数量的最小/最大范围（`count`​）、`DataComponentPredicate`​（`components`​）和 `ItemSubPredicate`​（`predicates`​）；所有字段都是可选的。需要 `minecraft:tool`​ 战利品参数，如果该参数不存在，则始终失败。

```json
{
    "function": "minecraft:filtered",
    // 要使用的自定义模型数据值。这也可以是数字提供器。
    "item_filter": {
        "items": [
            "minecraft:diamond_shovel"
        ]
    },
    // 要运行的其他战利品函数，可以是战利品修改器文件或内联函数列表。
    "modifier": "examplemod:example_modifier"
}
```

目前无法在数据生成期间创建此函数。

**警告**：此函数通常应被视为已弃用。请改用带有 `minecraft:match_tool`​ 条件的传递函数。

---

### `minecraft:reference`​

此函数引用一个物品修改器并将其应用于结果物品堆栈。有关更多信息，请参阅 [物品修改器](https://minecraft.fandom.com/wiki/Item_Modifiers)。

```json
{
    "function": "minecraft:reference",
    // 引用位于 data/examplemod/item_modifier/example_modifier.json 的物品修改器文件。
    "name": "examplemod:example_modifier"
}
```

在数据生成期间，调用 `FunctionReference#functionReference`​ 并传入引用的谓词文件的 ID，以构建此函数的构建器。

---

### `minecraft:sequence`​

此函数依次运行其他战利品函数。

```json
{
    "function": "minecraft:sequence",
    // 要运行的函数列表
    "functions": [
        {
            "function": "minecraft:set_count",
            // ...
        },
        {
            "function": "minecraft:explosion_decay"
        }
    ]
}
```

在数据生成期间，调用 `SequenceFunction#of`​ 并传入其他函数，以构建此条件的构建器。

---

### 另请参阅

* [Minecraft Wiki 上的物品修改器](https://minecraft.fandom.com/wiki/Item_Modifiers)
