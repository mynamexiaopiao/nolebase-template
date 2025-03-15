# 使用配置任务（Using Configuration Tasks）

### 使用配置任务（Using Configuration Tasks）

客户端和服务器的网络协议有一个特定的阶段，服务器可以在玩家实际加入游戏之前配置客户端。这个阶段称为配置阶段，例如原版服务器使用此阶段将资源包信息发送到客户端。

此阶段也可以由模组用于在玩家加入游戏之前配置客户端。

---

#### 注册配置任务

使用配置阶段的第一步是注册一个配置任务。这可以通过在 `RegisterConfigurationTasksEvent`​ 事件中注册一个新的配置任务来完成。

```java
@SubscribeEvent
public static void register(final RegisterConfigurationTasksEvent event) {
    event.register(new MyConfigurationTask());
}
```

​`RegisterConfigurationTasksEvent`​ 事件在模组总线上触发，并暴露服务器用于配置相关客户端的当前监听器。模组开发者可以使用暴露的监听器来确定客户端是否正在运行该模组，如果是，则注册一个配置任务。

---

#### 实现配置任务

配置任务是一个简单的接口：`ICustomConfigurationTask`​。该接口有两个方法：`void run(Consumer<CustomPacketPayload> sender);`​ 和 `ConfigurationTask.Type type();`​，后者返回配置任务的类型。类型用于标识配置任务。以下是一个配置任务的示例：

```java
public record MyConfigurationTask implements ICustomConfigurationTask {
    public static final ConfigurationTask.Type TYPE = new ConfigurationTask.Type(ResourceLocation.fromNamespaceAndPath("mymod", "my_task"));
    
    @Override
    public void run(final Consumer<CustomPacketPayload> sender) {
        final MyData payload = new MyData();
        sender.accept(payload);
    }

    @Override
    public ConfigurationTask.Type type() {
        return TYPE;
    }
}
```

---

#### 确认配置任务

你的配置在服务器上执行，服务器需要知道何时可以执行下一个配置任务。这是通过确认所述配置任务的执行来完成的。

有两种主要方法可以实现这一点：

---

##### 捕获监听器

当客户端不需要确认配置任务时，可以捕获监听器，并在服务器端直接确认配置任务。

```java
public record MyConfigurationTask(ServerConfigurationPacketListener listener) implements ICustomConfigurationTask {
    public static final ConfigurationTask.Type TYPE = new ConfigurationTask.Type(ResourceLocation.fromNamespaceAndPath("mymod", "my_task"));
    
    @Override
    public void run(final Consumer<CustomPacketPayload> sender) {
        final MyData payload = new MyData();
        sender.accept(payload);
        this.listener().finishCurrentTask(this.type());
    }

    @Override
    public ConfigurationTask.Type type() {
        return TYPE;
    }
}
```

要使用这样的配置任务，需要在 `RegisterConfigurationTasksEvent`​ 事件中捕获监听器。

```java
@SubscribeEvent
public static void register(final RegisterConfigurationTasksEvent event) {
    event.register(new MyConfigurationTask(event.getListener()));
}
```

然后，下一个配置任务将在当前配置任务完成后立即执行，客户端不需要确认配置任务。此外，服务器不会等待客户端正确处理发送的负载。

---

##### 确认配置任务

当客户端需要确认配置任务时，你需要向客户端发送自己的负载：

```java
public record AckPayload() implements CustomPacketPayload {
    public static final CustomPacketPayload.Type<AckPayload> TYPE = new CustomPacketPayload.Type<>(ResourceLocation.fromNamespaceAndPath("mymod", "ack"));
    
    // 无数据的单位编解码器
    public static final StreamCodec<ByteBuf, AckPayload> STREAM_CODEC = StreamCodec.unit(new AckPayload());

    @Override
    public CustomPacketPayload.Type<? extends CustomPacketPayload> type() {
        return TYPE;
    }
}
```

当服务器端配置任务的负载被正确处理时，你可以将此负载发送到服务器以确认配置任务。

```java
public void onMyData(MyData data, IPayloadContext context) {
    context.enqueueWork(() -> {
        blah(data.name());
    })
    .exceptionally(e -> {
        // 处理异常
        context.disconnect(Component.translatable("my_mod.configuration.failed", e.getMessage()));
        return null;
    })
    .thenAccept(v -> {
        context.reply(new AckPayload());
    });     
}
```

其中 `onMyData`​ 是处理服务器端配置任务发送的负载的处理程序。

当服务器接收到此负载时，它将确认配置任务，并执行下一个配置任务：

```java
public void onAck(AckPayload payload, IPayloadContext context) {
    context.finishCurrentTask(MyConfigurationTask.TYPE);
}
```

其中 `onAck`​ 是处理客户端发送的负载的处理程序。

---

#### 阻止登录过程

如果配置任务未被确认，服务器将永远等待，客户端将永远不会加入游戏。因此，除非配置任务失败，否则始终确认配置任务非常重要，否则你可以断开客户端的连接。
