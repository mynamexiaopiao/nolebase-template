# 流编解码器（Stream Codecs）

### 流编解码器（Stream Codecs）

流编解码器是一种序列化工具，用于描述如何将对象存储到流中或从流中读取对象，例如缓冲区。流编解码器主要由原版的网络系统用于同步数据。

**注意**：由于流编解码器与编解码器（Codecs）类似，因此本文档的格式与编解码器文档相同，以展示它们的相似性。

---

#### 使用流编解码器

流编解码器通过 `StreamCodec#encode`​ 和 `StreamCodec#decode`​ 分别将对象编码到流中或从流中解码对象。`encode`​ 方法接受流和要编码的对象。`decode`​ 方法接受流并返回解码后的对象。通常，流是 `ByteBuf`​、`FriendlyByteBuf`​ 或 `RegistryFriendlyByteBuf`​ 之一。

```java
// 假设 exampleStreamCodec 表示一个 StreamCodec<ExampleJavaObject>
// 假设 exampleObject 是一个 ExampleJavaObject
// 假设 buffer 是一个 RegistryFriendlyByteBuf

// 将 Java 对象编码到缓冲区流中
exampleStreamCodec.encode(buffer, exampleObject);

// 从缓冲区流中读取 Java 对象
ExampleJavaObject obj = exampleStreamCodec.decode(buffer);
```

**注意**：除非你手动处理缓冲区对象，否则通常不会直接调用 `encode`​ 和 `decode`​。

---

#### 现有的流编解码器

##### ByteBufCodecs

​`ByteBufCodecs`​ 包含某些基本类型和对象的静态流编解码器实例。

|流编解码器|Java 类型|
| ------------| -----------|
|​`BOOL`​|​`Boolean`​|
|​`BYTE`​|​`Byte`​|
|​`SHORT`​|​`Short`​|
|​`INT`​|​`Integer`​|
|​`FLOAT`​|​`Float`​|
|​`DOUBLE`​|​`Double`​|
|​`BYTE_ARRAY`​|​`byte[]`​*|
|​`STRING_UTF8`​|​`String`​**|
|​`TAG`​|​`Tag`​|
|​`COMPOUND_TAG`​|​`CompoundTag`​|
|​`VECTOR3F`​|​`Vector3f`​|
|​`QUATERNIONF`​|​`Quaternionf`​|
|​`GAME_PROFILE`​|​`GameProfile`​|

* ​`byte[]`​ 可以通过 `ByteBufCodecs#byteArray`​ 限制为特定数量的值。
* ​`String`​ 可以通过 `ByteBufCodecs#stringUtf8`​ 限制为特定数量的字符。

此外，还有一些静态实例使用不同的方法编码和解码基本类型和对象。

---

##### 无符号短整型（Unsigned Shorts）

​`UNSIGNED_SHORT`​ 是 `SHORT`​ 的替代方案，旨在被视为无符号数。由于 Java 中的数字是有符号的，因此无符号短整型作为整数发送和接收，高两位字节被屏蔽。

---

##### 可变大小数字（Variable-Sized Number）

​`VAR_INT`​ 和 `VAR_LONG`​ 是流编解码器，其中值被编码为尽可能小。这是通过每次编码七位并使用高位作为标记来实现的，以指示是否有更多数据。对于整数，0 到 2<sup>28-1 之间的数字将发送比整数字节数更短或相等的字节数；对于长整型，0 到 2</sup>56-1 之间的数字将发送比长整型字节数更短或相等的字节数。如果你的数字通常在此范围内且通常位于较低端，则应使用这些可变流编解码器。

**注意**：`VAR_INT`​ 是 `INT`​ 的替代方案。

---

##### 可信标签（Trusted Tags）

