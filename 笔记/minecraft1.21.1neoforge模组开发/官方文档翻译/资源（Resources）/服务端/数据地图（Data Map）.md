# 数据地图（Data Map）

### 数据地图（Data Maps）

数据地图包含数据驱动的、可重载的对象，这些对象可以附加到已注册的对象上。该系统使得游戏行为更容易通过数据驱动，因为它们提供了诸如同步或冲突解决等功能，从而带来更好且更可配置的用户体验。你可以将标签（tags）视为注册对象 ➜ 布尔值的映射，而数据地图则是更灵活的注册对象 ➜ 对象的映射。与标签类似，数据地图会添加到其对应的数据地图中，而不是覆盖它。

数据地图可以附加到静态的内置注册表（registries）和动态的数据驱动的数据包注册表（datapack registries）上。数据地图支持通过 `/reload`​ 命令或任何其他重新加载服务器资源的方式进行重载。

NeoForge 为常见用例提供了多种内置数据地图，取代了硬编码的原版字段。更多信息可以在相关文章中找到。

---

### 文件位置

数据地图从位于 `<mapNamespace>/data_maps/<registryNamespace>/<registryPath>/<mapPath>.json`​ 的 JSON 文件中加载，其中：

* ​`<mapNamespace>`​ 是数据地图 ID 的命名空间，
* ​`<mapPath>`​ 是数据地图 ID 的路径，
* ​`<registryNamespace>`​ 是注册表 ID 的命名空间（如果注册表是 `minecraft`​，则可以省略），
* ​`<registryPath>`​ 是注册表 ID 的路径。

**示例：**

* 对于名为 `mymod:drop_healing`​ 的数据地图，应用于 `minecraft:item`​ 注册表（如下例所示），路径将为 `mymod/data_maps/item/drop_healing.json`​。
* 对于名为 `somemod:somemap`​ 的数据地图，应用于 `minecraft:block`​ 注册表，路径将为 `somemod/data_maps/block/somemap.json`​。
* 对于名为 `example:stuff`​ 的数据地图，应用于 `somemod:custom`​ 注册表，路径将为 `example/data_maps/somemod/custom/stuff.json`​。

---

### JSON 结构

数据地图文件本身可能包含以下字段：

* ​**​`replace`​**​: 一个布尔值，表示在添加此文件的值之前清除数据地图。模组不应包含此字段，仅供数据包开发者用于覆盖此地图。
* ​**​`neoforge:conditions`​**​: 加载条件的列表。
* ​**​`values`​**​: 注册表 ID 或标签 ID 到值的映射，这些值将由你的模组添加到数据地图中。值本身的结构由数据地图的编解码器（codec）定义（见下文）。
* ​**​`remove`​**​: 要从数据地图中移除的注册表 ID 或标签 ID 的列表。

---

### 添加值

例如，假设我们有一个数据地图对象，包含两个浮点键 `amount`​ 和 `chance`​，应用于 `minecraft:item`​ 注册表。对应的数据地图文件可能如下所示：

```json
{
    "values": {
        // 为胡萝卜物品附加一个值
        "minecraft:carrot": {
            "amount": 12,
            "chance": 1
        },
        // 为 logs 标签中的所有物品附加一个值
        "#minecraft:logs": {
            "amount": 1,
            "chance": 0.1
        }
    }
}
```

数据地图可能支持合并器（mergers），这将在冲突情况下触发自定义合并行为（例如，如果两个模组为同一物品添加了数据地图值）。为了避免触发合并器，我们可以在元素级别指定 `replace`​ 字段，如下所示：

```json
{
    "values": {
        // 覆盖胡萝卜物品的值
        "minecraft:carrot": {
            "replace": true,
            // 新值将位于 value 子对象下
            "value": {
                "amount": 12,
                "chance": 1
            }
        }
    }
}
```

---

### 移除现有值

可以通过指定要移除的物品 ID 或标签 ID 列表来移除元素：

```json
{
    // 我们不希望土豆有值，即使其他模组的数据地图添加了它
    "remove": [
        "minecraft:potato"
    ]
}
```

移除操作在添加操作之后运行，因此我们可以先包含一个标签，然后再排除其中的某些元素：

