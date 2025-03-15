# 调试性能分析器（Debug Profiler）

### 调试性能分析器（Debug Profiler）

Minecraft 提供了一个调试性能分析器（Debug Profiler），用于提供系统数据、当前游戏设置、JVM 数据、世界数据以及客户端/服务器端的 Tick 信息，以帮助查找耗时代码。对于模组开发者和服务器管理员来说，这在分析 `TickEvents`​ 或 `BlockEntities`​ 等性能瓶颈时非常有用。

---

#### 使用调试性能分析器

调试性能分析器的使用非常简单。只需按下调试快捷键 `F3 + L`​ 即可启动性能分析器。10 秒后，它会自动停止；也可以通过再次按下快捷键提前停止。

**注意**：
自然，你只能分析实际被执行的代码路径。要分析的实体（Entities）和方块实体（BlockEntities）必须存在于世界中，才能在结果中显示。

停止分析器后，它会在运行目录的 `debug/profiling`​ 子目录中创建一个新的 ZIP 文件。文件名格式为 `yyyy-mm-dd_hh_mi_ss-WorldName-VersionNumber.zip`​。

---

#### 读取性能分析结果

在每个端（客户端和服务器）的文件夹中，你会找到一个 `profiling.txt`​ 文件，其中包含结果数据。文件顶部首先告诉你分析器运行了多长时间（以毫秒为单位）以及在此期间运行了多少个 Tick。

在下面，你会看到类似以下片段的信息：

```plaintext
[00] levels - 96.70%/96.70%
[01] |   Level Name - 99.76%/96.47%
[02] |   |   tick - 99.31%/95.81%
[03] |   |   |   entities - 47.72%/45.72%
[04] |   |   |   |   regular - 98.32%/44.95%
[04] |   |   |   |   blockEntities - 0.90%/0.41%
[05] |   |   |   |   |   unspecified - 64.26%/0.26%
[05] |   |   |   |   |   minecraft:furnace - 33.35%/0.14%
[05] |   |   |   |   |   minecraft:chest - 2.39%/0.01%
```

以下是每个部分的解释：

|示例|描述|
| ------| -----------------------------------------------------------------------------------------------------------------------------------------|
|​`[02] tick 99.31% 95.81%`​|- **深度**：部分的层级。<br />- **名称**：部分的名称。<br />- **第一个百分比**：相对于其父部分的时间占比（对于第 0 层，是整个 Tick 的时间占比）。<br />- **第二个百分比**：占整个 Tick 的时间占比。|

---

#### 分析你自己的代码

调试性能分析器对实体（Entity）和方块实体（BlockEntity）有基本支持。如果你需要分析其他内容，可能需要手动创建分析部分，如下所示：

```java
profiler.push("yourSectionName");
// 你想要分析的代码
profiler.pop();
```

你可以从 `Level`​、`MinecraftServer`​ 或 `Minecraft`​ 实例中获取 `ProfilerFiller`​ 实例。现在，你只需在结果文件中搜索你的部分名称即可。

---

#### 示例代码

以下是一个示例，展示如何在模组代码中使用性能分析器：

```java
import net.minecraft.util.profiling.ProfilerFiller;

public class MyBlockEntity extends BlockEntity {
    public MyBlockEntity(BlockPos pos, BlockState state) {
        super(MyMod.BLOCK_ENTITY_TYPE.get(), pos, state);
    }

    @Override
    public void tick() {
        ProfilerFiller profiler = level.getProfiler();
        profiler.push("myModBlockEntityTick");

        // 你的逻辑代码
        doSomeWork();

        profiler.pop();
    }

    private void doSomeWork() {
        // 模拟一些工作
        try {
            Thread.sleep(10); // 模拟耗时操作
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
```

在分析结果中，你会看到类似以下内容：

```plaintext
[03] |   |   |   blockEntities - 0.90%/0.41%
[04] |   |   |   |   myModBlockEntityTick - 50.00%/0.20%
```