​`TRUSTED_TAG`​ 和 `TRUSTED_COMPOUND_TAG`​ 分别是 `TAG`​ 和 `COMPOUND_TAG`​ 的变体，它们具有无限的堆来解码标签，而 `TAG`​ 和 `COMPOUND_TAG`​ 的限制为 2MiB。可信标签流编解码器应理想地仅用于客户端绑定的数据包，例如原版用于方块实体数据包和实体数据序列化器。

如果需要使用不同的限制，则可以使用 `NbtAccounter`​ 并提供给定的大小，使用 `ByteBufCodecs#tagCodec`​ 或 `#compoundTagCodec`​。

---

#### 创建流编解码器

流编解码器可以创建用于读取或写入任何对象到流中。本文档将重点介绍将流作为缓冲区的流编解码器，因为这是其主要用途。

流编解码器有两个泛型：`B`​ 表示缓冲区，`V`​ 表示对象值。`B`​ 通常是以下三种类型之一：`ByteBuf`​、`FriendlyByteBuf`​、`RegistryFriendlyByteBuf`​，每种类型都扩展了前一种类型。`FriendlyByteBuf`​ 添加了 Minecraft 特定的读写方法，而 `RegistryFriendlyByteBuf`​ 提供了对注册表及其对象的访问。

在构造流编解码器时，`B`​ 应是最不特定的缓冲区类型。例如，`ResourceLocation`​ 作为字符串发送。由于字符串由常规 `ByteBuf`​ 支持，因此其类型应为 `StreamCodec<ByteBuf, ResourceLocation>`​。`FriendlyByteBuf`​ 包含写入 `ChunkPos`​ 的方法，因此其类型应为 `StreamCodec<FriendlyByteBuf, ChunkPos>`​。`Item`​ 需要访问注册表，因此其类型应为 `StreamCodec<RegistryFriendlyByteBuf, Item>`​。

大多数接受流编解码器的方法都查找 `? super B`​ 作为缓冲区类型，这意味着如果缓冲区类型是 `RegistryFriendlyByteBuf`​，则上述所有示例都可以使用。

---

##### 成员编码器（Member Encoders）

​`StreamMemberEncoder`​ 是 `StreamEncoder`​ 的替代方案，其中编码对象首先出现，缓冲区其次。这通常用于编码对象包含将对象写入缓冲区的实例方法时。可以通过调用 `StreamCodec#ofMember`​ 使用 `StreamMemberEncoder`​ 创建流编解码器。

```java
// 要创建流编解码器的对象
public class ExampleObject {
    
    // 普通构造函数
    public ExampleObject(String arg1, int arg2, boolean arg3) { /* ... */ }

    // 流解码器引用
    public ExampleObject(ByteBuf buffer) { /* ... */ }

    // 流编码器引用
    public void encode(ByteBuf buffer) { /* ... */ }
}

// 流编解码器的样子
public static StreamCodec<ByteBuf, ExampleObject> =
    StreamCodec.ofMember(ExampleObject::encode, ExampleObject::new);
```

---

##### 组合（Composites）

流编解码器可以通过 `StreamCodec#composite`​ 读取和写入对象。每个组合流编解码器定义了一系列流编解码器和 getter，它们按提供的顺序读取/写入。`composite`​ 的重载最多支持六个参数。

每两个参数表示用于读取/写入字段的流编解码器和从对象中获取字段的 getter。最后一个参数是在解码时创建新对象的函数。

```java
// 要创建流编解码器的对象
public record SimpleExample(String arg1, int arg2, boolean arg3) {}
public record RegistryExample(double arg1, Holder<Item> arg2) {}

// 流编解码器
public static final StreamCodec<ByteBuf, SimpleExample> SIMPLE_STREAM_CODEC =
    StreamCodec.composite(
        // 流编解码器和 getter 对
        ByteBufCodecs.STRING_UTF8, SimpleExample::arg1,
        ByteBufCodecs.VAR_INT, SimpleExample::arg2,
        ByteBufCodecs.BOOL, SimpleExample::arg3,
        SimpleExample::new
    );

// 由于此对象包含 holder，因此使用 RegistryFriendlyByteBuf
public static final StreamCodec<RegistryFriendlyByteBuf, RegistryExample> REGISTRY_STREAM_CODEC =
    StreamCodec.composite(
        // 注意，ByteBuf 流编解码器可以在此处使用
        ByteBufCodecs.DOUBLE, RegistryExample::arg1,
        ByteBufCodecs.holderRegistry(Registries.ITEM), RegistryExample::arg2,
        RegistryExample::new
    );
```