```json
{
    "values": {
        "#minecraft:logs": { /* ... */ }
    },
    // 再次排除绯红菌柄
    "remove": [
        "minecraft:crimson_stem"
    ]
}
```

数据地图可能支持带有额外参数的自定义移除器（removers）。为了提供这些参数，`remove`​ 列表可以转换为一个 JSON 对象，其中包含要移除的元素作为键，额外数据作为关联值。例如，假设我们的移除器对象被序列化为字符串，那么我们的移除器映射可能如下所示：

```json
{
    "remove": {
        // 移除器将从值中反序列化（在此例中为 `somekey1`），
        // 并应用于附加到胡萝卜物品的值
        "minecraft:carrot": "somekey1"
    }
}
```

---

### 自定义数据地图

首先，我们定义数据地图条目的格式。数据地图条目必须是不可变的，因此记录（record）非常适合。沿用我们之前的示例，包含两个浮点值 `amount`​ 和 `chance`​，我们的数据地图条目将如下所示：

```java
public record ExampleData(float amount, float chance) {}
```

与许多其他内容一样，数据地图使用编解码器（codecs）进行序列化和反序列化。这意味着我们需要为数据地图条目提供一个编解码器：

```java
public record ExampleData(float amount, float chance) {
    public static final Codec<ExampleData> CODEC = RecordCodecBuilder.create(instance -> instance.group(
            Codec.FLOAT.fieldOf("amount").forGetter(ExampleData::amount),
            Codec.floatRange(0, 1).fieldOf("chance").forGetter(ExampleData::chance)
    ).apply(instance, ExampleData::new));
}
```

接下来，我们创建数据地图本身：

```java
// 在此示例中，我们为 minecraft:item 注册表注册数据地图，因此我们使用 Item 作为泛型。
// 如果要为其他注册表创建数据地图，请相应调整类型。
public static final DataMapType<Item, ExampleData> EXAMPLE_DATA = DataMapType.builder(
        // 数据地图的 ID。此数据地图的数据地图文件将位于
        // <yourmodid>:examplemod/data_maps/item/example_data.json。
        ResourceLocation.fromNamespaceAndPath("examplemod", "example_data"),
        // 注册数据地图的注册表。
        Registries.ITEM,
        // 数据地图条目的编解码器。
        ExampleData.CODEC
).build();
```

最后，在模组事件总线上注册数据地图：

```java
@SubscribeEvent
private static void registerDataMapTypes(RegisterDataMapTypesEvent event) {
    event.register(EXAMPLE_DATA);
}
```

---

### 同步

同步的数据地图将它们的值同步到客户端。可以通过在构建器上调用 `#synced`​ 来标记数据地图为同步的，如下所示：

```java
public static final DataMapType<Item, ExampleData> EXAMPLE_DATA = DataMapType.builder(...)
        .synced(
                // 用于同步的编解码器。可以与普通编解码器相同，也可以是
                // 字段较少的编解码器，省略客户端不需要的部分。
                ExampleData.CODEC,
                // 数据地图是否为强制性的。标记为强制性的数据地图将断开
                // 缺少该数据地图的客户端连接；这包括原版客户端。
                false
        ).build();
```

---

### 使用

由于数据地图可以用于任何注册表，因此必须通过 `Holder`​ 查询，而不是通过实际的注册表对象。此外，它仅适用于引用持有者（reference holders），而不是直接持有者（direct holders）。然而，大多数地方都会返回引用持有者，例如 `Registry#wrapAsHolder`​、`Registry#getHolder`​ 或不同的 `builtInRegistryHolder`​ 方法，因此在大多数情况下这不会成为问题。

然后，你可以通过 `Holder#getData(DataMapType)`​ 查询数据地图值。如果对象没有附加数据地图值，该方法将返回 `null`​。沿用之前的 `ExampleData`​，让我们在玩家拾取物品时使用它们来治疗玩家：

