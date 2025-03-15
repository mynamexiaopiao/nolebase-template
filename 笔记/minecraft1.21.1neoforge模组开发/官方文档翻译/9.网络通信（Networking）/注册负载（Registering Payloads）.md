# 注册负载（Registering Payloads）

### 注册负载（Registering Payloads）

负载（Payloads）是一种在客户端和服务器之间发送任意数据的方式。它们通过 `RegisterPayloadHandlersEvent`​ 事件中的 `PayloadRegistrar`​ 进行注册。

```java
@SubscribeEvent
public static void register(final RegisterPayloadHandlersEvent event) {
    // 设置当前网络版本
    final PayloadRegistrar registrar = event.registrar("1");
}
```

假设我们想要发送以下数据：

```java
public record MyData(String name, int age) {}
```

我们可以实现 `CustomPacketPayload`​ 接口来创建一个负载，用于发送和接收这些数据。

```java
public record MyData(String name, int age) implements CustomPacketPayload {
    
    public static final CustomPacketPayload.Type<MyData> TYPE = new CustomPacketPayload.Type<>(ResourceLocation.fromNamespaceAndPath("mymod", "my_data"));

    // 每对元素定义了要编码/解码的元素的流编解码器以及要编码的元素的 getter
    // 'name' 将被编码和解码为字符串
    // 'age' 将被编码和解码为整数
    // 最后一个参数按提供的顺序接受前面的参数以构造负载对象
    public static final StreamCodec<ByteBuf, MyData> STREAM_CODEC = StreamCodec.composite(
        ByteBufCodecs.STRING_UTF8,
        MyData::name,
        ByteBufCodecs.VAR_INT,
        MyData::age,
        MyData::new
    );
    
    @Override
    public CustomPacketPayload.Type<? extends CustomPacketPayload> type() {
        return TYPE;
    }
}
```

从上面的示例中可以看出，`CustomPacketPayload`​ 接口要求我们实现 `type`​ 方法。`type`​ 方法负责返回此负载的唯一标识符。然后我们还需要一个读取器来注册 `StreamCodec`​，以便读取和写入负载数据。

最后，我们可以将此负载注册到注册器中：

```java
@SubscribeEvent
public static void register(final RegisterPayloadHandlersEvent event) {
    final PayloadRegistrar registrar = event.registrar("1");
    registrar.playBidirectional(
        MyData.TYPE,
        MyData.STREAM_CODEC,
        new DirectionalPayloadHandler<>(
            ClientPayloadHandler::handleDataOnMain,
            ServerPayloadHandler::handleDataOnMain
        )
    );
}
```

分析上面的代码，我们可以注意到以下几点：

* 注册器有 `play*`​ 方法，可用于注册在游戏进行阶段发送的负载。
* 代码中未显示的是 `configuration*`​ 和 `common*`​ 方法；然而，它们也可以用于注册配置阶段的负载。`common`​ 方法可以同时用于注册配置阶段和游戏阶段的负载。
* 注册器使用 `*Bidirectional`​ 方法，可用于注册发送到逻辑服务器和逻辑客户端的负载。
* 代码中未显示的是 `*ToClient`​ 和 `*ToServer`​ 方法；然而，它们也可以分别用于仅注册到逻辑客户端或逻辑服务器的负载。
* 负载的类型用作负载的唯一标识符。
* 流编解码器用于从网络发送的缓冲区中读取和写入负载。
* 负载处理程序是负载到达逻辑端时的回调。
* 如果使用 `*Bidirectional`​ 方法，则可以使用 `DirectionalPayloadHandler`​ 为每个逻辑端提供单独的处理程序。

现在我们已经注册了负载，我们需要实现一个处理程序。对于此示例，我们将特别查看客户端处理程序，但服务器端处理程序非常相似。

```java
public class ClientPayloadHandler {
    
    public static void handleDataOnMain(final MyData data, final IPayloadContext context) {
        // 在主线程上处理数据
        blah(data.age());
    }
}
```

这里有几点需要注意：