---

##### 转换器（Transformers）

流编解码器可以使用映射方法转换为等效或部分等效的表示形式。两个映射方法应用于值，而一个映射方法应用于缓冲区。

​`map`​ 方法使用两个函数转换值：一个将当前类型转换为新类型，另一个将新类型转换回当前类型。这与编解码器转换器类似。

```java
public static final StreamCodec<ByteBuf, ResourceLocation> STREAM_CODEC = 
    ByteBufCodecs.STRING_UTF8.map(
        // String -> ResourceLocation
        ResourceLocation::new,
        // ResourceLocation -> String
        ResourceLocation::toString
    );
```

​`apply`​ 方法使用 `StreamCodec.CodecOperation`​ 转换值。`StreamCodec.CodecOperation`​ 接受当前类型的流编解码器并返回新类型的流编解码器。这些通常包装 `map`​ 或接受辅助方法。

```java
public static final StreamCodec<ByteBuf, List<ResourceLocation>> STREAM_CODEC =
    ResourceLocation.STREAM_CODEC.apply(ByteBufCodecs.list());
```

​`mapStream`​ 方法使用一个函数转换缓冲区，该函数接受新缓冲区类型并返回当前缓冲区类型。此方法应很少使用，因为大多数带有流编解码器的方法不需要更改缓冲区的类型。

```java
public static final StreamCodec<RegistryFriendlyByteBuf, Integer> STREAM_CODEC =
    ByteBufCodecs.VAR_INT.mapStream(buffer -> (ByteBuf) buffer);
```

---

##### 单位（Unit）

一个流编解码器可以提供代码中的值并编码为空，可以使用 `StreamCodec#unit`​ 表示。如果不应通过网络同步任何信息，则这很有用。

**警告**：单位流编解码器期望任何编码对象必须与指定的单位匹配；否则将抛出错误。因此，所有对象必须具有某些 `equals`​ 实现，该实现为单位对象返回 `true`​，或者提供给流编解码器的实例在编码时始终提供。

```java
public static final StreamCodec<ByteBuf, Item> UNIT_STREAM_CODEC =
    StreamCodec.unit(Items.AIR);
```

---

##### 延迟初始化（Lazy Initialized）

有时，流编解码器可能依赖于构造时不存在的数据。在这种情况下，可以使用 `NeoForgeStreamCodecs#lazy`​ 让流编解码器在首次读取/写入时构造自身。该方法接受提供的流编解码器。

```java
public static final StreamCodec<ByteBuf, Item> LAZY_STREAM_CODEC = 
    NeoForgeStreamCodecs.lazy(
        () -> StreamCodec.unit(Items.AIR)
    );
```

---

##### 集合（Collections）

可以从对象流编解码器生成集合的流编解码器，使用 `collection`​。`collection`​ 接受一个 `IntFunction`​，用于构造空集合、对象的流编解码器以及可选的最大大小。

```java
public static final StreamCodec<ByteBuf, Set<BlockPos>> COLLECTION_STREAM_CODEC =
    ByteBufCodecs.collection(
        HashSet::new, // 构造具有指定容量的集合
        BlockPos.STREAM_CODEC,
        256 // 集合最多只能有 256 个元素
    );
```

​`collection`​ 的另一个重载可以通过 `StreamCodec#apply`​ 指定。

