# 数据加载条件（Data Load Conditions）

### 数据加载条件

有时，希望在另一个 Mod 存在时禁用或启用某些功能，或者如果有任何 Mod 添加了另一种类型的矿石等。对于这些用例，NeoForge 添加了数据加载条件。这些条件最初被称为配方条件，因为配方是该系统的原始用例，但此后已扩展到其他系统。这也是为什么一些内置条件仅限于物品。

大多数 JSON 文件可以选择在根目录中声明一个块，该块将在实际加载数据文件之前进行评估。当且仅当所有条件都通过时，加载才会继续，否则数据文件将被忽略。（此规则的例外是战利品表，它们将被替换为空战利品表。）

```json
{
    "neoforge:conditions": [
        {
            // 条件 1
        },
        {
            // 条件 2
        },
        // ...
    ],
    // 数据文件的其余部分
}
```

例如，如果我们希望仅在 Mod ID 为 `examplemod` 的 Mod 存在时加载我们的文件，我们的文件将如下所示：

```json
{
    "neoforge:conditions": [
        {
            "type": "neoforge:mod_loaded",
            "modid": "examplemod"
        }
    ],
    "type": "minecraft:crafting_shaped",
    // ...
}
```

**注意**：大多数原版文件已使用 `ConditionalCodec` 包装器进行了修补以使用条件。然而，并非所有系统，尤其是那些不使用编解码器的系统，都可以使用条件。要确定数据文件是否可以使用条件，请检查支持的编解码器定义。

### 内置条件

#### `neoforge:true` 和 `neoforge:false`

这些条件不包含数据，并返回预期值。

```json
{
    // 将始终返回 true（对于 "neoforge:false" 返回 false）
    "type": "neoforge:true"
}
```

**提示**：使用 `neoforge:false` 条件可以非常干净地禁用任何数据文件。只需在所需位置放置一个包含以下内容的文件：

```json
{"neoforge:conditions":[{"type":"neoforge:false"}]}
```

以这种方式禁用文件不会导致日志垃圾信息。

#### `neoforge:not`

此条件接受另一个条件并对其进行反转。

```json
{
    // 反转存储条件的结果
    "type": "neoforge:not",
    "value": {
        // 另一个条件
    }
}
```

#### `neoforge:and` 和 `neoforge:or`

这些条件接受被操作的条件，并应用预期的逻辑。接受的条件数量没有限制。

```json
{
    // 将存储的条件进行 AND 运算（或对 "neoforge:or" 进行 OR 运算）
    "type": "neoforge:and",
    "values": [
        {
            // 第一个条件
        },
        {
            // 第二个条件
        }
    ]
}
```

#### `neoforge:mod_loaded`

如果加载了具有给定 Mod ID 的 Mod，则此条件返回 `true`，否则返回 `false`。

```json
{
    "type": "neoforge:mod_loaded",
    // 如果加载了 "examplemod"，则返回 true
    "modid": "examplemod"
}
```

#### `neoforge:item_exists`

如果注册了具有给定注册名称的物品，则此条件返回 `true`，否则返回 `false`。

```json
{
    "type": "neoforge:item_exists",
    // 如果注册了 "examplemod:example_item"，则返回 true
    "item": "examplemod:example_item"
}
```

#### `neoforge:tag_empty`

如果给定的物品标签为空，则此条件返回 `true`，否则返回 `false`。

```json
{
    "type": "neoforge:tag_empty",
    // 如果 "examplemod:example_tag" 是一个空的物品标签，则返回 true
    "tag": "examplemod:example_tag"
}
```

### 创建自定义条件

可以通过实现 `ICondition` 及其 `#test(IContext)` 方法以及为其创建映射编解码器来创建自定义条件。`IContext` 参数可以访问游戏状态的某些部分。目前，这仅允许您从注册表中查询标签。某些带有条件的对象可能比标签更早加载，在这种情况下，上下文将是 `IContext.EMPTY`，并且不包含任何标签信息。

例如，假设我们想重新实现 `tag_empty` 条件，但用于实体类型标签而不是物品标签，那么我们的条件将如下所示：

```java
// 这个类基本上是 TagEmptyCondition 的精简版，调整为用于实体类型而不是物品。
public record EntityTagEmptyCondition(TagKey<EntityType<?>> tag) implements ICondition {
    public static final MapCodec<EntityTagEmptyCondition> CODEC = RecordCodecBuilder.mapCodec(inst -> inst.group(
            ResourceLocation.CODEC.xmap(rl -> TagKey.create(Registries.ENTITY_TYPES, rl), TagKey::location).fieldOf("tag").forGetter(EntityTagEmptyCondition::tag)
    ).apply(inst, EntityTagEmptyCondition::new));

    @Override
    public boolean test(ICondition.IContext context) {
        return context.getTag(this.tag()).isEmpty();
    }

    @Override
    public MapCodec<? extends ICondition> codec() {
        return CODEC;
    }
}
```

条件是一个编解码器注册表。因此，我们需要注册我们的编解码器，如下所示：

```java
public static final DeferredRegister<MapCodec<? extends ICondition>> CONDITION_CODECS =
        DeferredRegister.create(NeoForgeRegistries.Keys.CONDITION_CODECS, ExampleMod.MOD_ID);

public static final Supplier<MapCodec<EntityTagEmptyCondition>> ENTITY_TAG_EMPTY =
        CONDITION_CODECS.register("entity_tag_empty", () -> EntityTagEmptyCondition.CODEC);
```

然后，我们可以在某些数据文件中使用我们的条件（假设我们在 `examplemod` 命名空间下注册了条件）：

```json
{
    "neoforge:conditions": [
        {
            "type": "examplemod:entity_tag_empty",
            "tag": "minecraft:zombies"
        }
    ],
    // 数据文件的其余部分
}
```

### 数据生成

虽然任何数据包 JSON 文件都可以使用加载条件，但只有少数数据生成器已被修改为能够生成它们。这些包括：

- `RecipeProvider`（通过 `RecipeOutput#withConditions`），包括配方进度
- `JsonCodecProvider` 及其子类 `SpriteSourceProvider`
- `DataMapProvider`
- `GlobalLootModifierProvider`

对于条件本身，`IConditionBuilder` 接口为每种内置条件类型提供了静态帮助程序，返回相应的 `ICondition`。
