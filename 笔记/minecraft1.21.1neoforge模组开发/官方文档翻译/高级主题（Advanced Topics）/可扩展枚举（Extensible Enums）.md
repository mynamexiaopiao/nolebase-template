# 可扩展枚举（Extensible Enums）

### 可扩展枚举（Extensible Enums）

可扩展枚举是对特定原版枚举的增强，允许添加新的枚举项。这是通过在运行时修改枚举的编译字节码来实现的。

#### IExtensibleEnum 接口

所有可以添加新项的枚举都实现了 `IExtensibleEnum`​ 接口。该接口作为一个标记，允许 `RuntimeEnumExtender`​ 启动插件服务知道哪些枚举应该被转换。

**警告**：
你不应该在自己的枚举上实现此接口。根据你的使用场景，使用映射（maps）或注册表（registries）来代替。
由于转换器的运行顺序，未修补以实现此接口的枚举无法通过 Mixin 或 Coremod 添加该接口。

---

### 创建枚举项

要创建新的枚举项，需要创建一个 JSON 文件，并在 `neoforge.mods.toml`​ 中通过 `[[mods]]`​ 块的 `enumExtensions`​ 条目引用它。指定的路径必须相对于资源目录：

```toml
# 在 neoforge.mods.toml 中：
[[mods]]
## 文件相对于资源的输出目录，或编译后 jar 内的根路径
## 'resources' 目录表示资源的根输出目录
enumExtensions = "META-INF/enumextensions.json"
```

枚举项的定义包括目标枚举的类名、新字段的名称（必须以模组 ID 为前缀）、用于构造项的构造函数的描述符以及传递给该构造函数的参数。

```json
{
    "entries": [
        {
            // 要添加项的枚举类
            "enum": "net/minecraft/world/item/ItemDisplayContext",
            // 新字段的名称，必须以模组 ID 为前缀
            "name": "EXAMPLEMOD_STANDING",
            // 要使用的构造函数
            "constructor": "(ILjava/lang/String;Ljava/lang/String;)V",
            // 直接提供的常量参数
            "parameters": [ -1, "examplemod:standing", null ]
        },
        {
            "enum": "net/minecraft/world/item/Rarity",
            "name": "EXAMPLEMOD_CUSTOM",
            "constructor": "(ILjava/lang/String;Ljava/util/function/UnaryOperator;)V",
            // 要使用的参数，作为对给定类中 EnumProxy<Rarity> 字段的引用
            "parameters": {
                "class": "example/examplemod/MyEnumParams",
                "field": "CUSTOM_RARITY_ENUM_PROXY"
            }
        },
        {
            "enum": "net/minecraft/world/damagesource/DamageEffects",
            "name": "EXAMPLEMOD_TEST",
            "constructor": "(Ljava/lang/String;Ljava/util/function/Supplier;)V",
            // 要使用的参数，作为对给定类中方法的引用
            "parameters": {
                "class": "example/examplemod/MyEnumParams",
                "method": "getTestDamageEffectsParameter"
            }
        }
    ]
}
```

```java
public class MyEnumParams {
    public static final EnumProxy<Rarity> CUSTOM_RARITY_ENUM_PROXY = new EnumProxy<>(
            Rarity.class, -1, "examplemod:custom", (UnaryOperator<Style>) style -> style.withItalic(true)
    );
    
    public static Object getTestDamageEffectsParameter(int idx, Class<?> type) {
        return type.cast(switch (idx) {
            case 0 -> "examplemod:test";
            case 1 -> (Supplier<SoundEvent>) () -> SoundEvents.DONKEY_ANGRY;
            default -> throw new IllegalArgumentException("Unexpected parameter index: " + idx);
        });
    }
}
```

---

### 构造函数

构造函数必须指定为方法描述符，并且只能包含源代码中可见的参数，忽略隐藏的常量名称和序号参数。
如果构造函数标记了 `@ReservedConstructor`​ 注解，则不能用于模组枚举常量。

