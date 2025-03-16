# 内置附魔效果组件（Built-In Enchantment Effect Components）

### 内置附魔效果组件（Built-In Enchantment Effect Components）

Minecraft 原版提供了多种不同类型的附魔效果组件，用于附魔定义。本文将解释每种组件的用途及其代码定义。

---

### 值效果组件（Value Effect Components）

值效果组件用于修改游戏中某个数值的附魔，由 `EnchantmentValueEffect` 类实现。如果一个值被多个值效果组件修改（例如多个附魔），所有效果都会叠加。

值效果组件可以使用以下操作来修改给定的值：

- **​`minecraft:set`​**: 覆盖给定的基于等级的值。
- **​`minecraft:add`​**: 将指定的基于等级的值添加到旧值上。
- **​`minecraft:multiply`​**: 将指定的基于等级的系数乘以旧值。
- **​`minecraft:remove_binomial`​**: 使用二项分布对给定的（基于等级的）概率进行采样。如果成功，则从值中减去 1。注意，许多值实际上是标志，1 表示完全开启，0 表示完全关闭。
- **​`minecraft:all_of`​**: 接受一个值效果列表，并按顺序应用它们。

例如，**锋利（Sharpness）** 附魔使用 `minecraft:damage` 值效果组件来实现其效果：

```json
"effects": {
    // 该效果组件的类型为 "minecraft:damage"。
    // 这意味着效果将修改武器伤害。
    "minecraft:damage": [
        {
            // 要应用的值效果。
            "effect": {
                // 值效果的类型。此处为 "minecraft:add"，因此值（见下文）将添加到武器伤害值上。
                "type": "minecraft:add",

                // 值块。此处值为 LevelBasedValue，初始为 1，每级增加 0.5。
                "value": {
                    "type": "minecraft:linear",
                    "base": 1.0,
                    "per_level_above_first": 0.5
                }
            }
        }
    ]
}
```

`value` 块中的对象是一个 `LevelBasedValue`，可用于使值效果组件的效果强度随等级变化。

`EnchantmentValueEffect#process` 方法可用于根据提供的数值操作调整值，例如：

```java
// `valueEffect` 是一个 EnchantmentValueEffect 实例。
// `enchantLevel` 是表示附魔等级的整数。
float baseValue = 1.0;
float modifiedValue = valueEffect.process(enchantLevel, server.random, baseValue);
```

---

### 原版附魔值效果组件类型

#### 定义为 `DataComponentType<EnchantmentValueEffect>`

- **​`minecraft:crossbow_charge_time`​**: 修改弩的蓄力时间（秒）。用于**快速装填（Quick Charge）** 。
- **​`minecraft:trident_spin_attack_strength`​**: 修改三叉戟旋转攻击的“强度”（见 `TridentItem#releaseUsing`）。用于**激流（Riptide）** 。

#### 定义为 `DataComponentType<List<ConditionalEffect<EnchantmentValueEffect>>>`

- **护甲相关**:

  - **​`minecraft:armor_effectiveness`​**: 决定护甲对武器的有效性，范围为 0（无保护）到 1（正常保护）。用于**穿透（Breach）** 。
  - **​`minecraft:damage_protection`​**: 每点伤害减免减少 4% 的伤害，最多减免 80%。用于**爆炸保护（Blast Protection）** 、**摔落保护（Feather Falling）** 、**火焰保护（Fire Protection）** 、**保护（Protection）**​****和****​**弹射物保护（Projectile Protection）** 。
- **攻击相关**:

  - **​`minecraft:damage`​**: 修改武器的攻击伤害。用于**锋利（Sharpness）** 、**穿刺（Impaling）** 、**节肢杀手（Bane of Arthropods）** 、**力量（Power）**​****和****​**亡灵杀手（Smite）** 。
  - **​`minecraft:smash_damage_per_fallen_block`​**: 为锤子增加每下落一个方块的伤害。用于**密度（Density）** 。
  - **​`minecraft:knockback`​**: 修改武器造成的击退距离（以游戏单位计）。用于**击退（Knockback）**​****和****​**冲击（Punch）** 。
  - **​`minecraft:mob_experience`​**: 修改击杀生物获得的经验值。未使用。
- **耐久相关**:

  - **​`minecraft:item_damage`​**: 修改物品的耐久损耗。值小于 1 时表示物品有概率不损耗耐久。用于**耐久（Unbreaking）** 。
  - **​`minecraft:repair_with_xp`​**: 使物品通过经验修复，并决定修复效果。用于**经验修补（Mending）** 。
- **弹射物相关**:

  - **​`minecraft:ammo_use`​**: 修改弓或弩发射时消耗的弹药量。值被限制为整数，小于 1 时表示不消耗弹药。用于**无限（Infinity）** 。
  - **​`minecraft:projectile_piercing`​**: 修改武器发射的弹射物穿透的实体数量。用于**穿透（Piercing）** 。
  - **​`minecraft:projectile_count`​**: 修改弓发射时生成的弹射物数量。用于**多重射击（Multishot）** 。
  - **​`minecraft:projectile_spread`​**: 修改弹射物发射时的最大散布角度（度）。用于**多重射击（Multishot）** 。
  - **​`minecraft:trident_return_acceleration`​**: 使三叉戟返回持有者，并修改返回时的加速度。用于**忠诚（Loyalty）** 。