* 这里的处理方法接收负载和一个上下文对象。
* 默认情况下，负载的处理方法在主线程上调用。
* 如果你需要进行一些资源密集型的计算，则应在网络线程上完成工作，而不是阻塞主线程。这可以通过在注册负载之前将 `PayloadRegistrar`​ 的 `HandlerThread`​ 设置为 `HandlerThread#NETWORK`​ 来实现。

```java
@SubscribeEvent
public static void register(final RegisterPayloadHandlersEvent event) {
    final PayloadRegistrar registrar = event.registrar("1")
        .executesOn(HandlerThread.NETWORK); // 所有后续负载将在网络线程上注册
    registrar.playBidirectional(
        MyData.TYPE,
        MyData.STREAM_CODEC,
        new DirectionalPayloadHandler<>(
            ClientPayloadHandler::handleDataOnNetwork,
            ServerPayloadHandler::handleDataOnNetwork
        )
    );
}
```

**注意**：所有在 `executesOn`​ 调用之后注册的负载将保留相同的线程执行位置，直到再次调用 `executesOn`​。

```java
PayloadRegistrar registrar = event.registrar("1");

registrar.playBidirectional(...); // 在主线程上
registrar.playBidirectional(...); // 在主线程上

// 配置方法通过创建新实例来修改注册器的状态，
// 因此需要通过存储结果来更新更改
registrar = registrar.executesOn(HandlerThread.NETWORK);

registrar.playBidirectional(...); // 在网络线程上
registrar.playBidirectional(...); // 在网络线程上

registrar = registrar.executesOn(HandlerThread.MAIN);

registrar.playBidirectional(...); // 在主线程上
registrar.playBidirectional(...); // 在主线程上
```

这里有几点需要注意：

* 如果你想在主游戏线程上运行代码，可以使用 `enqueueWork`​ 将任务提交到主线程。
* 该方法将返回一个 `CompletableFuture`​，它将在主线程上完成。
* 注意：返回了一个 `CompletableFuture`​，这意味着你可以将多个任务链接在一起，并在一个地方处理异常。
* 如果你不在 `CompletableFuture`​ 中处理异常，则它将被吞没，你将不会收到通知。

```java
public class ClientPayloadHandler {
    
    public static void handleDataOnNetwork(final MyData data, final IPayloadContext context) {
        // 在网络线程上处理数据
        blah(data.name());
        
        // 在主线程上处理数据
        context.enqueueWork(() -> {
            blah(data.age());
        })
        .exceptionally(e -> {
            // 处理异常
            context.disconnect(Component.translatable("my_mod.networking.failed", e.getMessage()));
            return null;
        });
    }
}
```

通过你自己的负载，你可以使用它们来配置客户端和服务器。

---

#### 发送负载（Sending Payloads）

​`CustomPacketPayload`​ 通过将负载包装在 `ServerboundCustomPayloadPacket`​（发送到服务器时）或 `ClientboundCustomPayloadPacket`​（发送到客户端时）中，通过原版的包系统发送。发送到客户端的负载最多只能包含 1 MiB 的数据，而发送到服务器的负载只能包含少于 32 KiB 的数据。

所有负载都通过 `Connection#send`​ 发送，带有一定程度的抽象；然而，如果你想根据给定条件向多个人发送数据包，调用这些方法通常不太方便。因此，`PacketDistributor`​ 包含了许多方便的实现来发送负载。只有一个方法可以将数据包发送到服务器（`sendToServer`​）；然而，有许多方法可以根据哪些玩家应接收负载来将数据包发送到客户端。

```java
// 在客户端

// 发送负载到服务器
PacketDistributor.sendToServer(new MyData(...));

// 在服务器

// 发送给一个玩家（ServerPlayer serverPlayer）
PacketDistributor.sendToPlayer(serverPlayer, new MyData(...));

/// 发送给跟踪此区块的所有玩家（ServerLevel serverLevel, ChunkPos chunkPos）
PacketDistributor.sendToPlayersTrackingChunk(serverLevel, chunkPos, new MyData(...));

/// 发送给所有连接的玩家
PacketDistributor.sendToAllPlayers(new MyData(...));
```

有关更多实现，请参阅 `PacketDistributor`​ 类。