---

### 参数

参数可以通过三种方式指定，具体取决于参数类型：

1. **在 JSON 文件中内联**：作为常量数组（仅允许原始值、字符串和将 `null`​ 传递给任何引用类型）。
2. **作为对类中** **​`EnumProxy<TheEnum>`​** ​ **类型字段的引用**：第一个参数指定目标枚举，后续参数是传递给枚举构造函数的参数。
3. **作为对返回** **​`Object`​**​ **的方法的引用**：返回值是要使用的参数值。该方法必须有两个参数：`int`​（参数索引）和 `Class<?>`​（参数的预期类型）。
    `Class<?>`​ 对象应用于强制转换（`Class#cast()`​）返回值，以保持模组代码中的 `ClassCastException`​。

**警告**：
用作参数值来源的字段和/或方法应放在单独的类中，以避免过早加载模组类。

某些参数有额外的规则：

* 如果参数是与枚举上的 `@IndexedEnum`​ 注解相关的 `int`​ ID 参数，则忽略它并替换为枚举项的序号。如果在 JSON 中内联指定该参数，则必须指定为 `-1`​，否则会抛出异常。
* 如果参数是与枚举上的 `@NamedEnum`​ 注解相关的 `String`​ 名称参数，则必须以模组 ID 为前缀，格式为 `namespace:path`​（类似于 `ResourceLocation`​），否则会抛出异常。

---

### 检索生成的常量

生成的枚举常量可以通过 `TheEnum.valueOf(String)`​ 检索。如果使用字段引用提供参数，则也可以通过 `EnumProxy`​ 对象的 `EnumProxy#getValue()`​ 检索常量。

---

### 为 NeoForge 贡献

要向 NeoForge 添加新的可扩展枚举，至少需要做两件事：

1. 让枚举实现 `IExtensibleEnum`​，以标记该枚举应通过 `RuntimeEnumExtender`​ 进行转换。
2. 添加一个 `getExtensionInfo`​ 方法，返回 `ExtensionInfo.nonExtended(TheEnum.class)`​。

根据枚举的具体细节，可能需要进一步的操作：

* 如果枚举有一个应与枚举项序号匹配的 `int`​ ID 参数，则枚举应使用 `@NumberedEnum`​ 注解，并将 ID 的参数索引作为注解的值（如果不是第一个参数）。
* 如果枚举有一个用于序列化的 `String`​ 名称参数，则应使用 `@NamedEnum`​ 注解，并将名称的参数索引作为注解的值（如果不是第一个参数）。
* 如果枚举通过网络发送，则应使用 `@NetworkedEnum`​ 注解，注解的参数指定值可以发送的方向（客户端、服务器或双向）。
  **警告**：一旦在 NeoForge 中实现了枚举的网络检查，网络枚举将需要额外的步骤。
* 如果枚举的构造函数不能被模组使用（例如，因为它们需要枚举上的注册表对象，而枚举可能在模组注册运行之前初始化），则应使用 `@ReservedConstructor`​ 注解。

**注意**：
`getExtensionInfo`​ 方法将在运行时被转换，以提供动态生成的 `ExtensionInfo`​（如果枚举实际添加了任何项）。

```java
// 这是一个示例，不是原版中的实际枚举
public enum ExampleEnum implements net.neoforged.fml.common.asm.enumextension.IExtensibleEnum {
    // VALUE_1 在这里表示名称参数
    VALUE_1(0, "value_1", false),
    VALUE_2(1, "value_2", true),
    VALUE_3(2, "value_3");

    ExampleEnum(int arg1, String arg2, boolean arg3) {
        // ...
    }

    ExampleEnum(int arg1, String arg2) {
        this(arg1, arg2, false);
    }

    public static net.neoforged.fml.common.asm.enumextension.ExtensionInfo getExtensionInfo() {
        return net.neoforged.fml.common.asm.enumextension.ExtensionInfo.nonExtended(ExampleEnum.class);
    }
}
```

‍