- **其他**:

  - **​`minecraft:block_experience`​**: 修改破坏方块获得的经验值。用于**精准采集（Silk Touch）** 。
  - **​`minecraft:fishing_time_reduction`​**: 减少钓鱼时浮标下沉所需的时间（秒）。用于**海之眷顾（Lure）** 。
  - **​`minecraft:fishing_luck_bonus`​**: 修改钓鱼战利品表中使用的幸运值。用于**海之眷顾（Luck of the Sea）** 。

#### 定义为 `DataComponentType<List<TargetedConditionalEffect<EnchantmentValueEffect>>>`

- **​`minecraft:equipment_drops`​**: 修改被该武器击杀的生物掉落装备的概率。用于**抢夺（Looting）** 。

---

### 基于位置的效果组件（Location Based Effect Components）

基于位置的效果组件实现了 `EnchantmentLocationBasedEffect`。这些组件定义了需要知道附魔持有者在世界中位置的操作。它们使用两个主要方法：`EnchantmentEntityEffect#onChangedBlock`（在附魔物品被装备或持有者位置变化时调用）和 `onDeactivate`（在附魔物品被移除时调用）。

以下示例使用 `minecraft:attributes` 基于位置的效果组件类型来修改持有者的实体大小：

```json
"minecraft:attributes": [
    {
        // "amount" 块是一个 LevelBasedValue。
        "amount": {
            "type": "minecraft:linear",
            "base": 1,
            "per_level_above_first": 1
        },

        // 要修改的属性。此处为 "minecraft:scale"。
        "attribute": "minecraft:generic.scale",
        // 该属性修饰符的唯一标识符。不应与其他修饰符重叠，但无需注册。
        "id": "examplemod:enchantment.size_change",
        // 对属性使用的操作。可以是 "add_value"、"add_multiplied_base" 或 "add_multiplied_total"。
        "operation": "add_value"
    }
]
```

原版添加了以下基于位置的事件：

- **​`minecraft:all_of`​**: 按顺序运行一系列实体效果。
- **​`minecraft:apply_mob_effect`​**: 对受影响的生物应用状态效果。
- **​`minecraft:attribute`​**: 对附魔持有者应用属性修饰符。
- **​`minecraft:damage_entity`​**: 对受影响的实体造成伤害。如果在攻击上下文中，则与攻击伤害叠加。
- **​`minecraft:damage_item`​**: 损耗物品的耐久。
- **​`minecraft:explode`​**: 召唤爆炸。
- **​`minecraft:ignite`​**: 点燃实体。
- **​`minecraft:play_sound`​**: 播放指定的声音。
- **​`minecraft:replace_block`​**: 替换给定偏移处的方块。
- **​`minecraft:replace_disk`​**: 替换一个方块圆盘。
- **​`minecraft:run_function`​**: 运行指定的数据包函数。
- **​`minecraft:set_block_properies`​**: 修改指定方块的方块状态属性。
- **​`minecraft:spawn_particles`​**: 生成粒子。
- **​`minecraft:summon_entity`​**: 召唤实体。

---

### 实体效果组件（Entity Effect Components）

实体效果组件实现了 `EnchantmentEntityEffect`，是 `EnchantmentLocationBasedEffect` 的子类型。这些组件重写 `EnchantmentLocationBasedEffect#onChangedBlock` 以运行 `EnchantmentEntityEffect#apply`；此 `apply` 方法也会在代码库的其他地方直接调用，具体取决于组件的类型。这使得效果可以在不等待持有者位置变化的情况下发生。

所有基于位置的效果组件类型也适用于实体效果组件，除了 `minecraft:attribute`，它仅注册为基于位置的效果组件。

以下示例展示了**火焰附加（Fire Aspect）** 附魔的 JSON 定义：

```json
"minecraft:post_attack": [
    {
        // 决定攻击的“受害者”、“攻击者”或“伤害实体”（如果有弹射物则为弹射物，否则为攻击者）是否受到效果影响。
        "affected": "victim",
        
        // 决定应用哪种实体效果。
        "effect": {
            // 该效果的类型为 "minecraft:ignite"。
            "type": "minecraft:ignite",
            // "minecraft:ignite" 需要一个 LevelBasedValue 作为实体被点燃的持续时间。
            "duration": {
                "type": "minecraft:linear",
                "base": 4.0,
                "per_level_above_first": 4.0
            }
        },

        // 决定谁（“受害者”、“攻击者”或“伤害实体”）必须拥有附魔才能生效。
        "enchanted": "attacker",

        // 可选的条件，控制效果是否生效。
        "requirements": {
            "condition": "minecraft:damage_source_properties",
            "predicate": {
                "is_direct": true
            }
        }
    }
]
```

---

### 其他原版附魔组件类型

- **​`minecraft:damage_immunity`​**: 对指定伤害类型应用免疫。用于**冰霜行者（Frost Walker）** 。
- **​`minecraft:prevent_equipment_drop`​**: 防止玩家死亡时掉落该物品。用于**消失诅咒（Curse of Vanishing）** 。
- **​`minecraft:prevent_armor_change`​**: 防止从护甲槽中卸下该物品。用于**绑定诅咒（Curse of Binding）** 。
- **​`minecraft:crossbow_charge_sounds`​**: 决定弩蓄力时的音效事件。每个条目代表附魔的一个等级。
- **​`minecraft:trident_sound`​**: 决定使用三叉戟时的音效事件。每个条目代表附魔的一个等级。

更多详细信息，请参阅 Minecraft Wiki 的相关页面。
