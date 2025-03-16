# 粒子（Particles）

### 粒子（Particles）

粒子是 2D 效果，用于增强游戏的视觉效果和沉浸感。它们可以在客户端和服务器端生成，但由于其主要是视觉性质，关键部分仅存在于物理（和逻辑）客户端。

---

### 注册粒子

#### ParticleType

粒子使用 `ParticleType`​ 注册。它们的工作方式类似于 `EntityType`​ 或 `BlockEntityType`​，即每个生成的粒子都是 `Particle`​ 类的一个实例，而 `ParticleType`​ 类则保存一些通用信息，用于注册。`ParticleType`​ 是一个注册表，这意味着我们希望像所有其他注册对象一样使用 `DeferredRegister`​ 来注册它们：

```java
public class MyParticleTypes {
    // 假设你的模组 ID 是 examplemod
    public static final DeferredRegister<ParticleType<?>> PARTICLE_TYPES =
        DeferredRegister.create(BuiltInRegistries.PARTICLE_TYPE, "examplemod");
    
    // 添加新粒子类型的最简单方法是重用原版的 SimpleParticleType。
    // 也可以实现自定义的 ParticleType，见下文。
    public static final DeferredHolder<ParticleType<?>, SimpleParticleType> MY_PARTICLE = PARTICLE_TYPES.register(
        // 粒子类型的名称。
        "my_particle",
        // 供应商。布尔参数表示在视频设置中将粒子选项设置为“最小”时是否会影响此粒子类型；
        // 对于大多数原版粒子，这是 false，但对于例如爆炸、篝火烟雾或墨鱼墨汁，这是 true。
        () -> new SimpleParticleType(false)
    );
}
```

**注意**
只有在需要在服务器端处理粒子时，才需要 `ParticleType`​。客户端也可以直接使用 `Particle`​。

---

#### Particle

​`Particle`​ 是最终生成到世界中并显示给玩家的内容。虽然你可以扩展 `Particle`​ 并自己实现内容，但在许多情况下，扩展 `TextureSheetParticle`​ 会更好，因为该类提供了诸如动画和缩放的辅助方法，并且还为你处理实际渲染（如果你直接扩展 `Particle`​，则需要自己实现这些内容）。

​`Particle`​ 的大多数属性由字段控制，例如 `gravity`​、`lifetime`​、`hasPhysics`​、`friction`​ 等。唯一需要自己实现的两个方法是 `tick`​ 和 `move`​，它们的功能正如你所期望的那样。因此，自定义粒子类通常很短，例如仅包含一个设置一些字段并让父类处理其余内容的构造函数。一个基本的实现可能如下所示：

```java
public class MyParticle extends TextureSheetParticle {
    private final SpriteSet spriteSet;
    
    // 前四个参数是不言自明的。SpriteSet 参数由 ParticleProvider 提供，见下文。
    // 你也可以根据需要添加其他参数，例如 xSpeed/ySpeed/zSpeed。
    public MyParticle(ClientLevel level, double x, double y, double z, SpriteSet spriteSet) {
        super(level, x, y, z);
        this.spriteSet = spriteSet;
        this.gravity = 0; // 我们的粒子现在漂浮在半空中，因为为什么不呢。

        // 我们在此处设置初始精灵，因为不能保证在调用渲染方法之前会通过 ticking 设置精灵。
        this.setSpriteFromAge(spriteSet);
    }
    
    @Override
    public void tick() {
        // 为当前粒子年龄设置精灵，即推进动画。
        this.setSpriteFromAge(spriteSet);
        // 让父类处理进一步的移动。如果需要，你可以用自己的移动替换此内容。
        // 如果你只想修改内置移动，也可以覆盖 move()。
        super.tick();
    }
}
```

---

#### ParticleProvider

接下来，粒子类型必须注册一个 `ParticleProvider`​。`ParticleProvider`​ 是一个仅客户端的类，负责通过 `createParticle`​ 方法实际创建我们的 `Particle`​。虽然可以在此处包含更复杂的代码，但许多粒子提供者都像这样简单：

