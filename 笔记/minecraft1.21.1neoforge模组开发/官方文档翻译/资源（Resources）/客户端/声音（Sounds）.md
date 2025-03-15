# 声音（Sounds）

### 声音（Sounds）

声音虽然不是必需的，但它们可以使模组感觉更加细腻和生动。Minecraft 提供了多种注册和播放声音的方式，本文将详细介绍这些内容。

---

### 术语

Minecraft 的声音引擎使用了许多术语来指代不同的事物：

* **声音事件（Sound Event）** ：声音事件是一个代码触发器，告诉声音引擎播放某个声音。`SoundEvent`​ 也是你注册到游戏中的内容。
* **声音类别或声音源（Sound Category/Sound Source）** ：声音类别是声音的粗略分组，可以单独切换。声音选项 GUI 中的滑块代表这些类别：`master`​、`block`​、`player`​ 等。在代码中，它们可以在 `SoundSource`​ 枚举中找到。
* **声音定义（Sound Definition）** ：将声音事件映射到一个或多个声音对象，加上一些可选的元数据。声音定义位于命名空间的 `sounds.json`​ 文件中。
* **声音对象（Sound Object）** ：一个 JSON 对象，包含声音文件的位置以及一些可选的元数据。
* **声音文件（Sound File）** ：磁盘上的声音文件。Minecraft 仅支持 `.ogg`​ 格式的声音文件。