```java
public static final StreamCodec<ByteBuf, Set<BlockPos>> COLLECTION_STREAM_CODEC =
    BlockPos.STREAM_CODEC.apply(
        ByteBufCodecs.collection(HashSet::new)
    );
```

基于列表的集合也可以通过 `StreamCodec#apply`​ 指定，调用 `ByteBufCodecs#list`​ 并可选地指定最大大小。

```java
public static final StreamCodec<ByteBuf, List<BlockPos>> LIST_STREAM_CODEC =
    BlockPos.STREAM_CODEC.apply(
        // 列表最多只能有 256 个元素
        ByteBufCodecs.list(256)
    );
```

---

##### 映射（Map）

可以使用两个流编解码器通过 `ByteBufCodecs#map`​ 生成键和值对象的映射的流编解码器。该函数还接受一个 `IntFunction`​，用于构造空映射，以及可选的最大大小。

```java
public static final StreamCodec<ByteBuf, Map<String, BlockPos>> MAP_STREAM_CODEC =
    ByteBufCodecs.map(
        HashMap::new, // 构造具有指定容量的映射
        ByteBufCodecs.STRING_UTF8,
        BlockPos.STREAM_CODEC,
        256 // 映射最多只能有 256 个元素
    );
```

---

##### 任一（Either）

可以使用两个流编解码器通过 `ByteBufCodecs#either`​ 生成两种不同方法读取/写入某些对象数据的流编解码器。此方法首先读取/写入一个布尔值，指示是读取/写入第一个还是第二个流编解码器。

```java
public static final StreamCodec<ByteBuf, Either<Integer, String>> EITHER_STREAM_CODEC = 
    ByteBufCodecs.either(
        ByteBufCodecs.VAR_INT,
        ByteBufCodecs.STRING_UTF8
    );
```

---

##### ID 映射器（Id Mapper）

在大多数情况下，当通过网络发送信息时，如果对象存在于双方，则发送表示 ID 的整数。表示对象的 ID 减少了需要通过网络同步的信息量。枚举和注册表都使用此方法。

​`ByteBufCodecs#idMapper`​ 提供了一种方便的方式来发送对象的 ID。它接受两个函数，将对象转换为 `int`​，反之亦然，或者接受一个 `IdMap`​。

```java
// 对于某个枚举
public enum ExampleIdObject {
    ;

    // 获取 ID -> 枚举
    public static final IntFunction<ExampleIdObject> BY_ID = 
        ByIdMap.continuous(
            ExampleIdObject::getId,
            ExampleIdObject.values(),
            ByIdMap.OutOfBoundsStrategy.ZERO
    );
    
    ExampleIdObject(int id) { /* ... */ }
}

// 流编解码器的样子
public static final StreamCodec<ByteBuf, ExampleIdObject> ID_STREAM_CODEC =
    ByteBufCodecs.idMapper(ExampleIdObject.BY_ID, ExampleIdObject::getId);
```

**注意**：NeoForge 提供了一个替代方案，用于不缓存枚举值的 ID 映射器，通过 `IExtensibleEnum#createStreamCodecForExtensibleEnum`​。然而，这很少需要在可扩展枚举之外使用。

---

##### 可选（Optional）

可以通过提供流编解码器给 `ByteBufCodecs#optional`​ 生成发送 `Optional`​ 包装值的流编解码器。此方法首先读取/写入一个布尔值，指示是否读取/写入对象。

```java
public static final StreamCodec<RegistryFriendlyByteBuf, Optional<DataComponentType<?>>> OPTIONAL_STREAM_CODEC =
    DataComponentType.STREAM_CODEC.apply(ByteBufCodecs::optional);
```

---

##### 注册表对象（Registry Objects）

注册表对象可以通过以下三种方法之一发送到网络：`registry`​、`holderRegistry`​ 或 `holder`​。每种方法都接受一个 `ResourceKey`​，表示注册表对象所在的注册表。