```java
// ParticleProvider 的泛型类型必须与此提供者所针对的粒子类型匹配。
public class MyParticleProvider implements ParticleProvider<SimpleParticleType> {
    // 一组粒子精灵。
    private final SpriteSet spriteSet;
    
    // 注册函数传递一个 SpriteSet，因此我们接受它并存储以供进一步使用。
    public MyParticleProvider(SpriteSet spriteSet) {
        this.spriteSet = spriteSet;
    }
    
    // 这是魔法发生的地方。每次调用此方法时，我们都会返回一个新粒子！
    // 第一个参数的类型与传递给父接口的泛型类型匹配。
    @Override
    public Particle createParticle(SimpleParticleType type, ClientLevel level,
            double x, double y, double z, double xSpeed, double ySpeed, double zSpeed) {
        // 我们不使用类型和速度，并传递其他所有内容。当然，如果需要，你也可以使用它们。
        return new MyParticle(level, x, y, z, spriteSet);
    }
}
```

然后，你的粒子提供者必须在客户端模组总线事件 `RegisterParticleProvidersEvent`​ 中与粒子类型关联：

```java
@SubscribeEvent
public static void registerParticleProviders(RegisterParticleProvidersEvent event) {
    // 有多种注册提供者的方法，它们在第二个参数中提供的函数类型不同。
    // 例如，#registerSpriteSet 表示一个 Function<SpriteSet, ParticleProvider<?>>：
    event.registerSpriteSet(MyParticleTypes.MY_PARTICLE.get(), MyParticleProvider::new);
    // 其他方法包括 #registerSprite，它本质上是一个 Supplier<TextureSheetParticle>，
    // 以及 #registerSpecial，它映射到 Supplier<Particle>。有关更多信息，请参见事件的源代码。
}
```

---

### 粒子描述（Particle Descriptions）

最后，我们必须将我们的粒子类型与纹理关联。类似于物品如何与物品模型关联，我们将粒子类型与所谓的粒子描述关联。粒子描述是位于 `assets/<namespace>/particles`​ 目录中的 JSON 文件，并且与粒子类型同名（例如，上述示例中的 `my_particle.json`​）。粒子定义 JSON 具有以下格式：

```json
{
    // 将按顺序播放的纹理列表。如果需要，将循环播放。
    // 纹理位置相对于 textures/particle 文件夹。
    "textures": [
        "examplemod:my_particle_0",
        "examplemod:my_particle_1",
        "examplemod:my_particle_2",
        "examplemod:my_particle_3"
    ]
}
```

当使用接受 `SpriteSet`​ 的粒子时，需要粒子定义，这是在通过 `registerSpriteSet`​ 或 `registerSprite`​ 注册粒子提供者时完成的。对于通过 `#registerSpecial`​ 注册的粒子提供者，不得提供粒子定义。

**危险**
不匹配的精灵集粒子工厂和粒子定义文件列表（即没有相应粒子工厂的粒子描述，反之亦然）将抛出异常！

**注意**
虽然粒子描述必须以某种方式注册提供者，但它们仅在 `ParticleRenderType`​（通过 `Particle#getRenderType`​ 设置）使用 `TextureAtlas#LOCATION_PARTICLES`​ 作为着色器纹理时使用。对于原版渲染类型，这些是 `PARTICLE_SHEET_OPAQUE`​、`PARTICLE_SHEET_TRANSLUCENT`​ 和 `PARTICLE_SHEET_LIT`​。

---

### 数据生成（Datagen）

粒子定义文件也可以通过扩展 `ParticleDescriptionProvider`​ 并覆盖 `#addDescriptions()`​ 方法来生成数据：

