# 配置（Configuration）

### 配置（Configuration）

配置用于定义模组的设置和用户偏好，可以在模组实例中应用。NeoForge 使用基于 TOML 文件的配置系统，并通过 NightConfig 进行读取。

---

#### 创建配置

配置可以通过 `IConfigSpec`​ 的子类型创建。NeoForge 通过 `ModConfigSpec`​ 实现该类型，并通过 `ModConfigSpec.Builder`​ 进行构建。构建器可以通过 `Builder#push`​ 创建配置部分，并通过 `Builder#pop`​ 离开部分。之后，可以使用以下两种方法之一构建配置：

|方法|描述|
| ------| ------------------------------------|
|​`build`​|创建 `ModConfigSpec`​。|
|​`configure`​|创建一个包含配置值类和 `ModConfigSpec`​ 的配对。|

**注意**：
`ModConfigSpec.Builder#configure`​ 通常与静态块和以 `ModConfigSpec.Builder`​ 为构造函数参数的类一起使用，以附加和保存配置值。

```java
// 定义字段以保存配置和规范
public static final ExampleConfig CONFIG;
public static final ModConfigSpec CONFIG_SPEC;

// CONFIG 和 CONFIG_SPEC 从同一个构建器构建，因此使用静态块分离属性
static {
    Pair<ExampleConfig, ModConfigSpec> pair =
            new ModConfigSpec.Builder().configure(ExampleConfig::new);

    // 存储结果值
    CONFIG = pair.getLeft();
    CONFIG_SPEC = pair.getRight();
}
```

---

#### 配置值（ConfigValue）

每个配置值都可以附加额外的上下文以提供额外行为。上下文必须在配置值完全构建之前定义：

|方法|描述|
| ------| ------------------------------------------------------|
|​`comment`​|提供配置值的描述。可以提供多个字符串以支持多行注释。|
|​`translation`​|提供配置值名称的翻译键。|
|​`worldRestart`​|必须重启世界才能更改配置值。|

配置值可以通过提供的上下文（如果已定义）使用任何 `#define`​ 方法构建。所有配置值方法至少需要两个组件：

1. 表示变量名称的路径：用 `.`​ 分隔的字符串，表示配置值所在的部分。
2. 当没有有效配置时的默认值。

​`ConfigValue`​ 特定方法还需要两个额外组件：

1. 验证器，用于确保反序列化的对象有效。
2. 表示配置值数据类型的类。

```java
// 将配置属性存储为公共 final 字段
public final ModConfigSpec.ConfigValue<String> welcomeMessage;

private ExampleConfig(ModConfigSpec.Builder builder) {
    // 定义每个属性
    // 一个属性可以是游戏初始化时记录到控制台的消息
    welcomeMessage = builder.define("welcome_message", "Hello from the config!");
}
```

配置值可以通过 `ConfigValue#get`​ 获取。这些值会被缓存，以防止多次读取文件。

---

#### 其他配置值类型

1. **范围值（Range Values）**

    * 描述：值必须在定义的范围内。
    * 类类型：`Comparable<T>`​
    * 方法名：`#defineInRange`​
    * 额外组件：最小值、最大值和表示数据类型的类。

    **注意**：
    `DoubleValues`​、`IntValues`​ 和 `LongValues`​ 是范围值，分别指定类为 `Double`​、`Integer`​ 和 `Long`​。
2. **白名单值（Whitelisted Values）**

    * 描述：值必须在提供的集合中。
    * 类类型：`T`​
    * 方法名：`#defineInList`​
    * 额外组件：允许的配置值集合。
3. **列表值（List Values）**

    * 描述：值是条目的列表。
    * 类类型：`List<T>`​
    * 方法名：`#defineList`​、`#defineListAllowEmpty`​（如果列表可以为空）。
    * 额外组件：返回默认值的供应商、验证器（可选）和列表条目数量的验证器。
4. **枚举值（Enum Values）**

    * 描述：集合中的枚举值。
    * 类类型：`Enum<T>`​
    * 方法名：`#defineEnum`​
    * 额外组件：将字符串或整数转换为枚举的获取器，以及允许的配置值集合。
5. **布尔值（Boolean Values）**

    * 描述：布尔值。
    * 类类型：`Boolean`​
    * 方法名：`#define`​

---

#### 注册配置

一旦构建了 `ModConfigSpec`​，必须注册它以允许 NeoForge 根据需要加载、跟踪和同步配置设置。配置应在模组构造函数中通过 `ModContainer#registerConfig`​ 注册。配置可以注册为特定类型，表示配置所属的端，并可选地指定配置文件的名称。