```java
@SubscribeEvent
private static void itemPickup(ItemPickupEvent event) {
    ItemStack stack = event.getItemStack();
    // 通过 ItemStack#getItemHolder 获取 Holder<Item>。
    Holder<Item> holder = stack.getItemHolder();
    // 从持有者中获取数据。
    ExampleData data = holder.getData(EXAMPLE_DATA);
    if (data != null) {
        // 值存在，因此我们可以使用它们！
        Player player = event.getPlayer();
        if (player.getLevel().getRandom().nextFloat() > data.chance()) {
            player.heal(data.amount());
        }
    }
}
```

当然，此过程也适用于 NeoForge 提供的所有数据地图。

---

### 高级数据地图

高级数据地图是使用 `AdvancedDataMapType`​ 而不是标准 `DataMapType`​ 的数据地图（`AdvancedDataMapType`​ 是 `DataMapType`​ 的子类）。它们具有一些额外的功能，即能够指定自定义合并器和自定义移除器。对于值类型为集合或类似集合（如列表或映射）的数据地图，强烈建议实现此功能。

虽然 `DataMapType`​ 有两个泛型 `R`​（注册表类型）和 `T`​（数据地图值类型），但 `AdvancedDataMapType`​ 还有一个额外的泛型：`VR extends DataMapValueRemover<R, T>`​。此泛型允许在数据生成时使用类型安全的移除器。

​`AdvancedDataMapType`​ 使用 `AdvancedDataMapType#builder()`​ 创建，而不是 `DataMapType#builder()`​，返回一个 `AdvancedDataMapType.Builder`​。此构建器有两个额外的方法 `#remover`​ 和 `#merger`​，分别用于指定移除器和合并器（见下文）。所有其他功能（包括同步）保持不变。

---

### 合并器

合并器可用于处理多个数据包尝试为同一对象添加值时的冲突。默认合并器（`DataMapValueMerger#defaultMerger`​）将用新值覆盖现有值（例如来自优先级较低的数据包的值），因此如果这不是所需的行为，则需要自定义合并器。

合并器将接收两个冲突的值，以及值附加到的对象（作为 `Either<TagKey<R>, ResourceKey<R>>`​，因为值可以附加到标签中的所有对象或单个对象）和对象的所属注册表，并应返回实际应附加的值。通常，合并器应简单地合并，而不是执行覆盖（即仅在正常合并方式不起作用时才覆盖）。如果数据包希望绕过合并器，则应在对象上指定 `replace`​ 字段（见“添加值”）。

让我们想象一个场景，其中我们有一个数据地图，将整数添加到物品中。我们可以通过将两个值相加来解决冲突，如下所示：

```java
public class IntMerger implements DataMapValueMerger<Item, Integer> {
    @Override
    public Integer merge(Registry<Item> registry,
            Either<TagKey<Item>, ResourceKey<Item>> first, Integer firstValue,
            Either<TagKey<Item>, ResourceKey<Item>> second, Integer secondValue) {
        return firstValue + secondValue;
    }
}
```

这样，如果一个数据包为 `minecraft:carrot`​ 指定值 `12`​，另一个数据包为 `minecraft:carrot`​ 指定值 `15`​，则 `minecraft:carrot`​ 的最终值将为 `27`​。如果其中任何一个对象指定了 `"replace": true`​，则使用该对象的值。如果两者都指定了 `"replace": true`​，则使用优先级较高的数据包的值。

最后，别忘了在构建器中指定合并器，如下所示：

```java
// 我们假设 AdvancedData 包含某种整数属性。
AdvancedDataMapType<Item, AdvancedData> ADVANCED_MAP = AdvancedDataMapType.builder(...)
        .merger(new IntMerger())
        .build();
```

**提示：**  NeoForge 在 `DataMapValueMerger`​ 中提供了列表、集合和映射的默认合并器。

---

### 移除器

与更复杂数据的合并器类似，移除器可用于正确处理元素的移除子句。默认移除器（`DataMapValueRemover.Default.INSTANCE`​）将简单地移除与指定对象相关的任何和所有信息，因此我们希望使用自定义移除器来仅移除对象数据的部分内容。

传递给构建器的编解码器（见下文）将用于解码移除器实例。移除器将接收当前附加到对象的值及其来源，并应返回一个 `Optional`​，其中包含要替换旧值的值。或者，返回空的 `Optional`​ 将导致值被实际移除。

考虑以下示例，一个移除器将从基于 `Map<String, String>`​ 的数据地图中移除具有特定键的值：