**警告**：自定义注册表必须通过调用 `RegistryBuilder#sync`​ 并将值设置为 `true`​ 来同步。否则，编码器将抛出异常。

​`registry`​ 和 `holderRegistry`​ 分别返回注册表对象或持有者包装的注册表对象。这些方法发送表示注册表对象的 ID。

```java
// 注册表对象
public static final StreamCodec<RegistryFriendlyByteBuf, Item> VALUE_STREAM_CODEC =
    BytebufCodecs.registry(Registries.ITEM);

// 持有者包装的注册表对象
public static final StreamCodec<RegistryFriendlyByteBuf, Holder<Item>> HOLDER_STREAM_CODEC =
    BytebufCodecs.holderRegistry(Registries.ITEM);
```

​`holder`​ 返回持有者包装的注册表对象。此方法发送表示注册表对象的 ID，或者如果提供的持有者是直接引用，则发送注册表对象本身。为此，`holder`​ 还接受注册表对象的流编解码器。

```java
public static final StreamCodec<RegistryFriendlyByteBuf, Holder<SoundEvent>> STREAM_CODEC =
    ByteBufCodecs.holder(
        Registries.SOUND_EVENT, SoundEvent.DIRECT_STREAM_CODEC
    );
```

**注意**：如果持有者不是直接的，则 `holder`​ 只会为非同步的自定义注册表抛出异常。

---

##### 持有者集合（Holder Sets）

可以使用 `holderSet`​ 发送标签或持有者包装的注册表对象的集合。它接受一个 `ResourceKey`​，表示注册表对象所在的注册表。

```java
public static final StreamCodec<RegistryFriendlyByteBuf, HolderSet<Item>> HOLDER_SET_STREAM_CODEC =
    BytebufCodecs.holderSet(Registries.ITEM);
```

---

##### 递归（Recursive）

有时，对象可能引用相同类型的对象作为字段。例如，`MobEffectInstance`​ 如果存在隐藏效果，则接受一个可选的 `MobEffectInstance`​。在这种情况下，可以使用 `StreamCodec#recursive`​ 将流编解码器作为创建流编解码器的函数的一部分提供。

```java
// 定义我们的递归对象
public record RecursiveObject(Optional<RecursiveObject> inner) { /* ... */ }

public static final StreamCodec<ByteBuf, RecursiveObject> RECURSIVE_CODEC = StreamCodec.recursive(
    recursedStreamCodec -> StreamCodec.composite(
        recursedStreamCodec.apply(ByteBufCodecs::optional),
        RecursiveObject::inner,
        RecursiveObject::new
    )
);
```

---

##### 分发（Dispatch）

流编解码器可以具有子流编解码器，这些子流编解码器可以根据某些指定类型解码特定对象，通过 `StreamCodec#dispatch`​。这通常与表示类型的注册表对象一起使用，例如 `ParticleType`​ 用于 `ParticleOptions`​ 或 `StatType`​ 用于 `Stats`​。

分发流编解码器首先尝试读取/写入类型对象。然后，使用方法中提供的函数之一读取/写入当前对象。第一个函数接受当前对象并获取要写入值的类型。第二个函数接受类型对象并获取当前对象的流编解码器以读取值。

```java
// 定义我们的对象
public abstract class ExampleObject {

    // 定义用于指定对象类型以进行编码的方法
    public abstract StreamCodec<? super RegistryFriendlyByteBuf, ? extends ExampleObject> streamCodec();
}

// 假设有一个 ResourceKey<StreamCodec< super RegistryFriendlyByteBuf, ? extends ExampleObject>> DISPATCH
public static final StreamCodec<RegistryFriendlyByteBuf, ExampleObject> DISPATCH_STREAM_CODEC =
    ByteBufCodecs.registry(DISPATCH).dispatch(
        // 从特定对象获取流编解码器
        ExampleObject::streamCodec,
        // 从注册表对象获取流编解码器
        Function.identity()
    );
```