**危险**
由于 OpenAL（Minecraft 的音频库）的实现，为了使你的声音具有衰减（即根据玩家与声音的距离使其变弱或变强），你的声音文件必须是单声道（单通道）的。立体声（多通道）声音文件不会受到衰减的影响，并且始终在玩家的位置播放，因此它们非常适合环境声音和背景音乐。参见 [MC-146721](https://bugs.mojang.com/browse/MC-146721)。

---

### 创建 SoundEvent

​`SoundEvent`​ 是注册对象，这意味着它们必须通过 `DeferredRegister`​ 注册到游戏中，并且是单例的：

```java
public class MySoundsClass {
    // 假设你的模组 ID 是 examplemod
    public static final DeferredRegister<SoundEvent> SOUND_EVENTS =
            DeferredRegister.create(BuiltInRegistries.SOUND_EVENT, "examplemod");
    
    // 所有原版声音都使用可变范围事件。
    public static final DeferredHolder<SoundEvent, SoundEvent> MY_SOUND = SOUND_EVENTS.register(
            "my_sound", // 必须与下一行的资源位置匹配
            () -> SoundEvent.createVariableRangeEvent(ResourceLocation.fromNamespaceAndPath("examplemod", "my_sound"))
    );
    
    // 还有一个当前未使用的方法来注册固定范围（= 无衰减）事件：
    public static final DeferredHolder<SoundEvent, SoundEvent> MY_FIXED_SOUND = SOUND_EVENTS.register("my_fixed_sound",
            // 16 是声音的默认范围。请注意，由于 OpenAL 的限制，
            // 大于 16 的值无效，并将被限制为 16。
            () -> SoundEvent.createFixedRangeEvent(ResourceLocation.fromNamespaceAndPath("examplemod", "my_fixed_sound"), 16)
    );
}
```

当然，别忘了在模组构造函数中将你的注册表添加到模组事件总线：

```java
public ExampleMod(IEventBus modBus) {
    MySoundsClass.SOUND_EVENTS.register(modBus);
    // 其他内容
}
```

就这样，你已经创建了一个声音事件！

---

### sounds.json

参见：[Minecraft Wiki 上的 sounds.json](https://minecraft.fandom.com/wiki/Sounds.json)

现在，为了将你的声音事件与实际的声音文件关联起来，我们需要创建声音定义。一个命名空间的所有声音定义都存储在一个名为 `sounds.json`​ 的文件中，也称为声音定义文件，直接位于命名空间的根目录中。每个声音定义都是声音事件 ID（例如 `my_sound`​）到 JSON 声音对象的映射。请注意，声音事件 ID 不指定命名空间，因为命名空间已经由声音定义文件所在的命名空间确定。一个示例 `sounds.json`​ 文件可能如下所示：

```json
{
    // 声音事件 "examplemod:my_sound" 的声音定义
    "my_sound": {
        // 声音对象列表。如果包含多个元素，将随机选择一个元素。
        "sounds": [
            // 只有 name 是必需的，其他属性都是可选的。
            {
                // 声音文件的位置，相对于命名空间的 sounds 文件夹。
                // 此示例引用位于 assets/examplemod/sounds/sound_1.ogg 的声音文件。
                "name": "examplemod:sound_1",
                // 可以是 "sound" 或 "event"。"sound" 使 name 引用声音文件。
                // "event" 使 name 引用另一个声音事件。默认为 "sound"。
                "type": "sound",
                // 播放此声音的音量。必须在 0.0 到 1.0（默认）之间。
                "volume": 0.8,
                // 播放此声音的音高值。
                // 必须在 0.0 到 2.0 之间。默认为 1.0。
                "pitch": 1.1,
                // 从声音列表中选择声音时的权重。默认为 1。
                "weight": 3,
                // 如果为 true，声音将从文件流式传输，而不是一次性加载。
                // 推荐用于超过几秒钟的声音文件。默认为 false。
                "stream": true,
                // 手动覆盖衰减距离。默认为 16。固定范围声音事件忽略此属性。
                "attenuation_distance": 8,
                // 如果为 true，声音将在资源包加载时加载到内存中，而不是在播放声音时加载。
                // 原版用于水下环境声音。默认为 false。
                "preload": true
            },
            // { "name": "examplemod:sound_2" } 的简写
            "examplemod:sound_2"
        ]
    },
    "my_fixed_sound": {
        // 可选。如果为 true，替换其他资源包中的声音，而不是添加到它们。
        // 有关更多信息，请参见下面的合并章节。
        "replace": true,
        // 触发此声音事件时显示的字幕的翻译键。
        "subtitle": "examplemod.my_fixed_sound",
        "sounds": [
            "examplemod:sound_1",
            "examplemod:sound_2"
        ]
    }
}
```

---

### 合并（Merging）

与大多数其他资源文件不同，`sounds.json`​ 不会覆盖位于其下方的资源包中的值。相反，它们会被合并在一起，然后作为一个组合的 `sounds.json`​ 文件进行解释。考虑在两个不同的资源包 `RP1`​ 和 `RP2`​ 中定义的声音 `sound_1`​、`sound_2`​、`sound_3`​ 和 `sound_4`​，其中 `RP2`​ 位于 `RP1`​ 下方：

​`RP1`​ 中的 `sounds.json`​：

```json
{
    "sound_1": {
        "sounds": [
            "sound_1"
        ]
    },
    "sound_2": {
        "replace": true,
        "sounds": [
            "sound_2"
        ]
    },
    "sound_3": {
        "sounds": [
            "sound_3"
        ]
    },
    "sound_4": {
        "replace": true,
        "sounds": [
            "sound_4"
        ]
    }
}
```

​`RP2`​ 中的 `sounds.json`​：

```json
{
    "sound_1": {
        "sounds": [
            "sound_5"
        ]
    },
    "sound_2": {
        "sounds": [
            "sound_6"
        ]
    },
    "sound_3": {
        "replace": true,
        "sounds": [
            "sound_7"
        ]
    },
    "sound_4": {
        "replace": true,
        "sounds": [
            "sound_8"
        ]
    }
}
```

然后，游戏将使用组合（合并）的 `sounds.json`​ 文件来加载声音，该文件可能如下所示（仅在内存中，此文件不会写入任何地方）：

```json
{
    "sound_1": {
        // replace false 和 false：从下层包添加，然后从上层包添加
        "sounds": [
            "sound_5",
            "sound_1"
        ]
    },
    "sound_2": {
        // 上层包中 replace true，下层包中 replace false：仅从上层包添加
        "sounds": [
            "sound_2"
        ]
    },
    "sound_3": {
        // 上层包中 replace false，下层包中 replace true：从下层包添加，然后从上层包添加
        // 仍然会丢弃位于 RP2 下方的第三个资源包中的值
        "sounds": [
            "sound_7",
            "sound_3"
        ]
    },
    "sound_4": {
        // replace true 和 true：仅从上层包添加
        "sounds": [
            "sound_8"
        ]
    }
}
```

---

### 播放声音

Minecraft 提供了多种播放声音的方法，有时不清楚应该使用哪种方法。所有方法都接受一个 `SoundEvent`​，它可以是你自己的，也可以是原版的（原版声音事件可以在 `SoundEvents`​ 类中找到）。对于以下方法描述，客户端和服务器分别指逻辑客户端和逻辑服务器。

#### Level

* ​**​`playSeededSound(Player player, double x, double y, double z, Holder<SoundEvent> soundEvent, SoundSource soundSource, float volume, float pitch, long seed)`​** ​

  * **客户端行为**：如果传入的玩家是本地玩家，则在给定位置向玩家播放声音事件，否则无操作。
  * **服务器行为**：向除传入玩家之外的所有玩家发送一个数据包，指示客户端在给定位置播放声音事件。
  * **用法**：从将在双方运行的客户端启动代码中调用。服务器不向启动玩家播放声音事件，以防止向他们播放两次声音。或者，从服务器启动代码（例如方块实体）中调用，传入 `null`​ 作为玩家，以向所有人播放声音。
* ​**​`playSound(Player player, double x, double y, double z, SoundEvent soundEvent, SoundSource soundSource, float volume, float pitch)`​** ​

  * 转发到 `playSeededSound`​，使用随机选择的种子并将 `Holder`​ 包装在 `SoundEvent`​ 周围。
* ​**​`playSound(Player player, BlockPos pos, SoundEvent soundEvent, SoundSource soundSource, float volume, float pitch)`​** ​

  * 转发到上述方法，`x`​、`y`​ 和 `z`​ 分别取 `pos.getX() + 0.5`​、`pos.getY() + 0.5`​ 和 `pos.getZ() + 0.5`​ 的值。
* ​**​`playLocalSound(double x, double y, double z, SoundEvent soundEvent, SoundSource soundSource, float volume, float pitch, boolean distanceDelay)`​** ​

  * **客户端行为**：在给定位置向玩家播放声音。不向服务器发送任何内容。如果 `distanceDelay`​ 为 `true`​，则根据与玩家的距离延迟声音。
  * **服务器行为**：无操作。
  * **用法**：从服务器发送的自定义数据包中调用。原版用于雷声。

#### ClientLevel

* ​**​`playLocalSound(BlockPos pos, SoundEvent soundEvent, SoundSource soundSource, float volume, float pitch, boolean distanceDelay)`​** ​

  * 转发到 `Level#playLocalSound`​，`x`​、`y`​ 和 `z`​ 分别取 `pos.getX() + 0.5`​、`pos.getY() + 0.5`​ 和 `pos.getZ() + 0.5`​ 的值。

#### Entity

* ​**​`playSound(SoundEvent soundEvent, float volume, float pitch)`​** ​

  * 转发到 `Level#playSound`​，`player`​ 为 `null`​，`SoundSource`​ 为 `Entity#getSoundSource`​，`x`​/`y`​/`z`​ 为实体的位置，其他参数传入。

#### Player

* ​**​`playSound(SoundEvent soundEvent, float volume, float pitch)`​** ​（覆盖 `Entity`​ 中的方法）

  * 转发到 `Level#playSound`​，`player`​ 为 `this`​，`SoundSource`​ 为 `SoundSource.PLAYER`​，`x`​/`y`​/`z`​ 为玩家的位置，其他参数传入。因此，客户端/服务器行为模仿 `Level#playSound`​：

    * **客户端行为**：在给定位置向客户端玩家播放声音事件。
    * **服务器行为**：在给定位置向除调用此方法的玩家之外的所有人播放声音事件。

---

### 数据生成（Datagen）

声音文件本身当然不能生成数据，但 `sounds.json`​ 文件可以。为此，我们扩展 `SoundDefinitionsProvider`​ 并覆盖 `registerSounds()`​ 方法：

```java
public class MySoundDefinitionsProvider extends SoundDefinitionsProvider {
    // 参数可以从 GatherDataEvent 获取。
    public MySoundDefinitionsProvider(PackOutput output, ExistingFileHelper existingFileHelper) {
        // 使用你的实际模组 ID 而不是 "examplemod"。
        super(output, "examplemod", existingFileHelper);
    }

    @Override
    public void registerSounds() {
        // 接受 Supplier<SoundEvent>、SoundEvent 或 ResourceLocation 作为第一个参数。
        add(MySoundsClass.MY_SOUND, SoundDefinition.definition()
            // 向声音定义添加声音对象。参数是可变参数。
            .with(
                // 接受字符串或 ResourceLocation 作为第一个参数。
                // 第二个参数可以是 SOUND 或 EVENT，如果是前者，则可以省略。
                sound("examplemod:sound_1", SoundDefinition.SoundType.SOUND)
                    // 设置音量。也有 double 版本。
                    .volume(0.8f)
                    // 设置音高。也有 double 版本。
                    .pitch(1.2f)
                    // 设置权重。
                    .weight(2)
                    // 设置衰减距离。
                    .attenuationDistance(8)
                    // 启用流式传输。
                    // 也有无参数重载，默认为 stream(true)。
                    .stream(true)
                    // 启用预加载。
                    // 也有无参数重载，默认为 preload(true)。
                    .preload(true),
                // 最短的写法。
                sound("examplemod:sound_2")
            )
            // 设置字幕。
            .subtitle("sound.examplemod.sound_1")
            // 启用替换。
            .replace(true)
        );
    }
}
```

与所有数据提供者一样，别忘了将提供者注册到事件中：

```java
@SubscribeEvent
public static void gatherData(GatherDataEvent event) {
    DataGenerator generator = event.getGenerator();
    PackOutput output = generator.getPackOutput();
    ExistingFileHelper existingFileHelper = event.getExistingFileHelper();

    // 其他提供者
    generator.addProvider(
        event.includeClient(),
        new MySoundDefinitionsProvider(output, existingFileHelper)
    );
}
```