```java
public record MapRemover(String key) implements DataMapValueRemover<Item, Map<String, String>> {
    public static final Codec<MapRemover> CODEC = Codec.STRING.xmap(MapRemover::new, MapRemover::key);
    
    @Override
    public Optional<Map<String, String>> remove(Map<String, String> value, Registry<Item> registry, Either<TagKey<Item>, ResourceKey<Item>> source, Item object) {
        final Map<String, String> newMap = new HashMap<>(value);
        newMap.remove(key);
        return Optional.of(newMap);
    }
}
```

考虑到此移除器，考虑以下数据文件：

```json
{
    "values": {
        "minecraft:carrot": {
            "somekey1": "value1",
            "somekey2": "value2"
        }
    }
}
```

现在，考虑此第二个数据文件，其优先级高于第一个文件：

```json
{
    "remove": {
        // 由于移除器被解码为字符串，我们可以在此处使用字符串作为值。
        // 如果它被解码为对象，则需要使用对象。
        "minecraft:carrot": "somekey1"
    }
}
```

这样，在应用两个文件后，最终结果将是（内存中的表示）：

```json
{
    "values": {
        "minecraft:carrot": {
            "somekey1": "value1"
        }
    }
}
```

与合并器一样，别忘了将它们添加到构建器中。注意，我们在此处简单地使用编解码器：

```java
// 我们假设 AdvancedData 包含某种 Map<String, String> 属性。
AdvancedDataMapType<Item, AdvancedData> ADVANCED_MAP = AdvancedDataMapType.builder(...)
        .remover(MapRemover.CODEC)
        .build();
```

---

### 数据生成

数据地图可以通过扩展 `DataMapProvider`​ 并重写 `#gather`​ 来生成数据。沿用之前的 `ExampleData`​（包含浮点值 `amount`​ 和 `chance`​），我们的数据生成文件可能如下所示：

```java
public class MyDataMapProvider extends DataMapProvider {
    public MyDataMapProvider(PackOutput packOutput, CompletableFuture<HolderLookup.Provider> lookupProvider) {
        super(packOutput, lookupProvider);
    }
    
    @Override
    protected void gather() {
        // 我们为 EXAMPLE_DATA 数据地图创建一个构建器，并使用 #add 添加条目。
        builder(EXAMPLE_DATA)
                // 我们启用替换。永远不要以这种方式发布模组！这仅用于教育目的。
                .replace(true)
                // 我们为所有台阶添加值 "amount": 10, "chance": 1。布尔参数控制
                // "replace" 字段，在模组中应始终为 false。
                .add(ItemTags.SLABS, new ExampleData(10, 1), false)
                // 我们为苹果添加值 "amount": 5, "chance": 0.2。
                .add(Items.APPLE.builtInRegistryHolder(), new ExampleData(5, 0.2f), false) // 也可以使用 Registry#wrapAsHolder 获取注册表对象的持有者
                // 我们再次移除木质台阶。
                .remove(ItemTags.WOODEN_SLABS)
                // 我们为 Botania 添加一个模组加载条件，因为为什么不呢。
                .conditions(new ModLoadedCondition("botania"));
    }
}
```

这将生成以下 JSON 文件：

```json
{
    "replace": true,
    "values": {
        "#minecraft:slabs": {
            "amount": 10,
            "chance": 1.0
        },
        "minecraft:apple": {
            "amount": 5,
            "chance": 0.2
        }
    },
    "remove": [
        "#minecraft:wooden_slabs"
    ],
    "neoforge:conditions": [
        {
            "type": "neoforge:mod_loaded",
            "modid": "botania"
        }
    ]
}
```

与所有数据提供器一样，别忘了将提供器添加到事件中：

```java
@SubscribeEvent
public static void gatherData(GatherDataEvent event) {
    DataGenerator generator = event.getGenerator();
    PackOutput output = generator.getPackOutput();
    CompletableFuture<HolderLookup.Provider> lookupProvider = event.getLookupProvider();

    // 其他提供器在这里
    generator.addProvider(
            event.includeServer(),
            new MyDataMapProvider(output, lookupProvider)
    );
}
```