```java
public class MyParticleDescriptionProvider extends ParticleDescriptionProvider {
    // 从 GatherDataEvent 获取参数。
    public AMParticleDefinitionsProvider(PackOutput output, ExistingFileHelper existingFileHelper) {
        super(output, existingFileHelper);
    }

    // 假设所有引用的粒子实际存在。将 "examplemod" 替换为你的模组 ID。
    @Override
    protected void addDescriptions() {
        // 添加一个单精灵粒子定义，文件位于
        // assets/examplemod/textures/particle/my_single_particle.png。
        sprite(MyParticleTypes.MY_SINGLE_PARTICLE.get(), ResourceLocation.fromNamespaceAndPath("examplemod", "my_single_particle"));
        // 添加一个多精灵粒子定义，带有可变参数。也可以接受列表。
        spriteSet(MyParticleTypes.MY_MULTI_PARTICLE.get(),
            ResourceLocation.fromNamespaceAndPath("examplemod", "my_multi_particle_0"),
            ResourceLocation.fromNamespaceAndPath("examplemod", "my_multi_particle_1"),
            ResourceLocation.fromNamespaceAndPath("examplemod", "my_multi_particle_2")
        );
        // 上述的替代方法，为给定数量的纹理将 "_<index>" 附加到给定的基本名称。
        spriteSet(MyParticleTypes.MY_ALT_MULTI_PARTICLE.get(),
            // 基本名称。
            ResourceLocation.fromNamespaceAndPath("examplemod", "my_multi_particle"),
            // 纹理数量。
            3,
            // 是否反转列表，即从最后一个元素开始而不是第一个元素。
            false
        );
    }
}
```

别忘了将提供者添加到 `GatherDataEvent`​：

```java
@SubscribeEvent
public static void gatherData(GatherDataEvent event) {
    DataGenerator generator = event.getGenerator();
    PackOutput output = generator.getPackOutput();
    ExistingFileHelper existingFileHelper = event.getExistingFileHelper();

    // 其他提供者在这里
    generator.addProvider(
        event.includeClient(),
        new MyParticleDescriptionProvider(output, existingFileHelper)
    );
}
```

---

### 自定义 ParticleType

虽然对于大多数情况 `SimpleParticleType`​ 已经足够，但有时需要在服务器端将附加数据附加到粒子上。这就是自定义 `ParticleType`​ 和关联的自定义 `ParticleOptions`​ 的用武之地。让我们从 `ParticleOptions`​ 开始，因为这是实际存储信息的地方：

```java
public class MyParticleOptions implements ParticleOptions {
    
    // 读取和写入信息，通常用于命令。
    // 由于此类型中没有信息，因此这将是一个空字符串。
    public static final MapCodec<MyParticleOptions> CODEC = MapCodec.unit(new MyParticleOptions());

    // 读取和写入信息到网络缓冲区。
    public static final StreamCodec<ByteBuf, MyParticleOptions> STREAM_CODEC = StreamCodec.unit(new MyParticleOptions());

    // 不需要任何参数，但可以定义粒子工作所需的任何字段。
    public MyParticleOptions() {}
}
```

然后我们在自定义的 `ParticleType`​ 中使用此 `ParticleOptions`​ 实现...

```java
public class MyParticleType extends ParticleType<MyParticleOptions> {
    // 布尔参数再次确定是否在较低的粒子设置下限制粒子。
    // 有关更多信息，请参见本文顶部附近的 MyParticleTypes 类的实现。
    public MyParticleType(boolean overrideLimiter) {
        // 将反序列化器传递给父类。
        super(overrideLimiter);
    }

    @Override
    public MapCodec<MyParticleOptions> codec() {
        return MyParticleOptions.CODEC;
    }

    @Override
    public StreamCodec<? super RegistryFriendlyByteBuf, MyParticleOptions> streamCodec() {
        return MyParticleOptions.STREAM_CODEC;
    }
}
```

...并在注册期间引用它：

```java
public static final Supplier<MyParticleType> MY_CUSTOM_PARTICLE = PARTICLE_TYPES.register(
    "my_custom_particle",
    () -> new MyParticleType(false)
);
```

---

### 生成粒子

作为提醒，服务器只知道 `ParticleType`​ 和 `ParticleOptions`​，而客户端直接使用与 `ParticleType`​ 关联的 `ParticleProvider`​ 提供的 `Particle`​。因此，生成粒子的方式在客户端和服务器端之间差异很大。

* **通用代码**：调用 `Level#addParticle`​ 或 `Level#addAlwaysVisibleParticle`​。这是创建对所有人可见的粒子的首选方法。
* **客户端代码**：使用通用代码方式。或者，使用你选择的粒子类创建一个新的 `Particle()`​，并使用该粒子调用 `Minecraft.getInstance().particleEngine#add(Particle)`​。请注意，以这种方式添加的粒子仅对客户端显示，因此其他玩家不可见。
* **服务器代码**：调用 `ServerLevel#sendParticles`​。原版中的 `/particle`​ 命令使用此方法。
