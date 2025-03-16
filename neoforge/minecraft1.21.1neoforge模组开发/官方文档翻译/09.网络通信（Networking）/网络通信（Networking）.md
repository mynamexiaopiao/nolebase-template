# 网络通信（Networking）

### 网络通信（Networking）

服务器和客户端之间的通信是成功实现模组的基石。

网络通信有两个主要目标：

1. **确保客户端视图与服务器视图“同步”** 
    例如：坐标 (X, Y, Z) 处的花朵刚刚生长。
2. **为客户端提供一种方式告诉服务器玩家的某些状态发生了变化**
    例如：玩家按下了某个键。

实现这些目标的最常见方法是在客户端和服务器之间传递消息。这些消息通常是结构化的，包含特定排列的数据，以便于发送和接收。

NeoForge 提供了一种基于 Netty 的通信技术，可以通过监听 `RegisterPayloadHandlersEvent`​ 事件来使用。在该事件中，你可以注册特定类型的负载（Payload）、其读取器（Reader）以及处理函数到注册器中。

---

#### 使用 NeoForge 的网络通信

NeoForge 的网络通信系统基于负载（Payload）的概念。负载是一个包含数据的对象，可以在客户端和服务器之间传递。以下是实现网络通信的基本步骤：

1. **定义负载（Payload）** 
    负载是一个包含需要传递的数据的对象。它需要实现 `CustomPacketPayload`​ 接口。
2. **注册负载和处理函数**
    在 `RegisterPayloadHandlersEvent`​ 事件中注册负载及其处理函数。
3. **发送负载**
    在需要时，客户端或服务器可以发送负载到另一端。
4. **处理负载**
    在接收端，处理函数会处理接收到的负载并执行相应的逻辑。

---

#### 示例：实现一个简单的网络通信

以下是一个简单的示例，展示了如何实现客户端和服务器之间的网络通信。

##### 1. 定义负载

首先，定义一个负载类，包含需要传递的数据。

```java
import net.minecraft.network.FriendlyByteBuf;
import net.minecraft.network.protocol.common.custom.CustomPacketPayload;
import net.minecraft.resources.ResourceLocation;
import net.neoforged.neoforge.network.handling.IPayloadContext;

public record ExamplePayload(String message) implements CustomPacketPayload {

    public static final ResourceLocation ID = new ResourceLocation("examplemod", "example_payload");

    public ExamplePayload(FriendlyByteBuf buf) {
        this(buf.readUtf());
    }

    @Override
    public void write(FriendlyByteBuf buf) {
        buf.writeUtf(message);
    }

    @Override
    public ResourceLocation id() {
        return ID;
    }
}
```

##### 2. 注册负载和处理函数

在 `RegisterPayloadHandlersEvent`​ 事件中注册负载及其处理函数。

```java
import net.neoforged.bus.api.SubscribeEvent;
import net.neoforged.fml.common.Mod;
import net.neoforged.neoforge.network.event.RegisterPayloadHandlersEvent;
import net.neoforged.neoforge.network.registration.PayloadRegistrar;

@Mod("examplemod")
public class ExampleMod {

    @SubscribeEvent
    public void onRegisterPayloadHandlers(RegisterPayloadHandlersEvent event) {
        PayloadRegistrar registrar = event.registrar("examplemod");

        // 注册负载及其处理函数
        registrar.play(ExamplePayload.ID, ExamplePayload::new, handler -> handler
            .client(ExamplePayload::handleClient)
            .server(ExamplePayload::handleServer)
        );
    }
}
```

##### 3. 处理负载

在负载类中添加处理函数，用于在客户端和服务器上处理接收到的负载。

```java
public record ExamplePayload(String message) implements CustomPacketPayload {

    // ...（省略其他代码）

    public void handleClient(IPayloadContext context) {
        // 在客户端处理负载
        System.out.println("Received message on client: " + message);
    }

    public void handleServer(IPayloadContext context) {
        // 在服务器处理负载
        System.out.println("Received message on server: " + message);
    }
}
```

##### 4. 发送负载

在需要时，客户端或服务器可以发送负载到另一端。

```java
import net.minecraft.server.level.ServerPlayer;
import net.neoforged.neoforge.network.PacketDistributor;

public class NetworkHandler {

    public static void sendToServer(String message) {
        // 从客户端发送负载到服务器
        PacketDistributor.SERVER.noArg().send(new ExamplePayload(message));
    }

    public static void sendToClient(ServerPlayer player, String message) {
        // 从服务器发送负载到特定客户端
        PacketDistributor.PLAYER.with(player).send(new ExamplePayload(message));
    }

    public static void sendToAllClients(String message) {
        // 从服务器发送负载到所有客户端
        PacketDistributor.ALL.noArg().send(new ExamplePayload(message));
    }
}
```

---

#### 总结

通过以上步骤，你可以实现客户端和服务器之间的网络通信。NeoForge 的网络通信系统基于负载的概念，使得消息的发送和处理变得简单而高效。

1. **定义负载**：创建一个包含数据的负载类。
2. **注册负载和处理函数**：在 `RegisterPayloadHandlersEvent`​ 事件中注册负载及其处理函数。
3. **发送负载**：在需要时，客户端或服务器可以发送负载到另一端。
4. **处理负载**：在接收端，处理函数会处理接收到的负载并执行相应的逻辑。

希望这些内容对你的开发有所帮助！如果有其他问题，欢迎随时提问。
