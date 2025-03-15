# 命名二进制标签（Named Binary Tag, NBT）

### 命名二进制标签（Named Binary Tag, NBT）

NBT 是 Minecraft 最早引入的一种格式，由 Notch 亲自编写。它在 Minecraft 代码库中广泛用于数据存储。

---

### 规范

NBT 规范与 JSON 规范类似，但有一些区别：

* **字节、短整型、长整型和浮点数的类型**：分别用 `b`​、`s`​、`l`​ 和 `f`​ 后缀表示，类似于 Java 代码中的表示方式。
* **双精度浮点数**：也可以用 `d`​ 后缀表示，但这不是必需的，类似于 Java 代码。Java 中整数的可选 `i`​ 后缀在 NBT 中不允许。
* **后缀不区分大小写**：例如，`64b`​ 与 `64B`​ 相同，`0.5F`​ 与 `0.5f`​ 相同。
* **布尔值**：NBT 中没有布尔类型，它们用字节表示。`true`​ 变为 `1b`​，`false`​ 变为 `0b`​。

  * 当前实现将所有非零值视为 `true`​，因此 `2b`​ 也会被视为 `true`​。
* **空值**：NBT 中没有 `null`​ 的等价物。
* **键的引号**：键的引号是可选的。因此，JSON 属性 `"duration": 20`​ 在 NBT 中可以变为 `duration: 20`​ 或 `"duration": 20`​。
* **复合标签**：在 JSON 中称为子对象的内容在 NBT 中称为复合标签（或简称复合）。
* **列表**：NBT 列表不能混合类型，这与 JSON 不同。列表类型由第一个元素决定，或在代码中定义。

  * 但是，列表的列表可以混合不同类型的列表。因此，一个包含两个列表的列表，其中第一个是字符串列表，第二个是字节列表，是允许的。
* **数组类型**：NBT 中有特殊的数组类型，它们与列表不同，但遵循用方括号包含元素的方案。有三种数组类型：

  * **字节数组**：以 `[B;`​ 开头。例如：`[B;0b,30b]`​
  * **整数数组**：以 `[I;`​ 开头。例如：`[I;0,-300]`​
  * **长整型数组**：以 `[L;`​ 开头。例如：`[L;0l,240l]`​
* **尾随逗号**：在列表、数组和复合标签中允许尾随逗号。

---

### NBT 文件

Minecraft 广泛使用 `.nbt`​ 文件，例如数据包中的结构文件。包含区域内容（即一组区块）的区域文件（`.mca`​），以及游戏中不同地方使用的各种 `.dat`​ 文件，也是 NBT 文件。

NBT 文件通常使用 GZip 压缩。因此，它们是二进制文件，不能直接编辑。

---

### 代码中的 NBT

与 JSON 类似，所有 NBT 对象都是一个封装对象的子对象。因此，让我们创建一个：

```java
CompoundTag tag = new CompoundTag();
```

我们现在可以将数据放入该标签中：

```java
tag.putInt("Color", 0xffffff);
tag.putString("Level", "minecraft:overworld");
tag.putDouble("IAmRunningOutOfIdeasForNamesHere", 1d);
```

这里有几个辅助方法，例如 `putIntArray`​ 除了接受 `int[]`​ 的标准变体外，还有一个接受 `List<Integer>`​ 的便捷方法。

当然，我们也可以从该标签中获取值：

```java
int color = tag.getInt("Color");
String level = tag.getString("Level");
double d = tag.getDouble("IAmRunningOutOfIdeasForNamesHere");
```

如果不存在，数字类型将返回 `0`​。字符串将返回 `""`​。更复杂的类型（列表、数组、复合标签）如果不存在，则会抛出异常。

因此，我们需要通过检查标签元素是否存在来进行保护：

```java
boolean hasColor = tag.contains("Color");
boolean hasColorMoreExplicitly = tag.contains("Color", Tag.TAG_INT);
```

​`TAG_INT`​ 常量在 `Tag`​ 中定义，它是所有标签类型的超接口。除了 `CompoundTag`​ 之外，大多数标签类型主要是内部的，例如 `ByteTag`​ 或 `StringTag`​，尽管如果你偶然遇到一些情况，`CompoundTag#get`​ 和 `#put`​ 方法可以直接使用它们。

不过，有一个明显的例外：`ListTag`​。处理这些标签是特殊的，因为通过 `CompoundTag#getList`​ 获取列表标签时，还必须指定列表类型。因此，获取字符串列表的示例如下：

```java
ListTag list = tag.getList("SomeListHere", Tag.TAG_STRING);
```

同样，在创建 `ListTag`​ 时，也必须在创建时指定列表类型：

```java
ListTag list = new ListTag(List.of("Value1", "Value2"), Tag.TAG_STRING);
```

最后，直接在另一个 `CompoundTag`​ 中使用 `CompoundTag`​ 时，使用 `CompoundTag#get`​ 和 `#put`​：

```java
tag.put("Tag", new CompoundTag());
tag.get("Tag");
```

---

### NBT 的用途

NBT 在 Minecraft 中的许多地方使用。一些最常见的例子包括 `BlockEntity`​ 和 `Entity`​。

**注意**
`ItemStack`​ 将 NBT 的使用抽象为数据组件。

---

### 参见

* [Minecraft Wiki 上的 NBT 格式](https://minecraft.fandom.com/wiki/NBT_format)
