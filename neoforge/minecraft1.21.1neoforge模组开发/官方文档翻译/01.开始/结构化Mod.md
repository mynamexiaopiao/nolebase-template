# 结构化Mod

文章翻译自官方文档：

### 结构化 Mod 的好处

结构化的 Mod 有助于维护、贡献和提供对底层代码库的更清晰理解。以下是来自 Java、Minecraft 和 NeoForge 的一些建议。

**注意**：您不必遵循以下建议；您可以按照您认为合适的方式构建您的 Mod。然而，仍然强烈建议这样做。

### 包结构

在构建您的 Mod 时，选择一个唯一的顶级包结构。许多程序员会为不同的类、接口等使用相同的名称。Java 允许类具有相同的名称，只要它们位于不同的包中。因此，如果两个类具有相同的包和相同的名称，则只会加载其中一个，很可能会导致游戏崩溃。

```plaintext
a.jar
    - com.example.ExampleClass
b.jar
    - com.example.ExampleClass // 这个类通常不会被加载
```

在加载模块时，这一点更为重要。如果两个模块中有相同名称的包，这将导致 Mod 加载器在启动时崩溃，因为 Mod 模块会导出到游戏和其他 Mod。

```plaintext
module A
    - package X
        - class I
        - class J
module B
    - package X // 这个包会导致 Mod 加载器崩溃，因为已经有一个模块导出了包 X
        - class R
        - class S
        - class T
```

因此，您的顶级包应该是您拥有的东西：域名、电子邮件地址、网站（或子域名）等。它甚至可以是您的名字或用户名，只要您能保证它在预期目标中是唯一可识别的。此外，顶级包还应与您的 Group ID 匹配。

|类型|值|顶级包|
| --------| -----------------| -----------------|
|域名|example.com|com.example|
|子域名|example.github.io|io.github.example|
|电子邮件|example@gmail.com|com.gmail.example|

下一级包应该是您的 Mod ID（例如 `com.example.examplemod`，其中 `examplemod` 是 Mod ID）。这将确保除非您有两个具有相同 ID 的 Mod（这不应该发生），否则您的包应该不会有任何加载问题。

您可以在 [Oracle 的教程页面](#) 上找到一些额外的命名约定。

### 子包组织

除了顶级包之外，强烈建议将您的 Mod 的类分解到子包中。有两种主要方法可以做到这一点：

1. **按功能分组**：为具有共同目的的类创建子包。例如，方块可以放在 `block` 下，物品放在 `item` 下，实体放在 `entity` 下等。Minecraft 本身使用了类似的结构（有一些例外）。
2. **按逻辑分组**：为具有共同逻辑的类创建子包。例如，如果您正在创建一个新的工作台类型，您会将其方块、菜单、物品等放在 `feature.crafting_table` 下。

### 客户端、服务器和数据包

通常，仅用于特定端或运行时的代码应与其它类隔离，放在单独的子包中。例如，与数据生成相关的代码应放在 `data` 包中，而仅在专用服务器上运行的代码应放在 `server` 包中。

强烈建议将仅客户端的代码隔离在 `client` 子包中。这是因为专用服务器无法访问 Minecraft 中的任何仅客户端包，如果您的 Mod 尝试访问它们，服务器将崩溃。因此，拥有一个专用的包可以提供一个良好的检查机制，以确保您不会在 Mod 中跨端访问。

### 类命名方案

常见的类命名方案使得更容易理解类的用途或轻松定位特定类。

类通常以其类型为后缀，例如：

- 一个名为 `PowerRing` 的物品 -> `PowerRingItem`。
- 一个名为 `NotDirt` 的方块 -> `NotDirtBlock`。
- 一个用于 `Oven` 的菜单 -> `OvenMenu`。

**提示**：Mojang 通常对所有类（除了实体）遵循类似的结构。实体类仅由其名称表示（例如 `Pig`、`Zombie` 等）。

### 从多种方法中选择一种

执行某些任务的方法有很多：注册对象、监听事件等。通常建议通过使用单一方法来完成任务，以保持一致性。这提高了可读性，并避免了可能发生的奇怪交互或冗余（例如，您的事件监听器运行两次）。
