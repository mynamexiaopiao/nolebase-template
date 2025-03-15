# 数据附加系统（Data Attachments）

### 数据附加系统（Data Attachments）

数据附加系统允许模组在方块实体（Block Entities）、区块（Chunks）和实体（Entities）上附加并存储额外的数据。

若要存储额外的世界数据，可以使用 `SavedData`​。

**注意**：物品堆栈（Item Stacks）的数据附加功能已被原版的**数据组件（Data Components）** 取代。

---

#### 创建附加类型（Attachment Type）

要使用该系统，你需要注册一个 `AttachmentType`​。附加类型包含以下配置：

* 一个默认值提供器（Default Value Supplier），用于在首次访问数据时创建实例。
* 一个可选的序列化器（Serializer），用于在需要持久化数据时使用。
* （如果配置了序列化器）`copyOnDeath`​ 标志，用于在实体死亡时自动复制数据（见下文）。

**提示**：如果你不希望附加数据持久化，请不要提供序列化器。

有几种方法可以提供附加数据的序列化器：

* 直接实现 `IAttachmentSerializer`​。
* 实现 `INBTSerializable`​ 并使用静态方法 `AttachmentType#serializable`​ 创建构建器。
* 向构建器提供一个编解码器（Codec）。

无论如何，附加类型必须注册到 `NeoForgeRegistries.ATTACHMENT_TYPES`​ 注册表中。以下是一个示例：

```java
// 创建附加类型的 DeferredRegister
private static final DeferredRegister<AttachmentType<?>> ATTACHMENT_TYPES = DeferredRegister.create(NeoForgeRegistries.ATTACHMENT_TYPES, MOD_ID);

// 通过 INBTSerializable 进行序列化
private static final Supplier<AttachmentType<ItemStackHandler>> HANDLER = ATTACHMENT_TYPES.register(
    "handler", () -> AttachmentType.serializable(() -> new ItemStackHandler(1)).build()
);

// 通过编解码器进行序列化
private static final Supplier<AttachmentType<Integer>> MANA = ATTACHMENT_TYPES.register(
    "mana", () -> AttachmentType.builder(() -> 0).serialize(Codec.INT).build()
);

// 无序列化
private static final Supplier<AttachmentType<SomeCache>> SOME_CACHE = ATTACHMENT_TYPES.register(
    "some_cache", () -> AttachmentType.builder(() -> new SomeCache()).build()
);

// 在模组构造函数中，别忘了将 DeferredRegister 注册到模组总线：
ATTACHMENT_TYPES.register(modBus);
```

---

#### 使用附加类型

一旦附加类型注册完成，就可以在任何持有者对象上使用它。如果数据不存在，调用 `getData`​ 时会附加一个新的默认实例。

```java
// 如果 ItemStackHandler 已经存在，则获取它；否则附加一个新的：
ItemStackHandler stackHandler = chunk.getData(HANDLER);

// 如果玩家法力值可用，则获取当前值；否则附加 0：
int playerMana = player.getData(MANA);

// 其他类似操作...
```

如果不希望附加默认实例，可以添加 `hasData`​ 检查：

```java
// 在操作之前检查区块是否具有 HANDLER 附加数据。
if (chunk.hasData(HANDLER)) {
    ItemStackHandler stackHandler = chunk.getData(HANDLER);
    // 对 chunk.getData(HANDLER) 进行操作。
}
```

还可以使用 `setData`​ 更新数据：

```java
// 将法力值增加 10。
player.setData(MANA, player.getData(MANA) + 10);
```

**重要提示**：通常，修改方块实体和区块时需要将它们标记为脏数据（使用 `setChanged`​ 和 `setUnsaved(true)`​）。对于 `setData`​ 的调用，这会自动完成：

```java
chunk.setData(MANA, chunk.getData(MANA) + 10); // 会自动调用 setUnsaved
```

但如果你修改了通过 `getData`​ 获取的数据（包括新创建的默认实例），则必须显式标记方块实体和区块为脏数据：

```java
var mana = chunk.getData(MUTABLE_MANA);
mana.set(10);
chunk.setUnsaved(true); // 必须手动完成，因为我们没有使用 setData
```

---

#### 与客户端共享数据

要将方块实体、区块或实体的附加数据同步到客户端，你需要自行向客户端发送数据包。对于区块，你可以使用 `ChunkWatchEvent.Sent`​ 来了解何时向玩家发送区块数据。

---

#### 在玩家死亡时复制数据

默认情况下，实体附加数据不会在玩家死亡时复制。要在玩家死亡时自动复制附加数据，请在附加构建器中设置 `copyOnDeath`​。

更复杂的处理可以通过 `PlayerEvent.Clone`​ 实现，通过从原始实体读取数据并将其分配给新实体。在此事件中，可以使用 `#isWasDeath`​ 方法来区分死亡后的重生和从末地返回。这很重要，因为从末地返回时数据已经存在，因此需要小心避免重复值。

例如：

```java
NeoForge.EVENT_BUS.register(PlayerEvent.Clone.class, event -> {
    if (event.isWasDeath() && event.getOriginal().hasData(MY_DATA)) {
        event.getEntity().getData(MY_DATA).fieldToCopy = event.getOriginal().getData(MY_DATA).fieldToCopy;
    }
});
```
