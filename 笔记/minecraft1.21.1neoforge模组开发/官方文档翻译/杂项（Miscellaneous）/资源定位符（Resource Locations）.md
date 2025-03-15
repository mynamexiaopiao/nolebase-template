# 资源定位符（Resource Locations）

### 资源定位符（Resource Locations）

​`ResourceLocation`​ 是 Minecraft 中最重要的概念之一。它们被用作注册表的键、数据或资源文件的标识符、代码中模型的引用，以及许多其他用途。`ResourceLocation`​ 由两部分组成：命名空间（namespace）和路径（path），用 `:`​ 分隔。

* **命名空间**：表示资源所属的模组、资源包或数据包。例如，模组 ID 为 `examplemod`​ 的模组将使用 `examplemod`​ 命名空间。Minecraft 使用 `minecraft`​ 命名空间。可以通过创建相应的数据文件夹来定义额外的命名空间，这通常由数据包完成，以将其逻辑与与 Minecraft 集成的部分分开。
* **路径**：在命名空间内引用你想要的任何对象。例如，`minecraft:cow`​ 是 `minecraft`​ 命名空间中名为 `cow`​ 的对象的引用——通常此位置用于从实体注册表中获取牛实体。另一个例子是 `examplemod:example_item`​，它可能用于从物品注册表中获取你的模组的 `example_item`​。

​`ResourceLocation`​ 只能包含小写字母、数字、下划线、点和连字符。路径还可以包含正斜杠。需要注意的是，由于 Java 模块的限制，模组 ID 不能包含连字符，这意味着模组命名空间也不能包含连字符（路径中仍然允许）。

**注意**：
`ResourceLocation`​ 本身并不说明我们将其用于什么类型的对象。例如，名为 `minecraft:dirt`​ 的对象存在于多个地方。接收 `ResourceLocation`​ 的对象负责将其与特定对象关联。

可以通过以下方式创建新的 `ResourceLocation`​：

* ​`ResourceLocation.fromNamespaceAndPath("examplemod", "example_item")`​
* ​`ResourceLocation.parse("examplemod:example_item")`​

如果使用 `withDefaultNamespace`​，字符串将用作路径，`minecraft`​ 将用作命名空间。例如，`ResourceLocation.withDefaultNamespace("example_item")`​ 将生成 `minecraft:example_item`​。

可以通过 `ResourceLocation#getNamespace()`​ 和 `#getPath()`​ 分别获取命名空间和路径，通过 `ResourceLocation#toString`​ 获取组合形式。

​`ResourceLocation`​ 是不可变的。`ResourceLocation`​ 上的所有实用方法（如 `withPrefix`​ 或 `withSuffix`​）都会返回一个新的 `ResourceLocation`​。

---

#### 解析 ResourceLocation

某些地方（例如注册表）直接使用 `ResourceLocation`​。然而，其他地方会根据需要解析 `ResourceLocation`​。例如：

1. **GUI 背景**：
    `ResourceLocation`​ 用作 GUI 背景的标识符。例如，熔炉 GUI 使用资源定位符 `minecraft:textures/gui/container/furnace.png`​。这映射到磁盘上的文件 `assets/minecraft/textures/gui/container/furnace.png`​。注意，此资源定位符需要 `.png`​ 后缀。
2. **方块模型**：
    `ResourceLocation`​ 用作方块模型的标识符。例如，泥土的方块模型使用资源定位符 `minecraft:block/dirt`​。这映射到磁盘上的文件 `assets/minecraft/models/block/dirt.json`​。注意，此资源定位符不需要 `.json`​ 后缀，并且会自动映射到 `models`​ 子文件夹。
3. **配方**：
    `ResourceLocation`​ 用作配方的标识符。例如，铁块的合成配方使用资源定位符 `minecraft:iron_block`​。这映射到磁盘上的文件 `data/minecraft/recipes/iron_block.json`​。注意，此资源定位符不需要 `.json`​ 后缀，并且会自动映射到 `recipes`​ 子文件夹。

​`ResourceLocation`​ 是否需要文件后缀，或者它具体解析为什么，取决于使用场景。

---

#### 模型资源定位符（ModelResourceLocation）

​`ModelResourceLocation`​ 是一种特殊的资源定位符，包含第三部分，称为变体（variant）。Minecraft 主要使用这些来区分模型的不同变体，这些变体在不同的显示上下文中使用（例如三叉戟，它在第一人称、第三人称和物品栏中有不同的模型）。对于物品，变体始终为 `inventory`​；对于方块状态，变体是属性-值对的逗号分隔字符串（例如 `facing=north,waterlogged=false`​，对于没有方块状态属性的方块则为空）。

变体附加到常规资源定位符后，并用 `#`​ 分隔。例如，钻石剑的物品模型的完整名称是 `minecraft:diamond_sword#inventory`​。然而，在大多数上下文中，`inventory`​ 变体可以省略。

​`ModelResourceLocation`​ 是一个仅客户端的类。这意味着服务器引用此类时会抛出 `NoClassDefFoundError`​ 并崩溃。

---

#### 资源键（ResourceKey）

​`ResourceKey`​ 将注册表 ID 与注册表名称结合在一起。例如，一个注册表键可能包含注册表 ID `minecraft:item`​ 和注册表名称 `minecraft:diamond_sword`​。与 `ResourceLocation`​ 不同，`ResourceKey`​ 实际上引用了一个唯一的元素，因此能够明确标识一个元素。它们最常用于多个不同注册表相互接触的上下文中。一个常见的用例是数据包，尤其是世界生成。

可以通过静态方法 `ResourceKey#create(ResourceKey<? extends Registry<T>>, ResourceLocation)`​ 创建新的 `ResourceKey`​。第二个参数是注册表名称，而第一个参数是注册表键。注册表键是一种特殊的 `ResourceKey`​，其注册表是根注册表（即所有其他注册表的注册表）。可以通过 `ResourceKey#createRegistryKey(ResourceLocation)`​ 创建注册表键，并指定所需注册表的 ID。

​`ResourceKey`​ 在创建时被内部化（interned）。这意味着可以通过引用相等性（`==`​）进行比较，并且鼓励这样做，但它们的创建相对较昂贵。
