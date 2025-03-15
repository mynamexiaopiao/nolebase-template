# 访问转换器（Access Transformers）

### 访问转换器（Access Transformers）

访问转换器（Access Transformers，简称 AT）允许扩大类的可见性并修改类、方法和字段的 `final`​ 标志。它们允许模组开发者访问和修改原本无法访问的类成员。

访问转换器的规范文档可以在 [NeoForged GitHub](https://github.com/NeoForged/NeoForged) 上查看。

---

#### 添加访问转换器

将访问转换器添加到你的模组项目中非常简单，只需在 `build.gradle`​ 中添加一行代码：

访问转换器需要在 `build.gradle`​ 中声明。AT 文件可以放在任何位置，只要它们在编译时被复制到资源输出目录中。

```gradle
// 在 build.gradle 中：
// 这个块也是你指定映射版本的地方
minecraft {
    accessTransformers {
        file('src/main/resources/META-INF/accesstransformer.cfg')
    }
}
```

默认情况下，NeoForge 会搜索 `META-INF/accesstransformer.cfg`​。如果 `build.gradle`​ 指定了其他位置的访问转换器，则需要在 `neoforge.mods.toml`​ 中定义它们的位置：

```toml
# 在 neoforge.mods.toml 中：
[[accessTransformers]]
## 文件相对于资源的输出目录，或编译后 jar 中的根路径
## 'resources' 目录表示资源的根输出目录
file="META-INF/accesstransformer.cfg"
```

此外，可以指定多个 AT 文件，并按顺序应用。这对于具有多个包的大型模组非常有用。

```gradle
// 在 build.gradle 中：
minecraft {
    accessTransformers {
        file('src/main/resources/accesstransformer_main.cfg')
        file('src/additions/resources/accesstransformer_additions.cfg')
    }
}
```

```toml
# 在 neoforge.mods.toml 中
[[accessTransformers]]
file="accesstransformer_main.cfg"

[[accessTransformers]]
file="accesstransformer_additions.cfg"
```

添加或修改任何访问转换器后，必须刷新 Gradle 项目以使转换生效。

---

#### 访问转换器规范

##### 注释

​`#`​ 之后的所有文本直到行尾都将被视为注释，不会被解析。

---

##### 访问修饰符

访问修饰符指定将给定目标的成员可见性转换为什么。按可见性从高到低排序：

* ​`public`​ - 对包内外的所有类可见
* ​`protected`​ - 仅对包内的类和子类可见
* ​`default`​ - 仅对包内的类可见
* ​`private`​ - 仅对类内部可见

可以在上述修饰符后附加特殊修饰符 `+f`​ 和 `-f`​，以分别添加或删除 `final`​ 修饰符，`final`​ 修饰符在应用时会阻止子类化、方法重写或字段修改。

**警告**：指令仅修改它们直接引用的方法；任何重写方法都不会被访问转换。建议确保转换后的方法没有限制可见性的未转换重写方法，否则会导致 JVM 抛出错误。

可以安全转换的方法示例包括私有方法、最终方法（或最终类中的方法）和静态方法。

---

##### 目标和指令

###### 类

要针对类：

```plaintext
<访问修饰符> <完全限定类名>
```

内部类通过将外部类的完全限定名称和内部类的名称与 `$`​ 作为分隔符组合来表示。

---

###### 字段

要针对字段：

```plaintext
<访问修饰符> <完全限定类名> <字段名>
```

---

###### 方法

针对方法需要一种特殊的语法来表示方法参数和返回类型：

```plaintext
<访问修饰符> <完全限定类名> <方法名>(<参数类型>)<返回类型>
```

---

##### 指定类型

也称为“描述符”：有关更多技术细节，请参阅 [Java 虚拟机规范，SE 21，第 4.3.2 和 4.3.3 节](https://docs.oracle.com/javase/specs/jvms/se21/html/jvms-4.html#jvms-4.3.2)。

* ​`B`​ - `byte`​，有符号字节
* ​`C`​ - `char`​，UTF-16 中的 Unicode 字符代码点
* ​`D`​ - `double`​，双精度浮点值
* ​`F`​ - `float`​，单精度浮点值
* ​`I`​ - `int`​，32 位整数
* ​`J`​ - `long`​，64 位整数
* ​`S`​ - `short`​，有符号短整型
* ​`Z`​ - `boolean`​，`true`​ 或 `false`​ 值
* ​`[`​ - 引用数组的一个维度
  示例：`[[S`​ 表示 `short[][]`​
* ​`L<class name>;`​ - 引用引用类型
  示例：`Ljava/lang/String;`​ 表示 `java.lang.String`​ 引用类型（注意使用斜杠而不是句点）
* ​`(`​ - 引用方法描述符，参数应在此处提供，如果没有参数则为空
  示例：`<method>(I)Z`​ 表示需要一个整数参数并返回布尔值的方法
* ​`V`​ - 表示方法不返回值，只能用于方法描述符的末尾
  示例：`<method>()V`​ 表示没有参数且不返回任何内容的方法

---

##### 示例

```plaintext
# 使 Crypt 中的 ByteArrayToKeyFunction 接口变为 public
public net.minecraft.util.Crypt$ByteArrayToKeyFunction

# 使 MinecraftServer 中的 'random' 字段变为 protected 并移除 final 修饰符
protected-f net.minecraft.server.MinecraftServer random

# 使 Util 中的 'makeExecutor' 方法变为 public，
# 接受一个 String 并返回一个 ExecutorService
public net.minecraft.Util makeExecutor(Ljava/lang/String;)Ljava/util/concurrent/ExecutorService;

# 使 UUIDUtil 中的 'leastMostToIntArray' 方法变为 public，
# 接受两个 long 并返回一个 int[]
public net.minecraft.core.UUIDUtil leastMostToIntArray(JJ)[I
```

---

希望这些内容对你的开发有所帮助！如果有其他问题，欢迎随时提问。
