# 战利品条件（Loot Conditions）

### 战利品条件（Loot Conditions）

战利品条件用于检查在当前上下文中是否应使用某个战利品条目或战利品池。在这两种情况下，都会定义一个条件列表；只有当所有条件都通过时，条目或池才会被使用。在数据生成期间，可以通过调用 `#when`​ 并将所需条件的实例传递给它，将条件添加到 `LootPoolEntryContainer.Builder<?>`​ 或 `LootPool.Builder`​ 中。本文将概述可用的战利品条件。要创建自定义战利品条件，请参阅 [自定义战利品条件](#custom-loot-conditions)。

---

### `minecraft:inverted`​

此条件接受另一个条件并反转其结果。需要另一个条件所需的战利品参数。

```json
{
    "condition": "minecraft:inverted",
    "term": {
        // 其他战利品条件
    }
}
```

在数据生成期间，调用 `InvertedLootItemCondition#invert`​ 并传入要反转的条件，以构建此条件的构建器。

---

### `minecraft:all_of`​

此条件接受任意数量的其他条件，如果所有子条件都返回 `true`​，则返回 `true`​。如果列表为空，则返回 `false`​。需要其他条件所需的战利品参数。

```json
{
    "condition": "minecraft:all_of",
    "terms": [
        {
            // 一个战利品条件
        },
        {
            // 另一个战利品条件
        },
        {
            // 又一个战利品条件
        }
    ]
}
```

在数据生成期间，调用 `AllOfCondition#allOf`​ 并传入所需的条件，以构建此条件的构建器。

---

### `minecraft:any_of`​

此条件接受任意数量的其他条件，如果至少有一个子条件返回 `true`​，则返回 `true`​。如果列表为空，则返回 `false`​。需要其他条件所需的战利品参数。

```json
{
    "condition": "minecraft:any_of",
    "terms": [
        {
            // 一个战利品条件
        },
        {
            // 另一个战利品条件
        },
        {
            // 又一个战利品条件
        }
    ]
}
```

在数据生成期间，调用 `AnyOfCondition#anyOf`​ 并传入所需的条件，以构建此条件的构建器。

---

### `minecraft:random_chance`​

此条件接受一个表示 0 到 1 之间几率的数字提供器，并根据该几率随机返回 `true`​ 或 `false`​。数字提供器通常不应返回超出 [0, 1] 区间的值。

```json
{
    "condition": "minecraft:random_chance",
    // 50% 的几率应用此条件
    "chance": 0.5
}
```

在数据生成期间，调用 `RandomChance#randomChance`​ 并传入数字提供器或（常量）浮点值，以构建此条件的构建器。

---

### `minecraft:random_chance_with_enchanted_bonus`​

此条件接受一个附魔 ID、一个 `LevelBasedValue`​ 和一个常量回退浮点值。如果指定的附魔存在，则查询 `LevelBasedValue`​ 的值。如果指定的附魔不存在或无法从 `LevelBasedValue`​ 中检索到值，则使用常量回退值。然后，条件随机返回 `true`​ 或 `false`​，之前确定的值表示返回 `true`​ 的几率。需要 `minecraft:attacking_entity`​ 参数，如果不存在则回退到等级 0。

```json
{
    "condition": "minecraft:random_chance_with_enchanted_bonus",
    // 每级抢夺附魔增加 20% 的成功几率
    "enchantment": "minecraft:looting",
    "enchanted_chance": {
        "type": "linear",
        "base": 0.2,
        "per_level_above_first": 0.2
    },
    // 如果没有抢夺附魔，则始终失败
    "unenchanted_chance": 0.0
}
```

在数据生成期间，调用 `LootItemRandomChanceWithEnchantedBonusCondition#randomChanceAndLootingBoost`​ 并传入注册表查找（`HolderLookup.Provider`​）、基础值和每级增加值，以构建此条件的构建器。或者，调用 `new LootItemRandomChanceWithEnchantedBonusCondition`​ 以进一步指定值。

---

### `minecraft:value_check`​

此条件接受一个数字提供器和一个 `IntRange`​，如果数字提供器的结果在范围内，则返回 `true`​。

```json
{
    "condition": "minecraft:value_check",
    // 可以是任何数字提供器
    "value": {
        "type": "minecraft:uniform",
        "min": 0.0,
        "max": 10.0
    },
    // 带有最小/最大值的范围
    "range": {
        "min": 2.0,
        "max": 5.0
    }
}
```

在数据生成期间，调用 `ValueCheckCondition#hasValue`​ 并传入数字提供器和范围，以构建此条件的构建器。

---

### `minecraft:time_check`​

此条件检查世界时间是否在 `IntRange`​ 内。可以选择提供一个 `period`​ 参数来对时间取模；例如，如果 `period`​ 为 24000（一个游戏日/夜周期有 24000 刻），则可以用于检查当前时间是否为白天。

```json
{
    "condition": "minecraft:time_check",
    // 可选，可以省略。如果省略，则不进行取模操作。
    // 我们使用 24000，这是一个游戏日/夜周期的长度。
    "period": 24000,
    // 带有最小/最大值的范围。此示例检查时间是否在 0 到 12000 之间。
    // 结合上面指定的 24000 取模操作，此示例检查当前是否为白天。
    "value": {
        "min": 0,
        "max": 12000
    }
}
```

在数据生成期间，调用 `TimeCheck#time`​ 并传入所需的范围，以构建此条件的构建器。然后可以使用 `#setPeriod`​ 在构建器上设置 `period`​ 值。

---

### `minecraft:weather_check`​

此条件检查当前天气是否为下雨或雷暴。

```json
{
    "condition": "minecraft:weather_check",
    // 可选。如果未指定，则不检查下雨状态。
    "raining": true,
    // 可选。如果未指定，则不检查雷暴状态。
    // 指定 "raining": true 和 "thundering": true 在功能上等同于仅指定
    // "thundering": true，因为雷暴发生时总是下雨。
    "thundering": false
}
```

在数据生成期间，调用 `WeatherCheck#weather`​ 以构建此条件的构建器。然后可以使用 `#setRaining`​ 和 `#setThundering`​ 在构建器上设置下雨和雷暴值。

---

### `minecraft:location_check`​

此条件接受一个 `LocationPredicate`​ 和每个轴方向的可选偏移值。`LocationPredicate`​ 允许检查位置本身、该位置的方块或流体状态、维度、生物群系或结构、光照等级、天空是否可见等条件。所有可能的值可以在 `LocationPredicate`​ 类定义中查看。需要 `minecraft:origin`​ 战利品参数，如果该参数不存在，则始终失败。

```json
{
    "condition": "minecraft:location_check",
    "predicate": {
        // 如果目标在下界，则成功
        "dimension": "the_nether"
    },
    // 可选的位置偏移值。仅在以某种方式检查位置时相关。
    // 必须同时提供所有偏移值，或者完全不提供。
    "offsetX": 10,
    "offsetY": 10,
    "offsetZ": 10
}
```

在数据生成期间，调用 `LocationCheck#checkLocation`​ 并传入 `LocationPredicate`​ 和可选的 `BlockPos`​，以构建此条件的构建器。

---

### `minecraft:block_state_property`​

此条件检查被破坏的方块状态是否具有指定的方块状态属性值。需要 `minecraft:block_state`​ 战利品参数，如果该参数不存在，则始终失败。

```json
{
    "condition": "minecraft:block_state_property",
    // 预期的方块。如果与实际破坏的方块不匹配，则条件失败。
    "block": "minecraft:oak_slab",
    // 要匹配的方块状态属性。未指定的属性可以具有任意值。
    // 在此示例中，我们只希望在被破坏的方块是顶部石板（无论是否含水）时成功。
    // 如果指定了方块上不存在的属性，则会打印日志警告。
    "properties": {
        "type": "top"
    }
}
```

在数据生成期间，调用 `LootItemBlockStatePropertyCondition#hasBlockStateProperties`​ 并传入方块，以构建此条件的构建器。然后可以使用 `#setProperties`​ 在构建器上设置所需的方块状态属性值。

---

### `minecraft:survives_explosion`​

此条件随机销毁掉落物。掉落物幸存的几率为 1 / `explosion_radius`​ 战利品参数。此函数用于所有方块掉落物，除了信标或龙蛋等极少数例外。需要 `minecraft:explosion_radius`​ 战利品参数，如果该参数不存在，则始终成功。

```json
{
    "condition": "minecraft:survives_explosion"
}
```

在数据生成期间，调用 `ExplosionCondition#survivesExplosion`​ 以构建此条件的构建器。

---

### `minecraft:match_tool`​

此条件接受一个 `ItemPredicate`​，用于检查 `tool`​ 战利品参数。`ItemPredicate`​ 可以指定有效的物品 ID 列表（`items`​）、物品数量的最小/最大范围（`count`​）、`DataComponentPredicate`​（`components`​）和 `ItemSubPredicate`​（`predicates`​）；所有字段都是可选的。需要 `minecraft:tool`​ 战利品参数，如果该参数不存在，则始终失败。

```json
{
    "condition": "minecraft:match_tool",
    // 匹配下界合金镐或斧
    "predicate": {
        "items": [
            "minecraft:netherite_pickaxe",
            "minecraft:netherite_axe"
        ]
    }
}
```

在数据生成期间，调用 `MatchTool#toolMatches`​ 并传入 `ItemPredicate.Builder`​，以构建此条件的构建器。

---

### `minecraft:enchantment_active`​

此条件返回附魔是否处于活动状态。需要 `minecraft:enchantment_active`​ 战利品参数，如果该参数不存在，则始终失败。

```json
{
    "condition": "minecraft:enchantment_active",
    // 附魔应处于活动状态（true）或不活动状态（false）
    "active": true
}
```

在数据生成期间，调用 `EnchantmentActiveCheck#enchantmentActiveCheck`​ 或 `#enchantmentInactiveCheck`​ 以构建此条件的构建器。

---

### `minecraft:table_bonus`​

此条件类似于 `minecraft:random_chance_with_enchanted_bonus`​，但使用固定值而不是随机值。需要 `minecraft:tool`​ 战利品参数，如果该参数不存在，则始终失败。

```json
{
    "condition": "minecraft:table_bonus",
    // 如果存在时运附魔，则应用加成
    "enchantment": "minecraft:fortune",
    // 每级使用的几率。此示例在未附魔时有 20% 的成功几率，
    // 1 级附魔时为 30%，2 级或以上时为 60%。
    "chances": [0.2, 0.3, 0.6]
}
```

在数据生成期间，调用 `BonusLevelTableCondition#bonusLevelFlatChance`​ 并传入附魔 ID 和几率，以构建此条件的构建器。

---

### `minecraft:entity_properties`​

此条件检查给定的 `EntityPredicate`​ 是否符合实体目标。`EntityPredicate`​ 可以检查实体类型、效果、NBT 值、装备、位置等。

```json
{
    "condition": "minecraft:entity_properties",
    // 要使用的实体目标。有效值为 "this"、"attacker"、"direct_attacker" 或 "attacking_player"。
    // 这些分别对应于 "this_entity"、"attacking_entity"、"direct_attacking_entity" 和
    // "last_damage_player" 战利品参数。
    "entity": "attacker",
    // 仅当目标是猪时成功。谓词也可以为空，这可以用于
    // 检查指定的实体目标是否已设置。
    "predicate": {
        "type": "minecraft:pig"
    }
}
```

在数据生成期间，调用 `LootItemEntityPropertyCondition#entityPresent`​ 并传入实体目标，或调用 `LootItemEntityPropertyCondition#hasProperties`​ 并传入实体目标和 `EntityPredicate`​，以构建此条件的构建器。

---

### `minecraft:damage_source_properties`​

此条件检查给定的 `DamageSourcePredicate`​ 是否符合 `damage_source`​ 战利品参数。需要 `minecraft:origin`​ 和 `minecraft:damage_source`​ 战利品参数，如果这些参数不存在，则始终失败。

```json
{
    "condition": "minecraft:damage_source_properties",
    "predicate": {
        // 检查源实体是否为僵尸
        "source_entity": {
            "type": "zombie"
        }
    }
}
```

在数据生成期间，调用 `DamageSourceCondition#hasDamageSource`​ 并传入 `DamageSourcePredicate.Builder`​，以构建此条件的构建器。

---

### `minecraft:killed_by_player`​

此条件确定击杀是否为玩家击杀。用于某些实体掉落物，例如烈焰人掉落的烈焰棒。需要 `minecraft:last_player_damage`​ 战利品参数，如果该参数不存在，则始终失败。

```json
{
    "condition": "minecraft:killed_by_player"
}
```

在数据生成期间，调用 `LootItemKilledByPlayerCondition#killedByPlayer`​ 以构建此条件的构建器。

---

### `minecraft:entity_scores`​

此条件检查实体目标的记分板。需要与指定实体目标对应的战利品参数，如果该参数不存在，则始终失败。

```json
{
    "condition": "minecraft:entity_scores",
    // 要使用的实体目标。有效值为 "this"、"attacker"、"direct_attacker" 或 "attacking_player"。
    // 这些分别对应于 "this_entity"、"attacking_entity"、"direct_attacking_entity" 和
    // "last_damage_player" 战利品参数。
    "entity": "attacker",
    // 必须在给定范围内的记分板值列表
    "scores": {
        "score1": {
            "min": 0,
            "max": 100
        },
        "score2": {
            "min": 10,
            "max": 20
        }
    }
}
```

在数据生成期间，调用 `EntityHasScoreCondition#hasScores`​ 并传入实体目标，以构建此条件的构建器。然后使用 `#withScore`​ 向构建器添加所需的记分板值。

---

### `minecraft:reference`​

此条件引用一个谓词文件并返回其结果。有关更多信息，请参阅 [物品谓词](https://minecraft.fandom.com/wiki/Predicate)。

```json
{
    "condition": "minecraft:reference",
    // 引用位于 data/examplemod/predicate/example_predicate.json 的谓词文件
    "name": "examplemod:example_predicate"
}
```

在数据生成期间，调用 `ConditionReference#conditionReference`​ 并传入引用的谓词文件的 ID，以构建此条件的构建器。

---

### `neoforge:loot_table_id`​

此条件仅在周围的战利品表 ID 匹配时返回 `true`​。通常用于全局战利品修改器中。

```json
{
    "condition": "neoforge:loot_table_id",
    // 仅在战利品表为泥土时应用
    "loot_table_id": "minecraft:blocks/dirt"
}
```

在数据生成期间，调用 `LootTableIdCondition#builder`​ 并传入所需的战利品表 ID，以构建此条件的构建器。

---

### `neoforge:can_item_perform_ability`​

此条件仅在 `tool`​ 战利品上下文参数（`LootContextParams.TOOL`​）中的物品（通常是用于破坏方块或杀死实体的物品）能够执行指定的 `ItemAbility`​ 时返回 `true`​。需要 `minecraft:tool`​ 战利品参数，如果该参数不存在，则始终失败。

```json
{
    "condition": "neoforge:can_item_perform_ability",
    // 仅在工具可以像斧头一样剥去皮时应用
    "ability": "axe_strip"
}
```

在数据生成期间，调用 `CanItemPerformAbility#canItemPerformAbility`​ 并传入所需的物品能力的 ID，以构建此条件的构建器。

---

### 另请参阅

* [Minecraft Wiki 上的物品谓词](https://minecraft.fandom.com/wiki/Predicate)
