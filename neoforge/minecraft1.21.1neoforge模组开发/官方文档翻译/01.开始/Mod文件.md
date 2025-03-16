# Mod文件

文章翻译自官方文档：

### Mod 文件

Mod 文件负责确定哪些 Mod 被打包到您的 JAR 文件中，在“Mods”菜单中显示哪些信息，以及您的 Mod 应如何在游戏中加载。

### gradle.properties

该文件保存了您的 Mod 的各种常见属性，例如 Mod ID 或 Mod 版本。在构建过程中，Gradle 会读取这些文件中的值，并将它们内联到各种地方，例如 `neoforge.mods.toml` 文件。这样，您只需在一个地方更改值，它们就会自动应用到所有相关的地方。

大多数值在 MDK 的 `gradle.properties` 文件中也有注释说明。

|属性|描述|示例|
| ----| ------------------------------------------------------------------------------------------------------------------------------------------------| ----|
|`org.gradle.jvmargs`|允许您向 Gradle 传递额外的 JVM 参数。通常用于为 Gradle 分配更多/更少的内存。请注意，这是针对 Gradle 本身的，而不是 Minecraft。|`org.gradle.jvmargs=-Xmx3G`|
|`org.gradle.daemon`|Gradle 是否应在构建时使用守护进程。|`org.gradle.daemon=false`|
|`org.gradle.debug`|Gradle 是否设置为调试模式。调试模式主要意味着更多的 Gradle 日志输出。请注意，这是针对 Gradle 本身的，而不是 Minecraft。|`org.gradle.debug=false`|
|`minecraft_version`|您正在 Mod 的 Minecraft 版本。必须与 `.neo_version` 匹配。|`minecraft_version=1.20.6`|
|`minecraft_version_range`|此 Mod 可以使用的 Minecraft 版本范围，作为 Maven 版本范围。请注意，快照版、预发布版和候选发布版不能保证正确排序，因为它们不遵循 Maven 版本控制。|`minecraft_version_range=[1.20.6,1.21)`|
|`neo_version`|您正在 Mod 的 NeoForge 版本。必须与 `minecraft_version` 匹配。有关 NeoForge 版本控制的工作原理，请参阅 [NeoForge 版本控制](assets/3.版本控制-20250315093053-z0qccsa.md)。|`neo_version=20.6.62`|
|`neo_version_range`|此 Mod 可以使用的 NeoForge 版本范围，作为 Maven 版本范围。|`neo_version_range=[20.6.62,20.7)`|
|`loader_version_range`|此 Mod 可以使用的 Mod 加载器版本范围，作为 Maven 版本范围。请注意，加载器版本控制与 NeoForge 版本控制是分离的。|`loader_version_range=[1,)`|
|`mod_id`|参见 [Mod ID](#)。|`mod_id=examplemod`|
|`mod_name`|您的 Mod 的可读显示名称。默认情况下，这只能在 Mod 列表中看到，但像 JEI 这样的 Mod 也会在物品提示中显眼地显示 Mod 名称。|`mod_name=Example Mod`|
|`mod_license`|您的 Mod 提供的许可证。建议将其设置为您使用的 SPDX 标识符和/或许可证的链接。您可以访问 [https://choosealicense.com/](https://choosealicense.com/) 以帮助选择您想要的许可证。|`mod_license=MIT`|
|`mod_version`|您的 Mod 的版本，显示在 Mod 列表中。有关更多信息，请参阅 [版本控制](assets/3.版本控制-20250315093053-z0qccsa.md) 页面。|`mod_version=1.0`|
|`mod_group_id`|参见 [Group ID](#)。|`mod_group_id=com.example.examplemod`|
|`mod_authors`|Mod 的作者，显示在 Mod 列表中。|`mod_authors=ExampleModder`|
|`mod_description`|Mod 的描述，作为多行字符串，显示在 Mod 列表中。可以使用换行符 (`\n`)，并将被正确替换。|`mod_description=Example mod description.`|

### Mod ID

Mod ID 是区分您的 Mod 与其他 Mod 的主要方式。它在许多地方使用，包括作为您的 Mod 注册表的命名空间，以及作为资源和数据包的命名空间。如果两个 Mod 具有相同的 ID，游戏将无法加载。

因此，您的 Mod ID 应该是独特且易于记忆的。通常，它会是您的 Mod 的显示名称（但为小写），或其某种变体。Mod ID 只能包含小写字母、数字和下划线，并且长度必须在 2 到 64 个字符之间（包括 2 和 64）。

**注意**：在 `gradle.properties` 文件中更改此属性会自动在所有地方应用更改，除了您的主 Mod 类中的 `@Mod` 注解。在那里，您需要手动更改它以匹配 `gradle.properties` 文件中的值。

### Group ID

虽然 `gradle.properties` 文件中的 `group` 属性仅在您计划将 Mod 发布到 Maven 时才需要，但始终正确设置此属性被认为是良好的做法。这是通过 `gradle.properties` 文件中的 `mod_group_id` 属性为您完成的。

Group ID 应设置为您的顶级包。有关更多信息，请参阅 [2.结构化Mod](assets/2.结构化Mod-20250315093053-mqcn5ed.md)。

```properties
# 在您的 gradle.properties 文件中
mod_group_id=com.example
```

您的 Java 源代码 (`src/main/java`) 中的包也应符合此结构，内部包表示 Mod ID：

```
com
- example (在 group 属性中指定的顶级包)
    - mymod (Mod ID)
        - MyMod.java (重命名为 ExampleMod.java)
```

### neoforge.mods.toml

`neoforge.mods.toml` 文件位于 `src/main/resources/META-INF/neoforge.mods.toml`，是一个 TOML 格式的文件，定义了您的 Mod 的元数据。它还包含有关您的 Mod 应如何加载到游戏中的其他信息，以及在“Mods”菜单中显示的显示信息。MDK 提供的 `neoforge.mods.toml` 文件包含解释每个条目的注释，这里将更详细地解释它们。

`neoforge.mods.toml` 可以分为三个部分：非 Mod 特定的属性，这些属性与 Mod 文件相关联；Mod 属性，每个 Mod 都有一个部分；以及依赖配置，每个 Mod 或 Mod 的依赖项都有一个部分。与 `neoforge.mods.toml` 文件相关联的一些属性是强制性的；强制性属性需要指定一个值，否则将抛出异常。

**注意**：在默认的 MDK 中，Gradle 会使用 `gradle.properties` 文件中指定的值替换此文件中的各种属性。例如，`license="${mod_license}"` 行意味着 `license` 字段将被 `gradle.properties` 中的 `mod_license` 属性替换。像这样替换的值应在 `gradle.properties` 中更改，而不是在此处更改。

#### 非 Mod 特定的属性

非 Mod 特定的属性是与 JAR 本身相关联的属性，指示如何加载 Mod 以及任何其他全局元数据。

|属性|类型|默认值|描述|示例|
| ----| ------| ------| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------| ----------------------|
|`modLoader`|字符串|强制性|用于 Mod 的语言加载器。可用于支持替代语言结构，例如主文件的 Kotlin 对象，或确定入口点的不同方法，例如接口或方法。NeoForge 提供了 Java 加载器 `javafml` 和低代码/无代码加载器 `lowcodefml`。|`modLoader="javafml"`|
|`loaderVersion`|字符串|强制性|语言加载器的可接受版本范围，表示为 Maven 版本范围。对于 `javafml` 和 `lowcodefml`，当前版本为 `1`。|`loaderVersion="[1,)"`|
|`license`|字符串|强制性|此 JAR 中的 Mod 提供的许可证。建议将其设置为您使用的 SPDX 标识符和/或许可证的链接。您可以访问 [https://choosealicense.com/](https://choosealicense.com/) 以帮助选择您想要的许可证。|`license="MIT"`|
|`showAsResourcePack`|布尔值|`false`|当为 `true` 时，Mod 的资源将作为单独的资源包显示在“资源包”菜单中，而不是与“Mod 资源”包合并。|`showAsResourcePack=true`|
|`showAsDataPack`|布尔值|`false`|当为 `true` 时，Mod 的数据文件将作为单独的数据包显示在“数据包”菜单中，而不是与“Mod 数据”包合并。|`showAsDataPack=true`|
|`services`|数组|`[]`|您的 Mod 使用的服务数组。这是作为 NeoForge 的 Java 平台模块系统实现中创建的模块的一部分消耗的。|`services=["net.neoforged.neoforgespi.language.IModLanguageProvider"]`|
|`properties`|表|`{}`|替换属性的表。这由 `StringSubstitutor` 使用，以用其对应的值替换 `${file.<key>}`。|`properties={"example"="1.2.3"}`（然后可以通过 `${file.example}` 引用）|
|`issueTrackerURL`|字符串|无|表示报告和跟踪 Mod 问题的地方的 URL。|`"https://github.com/neoforged/NeoForge/issues"`|

**注意**：`services` 属性在功能上等同于在模块中指定 `uses` 指令，这允许加载给定类型的服务。

或者，它可以在 `src/main/resources/META-INF/services` 文件夹中的服务文件中定义，其中文件名是服务的完全限定名称，文件内容是服务的名称（另请参阅 AtlasViewer Mod 中的 [此示例](#)）。

#### Mod 特定的属性

Mod 特定的属性使用 `[[mods]]` 标头绑定到指定的 Mod。这是一个表数组；所有键/值属性将附加到该 Mod，直到下一个标头。

```toml
# examplemod1 的属性
[[mods]]
modId = "examplemod1"

# examplemod2 的属性
[[mods]]
modId = "examplemod2"
```

|属性|类型|默认值|描述|示例|
| ----| ------| ------| -------------------------------------------------------------------------------------------------------------------------| ----|
|`modId`|字符串|强制性|参见 [Mod ID](#)。|`modId="examplemod"`|
|`namespace`|字符串|`modId` 的值|Mod 的覆盖命名空间。必须也是有效的 Mod ID，但可以额外包括点或破折号。目前未使用。|`namespace="example"`|
|`version`|字符串|`"1"`|Mod 的版本，最好采用 Maven 版本控制的变体。当设置为 `${file.jarVersion}` 时，它将被替换为 JAR 清单中的 `Implementation-Version` 属性的值（在开发环境中显示为 `0.0NONE`）。|`version="1.20.2-1.0.0"`|
|`displayName`|字符串|`modId` 的值|Mod 的显示名称。用于在屏幕上表示 Mod（例如，Mod 列表，Mod 不匹配）。|`displayName="Example Mod"`|
|`description`|字符串|`'''MISSING DESCRIPTION'''`|在 Mod 列表屏幕上显示的 Mod 描述。建议使用多行字面字符串。此值也是可翻译的，有关更多信息，请参阅 [翻译 Mod 元数据](#)。|`description='''This is an example.'''`|
|`logoFile`|字符串|无|用于 Mod 列表屏幕上的图像文件的名称和扩展名。徽标必须位于 JAR 的根目录中或直接位于源集的根目录中（例如，对于主源集为 `src/main/resources`）。|`logoFile="example_logo.png"`|
|`logoBlur`|布尔值|`true`|是否使用 `GL_LINEAR`（true）或 `GL_NEAREST`（false）来渲染 `logoFile`。简单来说，这意味着在尝试缩放徽标时是否应模糊徽标。|`logoBlur=false`|
|`updateJSONURL`|字符串|无|更新检查器用于确保您正在玩的 Mod 是最新版本的 JSON 的 URL。|`updateJSONURL="https://example.github.io/update_checker.json"`|
|`features`|表|`{}`|参见 [features](#)。|`features={java_version="[17,)"}`|
|`modproperties`|表|`{}`|与此 Mod 关联的键/值表。NeoForge 未使用，但主要用于 Mod。|`modproperties={example="value"}`|
|`modUrl`|字符串|无|Mod 下载页面的 URL。目前未使用。|`modUrl="https://neoforged.net/"`|
|`credits`|字符串|无|在 Mod 列表屏幕上显示的 Mod 的致谢和鸣谢。|`credits="The person over here and there."`|
|`authors`|字符串|无|在 Mod 列表屏幕上显示的 Mod 的作者。|`authors="Example Person"`|
|`displayURL`|字符串|无|在 Mod 列表屏幕上显示的 Mod 的显示页面的 URL。|`displayURL="https://neoforged.net/"`|
|`enumExtensions`|字符串|无|用于枚举扩展的 JSON 文件的文件路径。|`enumExtensions="META_INF/enumextensions.json"`|

#### 功能

功能系统允许 Mod 在加载系统时要求某些设置、软件或硬件可用。当功能未满足时，Mod 加载将失败，并通知用户有关要求。目前，NeoForge 提供以下功能：

|功能|描述|示例|
| ----| --------------------------------------------------------------------------------------------------------------------------------------------| ----|
|`javaVersion`|Java 版本的可接受范围，表示为 Maven 版本范围。这应该是 Minecraft 使用的受支持版本。|`features={javaVersion="[17,)"}`|
|`openGLVersion`|OpenGL 版本的可接受范围，表示为 Maven 版本范围。Minecraft 需要 OpenGL 3.2 或更新版本。如果您想要求更新的 OpenGL 版本，可以在此处执行此操作。|`features={openGLVersion="[4.6,)"}`|

#### 访问转换器特定的属性

访问转换器特定的属性使用 `[[accessTransformers]]` 标头绑定到指定的访问转换器。这是一个表数组；所有键/值属性将附加到该访问转换器，直到下一个标头。访问转换器标头是可选的；但是，当指定时，所有元素都是强制性的。

|属性|类型|默认值|描述|示例|
| ----| ------| ------| -------| ----|
|`file`|字符串|强制性|参见 [添加 ATs](#)。|`file="at.cfg"`|

#### Mixin 配置属性

Mixin 配置属性使用 `[[mixins]]` 标头绑定到指定的 Mixin 配置。这是一个表数组；所有键/值属性将附加到该 Mixin 块，直到下一个标头。Mixin 标头是可选的；但是，当指定时，所有元素都是强制性的。

|属性|类型|默认值|描述|示例|
| ----| ------| ------| ----------------------| ----|
|`config`|字符串|强制性|Mixin 配置文件的位置。|`config="examplemod.mixins.json"`|

#### 依赖配置

Mod 可以指定它们的依赖项，这些依赖项由 NeoForge 在加载 Mod 之前检查。这些配置是使用 `[[dependencies.<modid>]]` 表数组创建的，其中 `modid` 是消耗依赖项的 Mod 的标识符。

|属性|类型|默认值|描述|示例|
| ----| ------| ------| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------| ----|
|`modId`|字符串|强制性|添加为依赖项的 Mod 的标识符。|`modId="jei"`|
|`type`|字符串|`"required"`|指定此依赖项的性质：`"required"` 是默认值，如果此依赖项缺失，则阻止 Mod 加载；`"optional"` 不会阻止 Mod 加载，但仍会验证依赖项是否兼容；`"incompatible"` 如果此依赖项存在，则阻止 Mod 加载；`"discouraged"` 仍允许 Mod 加载，但向用户显示警告。|`type="incompatible"`|
|`reason`|字符串|无|描述为什么需要此依赖项或为什么它不兼容的可选用户面向消息。|`reason="integration"`|
|`versionRange`|字符串|`""`|语言加载器的可接受版本范围，表示为 Maven 版本范围。空字符串匹配任何版本。|`versionRange="[1, 2)"`|
|`ordering`|字符串|`"NONE"`|定义 Mod 是否必须在此依赖项之前 (`"BEFORE"`) 或之后 (`"AFTER"`) 加载。如果顺序无关紧要，则返回 `"NONE"`。|`ordering="AFTER"`|
|`side`|字符串|`"BOTH"`|依赖项必须存在的物理端：`"CLIENT"`、`"SERVER"` 或 `"BOTH"`。|`side="CLIENT"`|
|`referralUrl`|字符串|无|依赖项下载页面的 URL。目前未使用。|`referralUrl="https://library.example.com/"`|

**警告**：两个 Mod 的 `ordering` 为 `"BEFORE"` 可能会导致崩溃，例如，如果 Mod A 必须加载 Mod B，同时 Mod B 必须加载 Mod A。

### Mod 入口点

现在 `neoforge.mods.toml` 已填写完毕，我们需要为 Mod 提供一个入口点。入口点本质上是执行 Mod 的起点。入口点本身由 `neoforge.mods.toml` 中使用的语言加载器确定。

#### javafml 和 @Mod

`javafml` 是 NeoForge 为 Java 编程语言提供的语言加载器。入口点使用带有 `@Mod` 注解的公共类定义。`@Mod` 的值必须包含 `neoforge.mods.toml` 中指定的一个 Mod ID。从那里，所有初始化逻辑（例如注册事件或添加 DeferredRegisters）可以在类的构造函数中指定。

主 Mod 类只能有一个公共构造函数；否则将抛出 `RuntimeException`。构造函数可以按任何顺序具有以下任何参数；它们都不是明确必需的。但是，不允许有重复的参数。

|参数类型|描述|
| --------| ------------------------------------------------|
|`IEventBus`|Mod 特定的事件总线（用于注册、事件等）|
|`ModContainer`|保存此 Mod 元数据的抽象容器|
|`FMLModContainer`|由 `javafml` 定义的实际容器，保存此 Mod 的元数据；`ModContainer` 的扩展|
|`Dist`|此 Mod 正在加载的物理端|

```java
@Mod("examplemod") // 必须与 neoforge.mods.toml 中的 mod id 匹配
public class ExampleMod {
    // 有效的构造函数，仅使用两个可用的参数类型
    public ExampleMod(IEventBus modBus, ModContainer container) {
        // 在此处初始化逻辑
    }
}
```

默认情况下，`@Mod` 注解在两端加载。这可以通过指定 `dist` 参数来更改：

```java
// 必须与 neoforge.mods.toml 中的 mod id 匹配
// 此 Mod 类将仅在物理客户端加载
@Mod(value = "examplemod", dist = Dist.CLIENT) 
public class ExampleModClient {
    // 有效的构造函数
    public ExampleModClient(FMLModContainer container, IEventBus modBus, Dist dist) {
        // 在此处初始化仅客户端的逻辑
    }
}
```

**注意**：`neoforge.mods.toml` 中的条目不需要相应的 `@Mod` 注解。同样，`neoforge.mods.toml` 中的条目可以有多个 `@Mod` 注解，例如，如果您想分离通用逻辑和仅客户端逻辑。

#### lowcodefml

`lowcodefml` 是一种语言加载器，用于将数据包和资源包作为 Mod 分发，而无需代码中的入口点。它被指定为 `lowcodefml` 而不是 `nocodefml`，以便将来可能需要进行最小的编码添加。