```java
// 在主模组文件中使用 ModConfigSpec CONFIG
public ExampleMod(ModContainer container) {
    ...
    // 注册配置
    container.registerConfig(ModConfig.Type.COMMON, ExampleConfig.CONFIG);
    ...
}
```

---

#### 配置类型

配置类型决定了配置文件的位置、加载时间以及是否通过网络同步。所有配置默认从物理客户端的 `.minecraft/config`​ 或物理服务器的 `<server_folder>/config`​ 加载。以下是每种配置类型的详细信息：

1. **STARTUP**

    * 从配置文件夹加载到物理客户端和物理服务器。
    * 注册时立即读取。
    * 不通过网络同步。
    * 默认后缀为 `-startup`​。

    **警告**：
    `STARTUP`​ 类型的配置可能导致客户端和服务器之间的不同步，例如用于禁用内容注册的配置。因此，强烈建议不要在 `STARTUP`​ 中使用启用或禁用功能的配置。
2. **CLIENT**

    * 仅从配置文件夹加载到物理客户端。
    * 服务器没有此配置类型的位置。
    * 在 `FMLCommonSetupEvent`​ 触发之前立即读取。
    * 不通过网络同步。
    * 默认后缀为 `-client`​。
3. **COMMON**

    * 从配置文件夹加载到物理客户端和物理服务器。
    * 在 `FMLCommonSetupEvent`​ 触发之前立即读取。
    * 不通过网络同步。
    * 默认后缀为 `-common`​。
4. **SERVER**

    * 从配置文件夹加载到物理客户端和物理服务器。
    * 可以通过以下路径为每个世界覆盖配置：

      * 客户端：`.minecraft/saves/<world_name>/serverconfig`​
      * 服务器：`<server_folder>/world/serverconfig`​
    * 在 `ServerAboutToStartEvent`​ 触发之前立即读取。
    * 通过网络同步到客户端。
    * 默认后缀为 `-server`​。

---

#### 配置事件

配置加载或重新加载时执行的操作可以通过 `ModConfigEvent.Loading`​ 和 `ModConfigEvent.Reloading`​ 事件完成。这些事件必须注册到模组事件总线。

**注意**：
这些事件为模组的所有配置调用；提供的 `ModConfig`​ 对象应用于标识正在加载或重新加载的配置。

---

#### 配置屏幕

配置屏幕允许用户在游戏内编辑模组的配置值，而无需打开任何文件。屏幕会自动解析注册的配置文件并填充屏幕。

模组可以使用 NeoForge 提供的内置配置屏幕。模组可以扩展 `ConfigurationScreen`​ 以更改默认屏幕的行为，或创建自己的配置屏幕。模组还可以从头开始创建自己的屏幕，并通过以下扩展点将其提供给 NeoForge。

配置屏幕可以在客户端模组构造函数中通过注册 `IConfigScreenFactory`​ 扩展点来注册：

```java
// 在主客户端模组文件中
public ExampleModClient(ModContainer container) {
    ...
    // 这将使用 NeoForge 的 ConfigurationScreen 显示此模组的配置
    container.registerExtensionPoint(IConfigScreenFactory.class, ConfigurationScreen::new);
    ...
}
```

配置屏幕可以在游戏中通过进入“模组”页面，从侧边栏选择模组，然后点击“配置”按钮访问。`Startup`​、`Common`​ 和 `Client`​ 配置选项始终可编辑。`Server`​ 配置仅在本地玩世界时可在屏幕中编辑。如果连接到服务器或其他人的局域网世界，`Server`​ 配置选项将在屏幕中禁用。模组配置屏幕的第一页将显示每个注册的配置文件，供玩家选择要编辑的文件。

**警告**：
如果创建屏幕，应为所有配置条目添加翻译键并在语言 JSON 中定义文本。

可以通过 `ModConfigSpec$Builder#translation`​ 方法指定配置的翻译键，因此我们可以扩展之前的代码：

```java
ConfigValue<T> value = builder.comment("This value is called 'config_value_name', and is set to defaultValue if no existing config is present")
    .translation("modid.config.config_value_name")
    .define("config_value_name", defaultValue);
```

为了简化翻译，打开配置屏幕并访问所有配置及其子部分，然后返回模组列表屏幕。此时，所有未翻译的配置条目将打印到控制台。这样可以更容易地知道需要翻译的内容以及翻译键是什么。
