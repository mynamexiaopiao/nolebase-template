# 实体（Entities）

### 实体（Entities）

除了常规的网络消息外，Minecraft 还提供了各种其他系统来处理实体数据的同步。

---

#### 生成数据（Spawn Data）

自 1.20.2 版本以来，Mojang 引入了**捆绑数据包（Bundle packets）** 的概念，用于将实体生成数据包一起发送。这允许在生成数据包中发送更多数据，并且可以更高效地发送这些数据。

你可以通过实现以下接口向 NeoForge 发送的生成数据包添加额外数据。

---

##### `IEntityWithComplexSpawn`​

如果你的实体有需要在客户端使用的数据，但这些数据不会随时间变化，则可以使用此接口将这些数据添加到实体生成数据包中。`#writeSpawnData`​ 和 `#readSpawnData`​ 控制如何将数据编码到网络缓冲区或从网络缓冲区解码。或者，你可以重写 `IEntityExtension#sendPairingData`​ 方法，该方法在实体的初始数据发送到客户端时调用。此方法在服务器上调用，可用于在生成数据包的同一捆绑包中向客户端发送额外的负载。

---

#### 动态数据参数（Dynamic Data Parameters）

这是原版用于将实体数据从服务器同步到客户端的主要系统。因此，有许多原版示例可供参考。

首先，你需要一个 `EntityDataAccessor<T>`​ 来保存你想要同步的数据。这应该作为静态最终字段存储在实体类中，通过调用 `SynchedEntityData#defineId`​ 并传递实体类和数据类型的序列化器来获取。可用的序列化器实现可以在 `EntityDataSerializers`​ 类中的静态常量中找到。

**注意**：你应该只为你自己的实体创建数据参数，并在该实体的类中定义。向你不控制的实体添加参数可能会导致用于通过网络发送数据的 ID 不同步，从而导致难以调试的崩溃。

然后，重写 `Entity#defineSynchedData`​ 并为每个数据参数调用 `SynchedEntityData.Builder#define`​，传递参数和初始值。记住始终首先调用父类方法！

你可以通过实体的 `entityData`​ 实例获取和设置这些值。所做的更改将自动同步到客户端。

---

#### 示例代码

以下是一个示例，展示了如何为自定义实体添加动态数据参数：

```java
public class MyEntity extends LivingEntity {

    // 定义数据参数
    private static final EntityDataAccessor<Boolean> IS_ACTIVE = SynchedEntityData.defineId(MyEntity.class, EntityDataSerializers.BOOLEAN);

    public MyEntity(EntityType<? extends LivingEntity> type, Level level) {
        super(type, level);
    }

    @Override
    protected void defineSynchedData() {
        super.defineSynchedData();
        // 定义数据参数并设置初始值
        this.entityData.define(IS_ACTIVE, false);
    }

    // 获取和设置数据参数
    public boolean isActive() {
        return this.entityData.get(IS_ACTIVE);
    }

    public void setActive(boolean active) {
        this.entityData.set(IS_ACTIVE, active);
    }
}
```
