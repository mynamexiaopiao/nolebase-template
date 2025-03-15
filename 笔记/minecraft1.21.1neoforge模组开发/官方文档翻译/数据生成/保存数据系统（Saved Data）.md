# 保存数据系统（Saved Data）

### 保存数据系统（Saved Data）

保存数据系统（Saved Data，简称 SD）可用于在游戏世界中保存额外的数据。

如果数据与某些方块实体（Block Entities）、区块（Chunks）或实体（Entities）相关，请考虑使用**数据附加系统（Data Attachments）** 。

---

#### 声明

每个 SD 实现都必须继承 `SavedData`​ 类。有两个重要的方法需要注意：

1. ​**​`save`​**​：允许实现将 NBT 数据写入世界。
2. ​**​`setDirty`​**​：在更改数据后必须调用的方法，用于通知游戏有需要保存的更改。如果不调用此方法，`#save`​ 将不会被调用，原始数据将保持不变。

---

#### 附加到世界

任何 `SavedData`​ 都是动态加载和/或附加到世界的。因此，如果从未在世界中创建过 SD，则它不会存在。

​`SavedData`​ 是从 `DimensionDataStorage`​ 创建和加载的，可以通过调用 `ServerChunkCache#getDataStorage`​ 或 `ServerLevel#getDataStorage`​ 来访问它。然后，你可以通过调用 `DimensionDataStorage#computeIfAbsent`​ 来获取或创建 SD 的实例。如果 SD 存在，则尝试获取当前实例；如果不存在，则创建一个新实例并加载所有可用数据。

​`DimensionDataStorage#computeIfAbsent`​ 接受两个参数：

1. 一个 `SavedData.Factory`​ 实例，它包含一个用于构造新 SD 实例的提供器（Supplier）和一个用于将 NBT 数据加载到 SD 并返回它的函数。
2. 存储在数据文件夹中用于实现世界的 `.dat`​ 文件的名称。名称必须是有效的文件名，且不能包含 `/`​ 或 `\`​。

例如，如果在下界（Nether）中创建了一个名为 `"example"`​ 的 SD，则会在 `./<level_folder>/DIM-1/data/example.dat`​ 路径下创建一个文件，并可以按如下方式实现：

```java
// 在某个保存数据的实现中
public class ExampleSavedData extends SavedData {

    // 创建新的保存数据实例
    public static ExampleSavedData create() {
        return new ExampleSavedData();
    }

    // 加载现有的保存数据实例
    public static ExampleSavedData load(CompoundTag tag, HolderLookup.Provider lookupProvider) {
        ExampleSavedData data = ExampleSavedData.create();
        // 加载保存的数据
        return data;
    }

    @Override
    public CompoundTag save(CompoundTag tag, HolderLookup.Provider registries) {
        // 将数据写入 tag
        return tag;
    }

    public void foo() {
        // 更改保存数据中的数据
        // 如果数据更改，则调用 setDirty
        this.setDirty();
    }
}

// 在类的某个方法中
netherDataStorage.computeIfAbsent(new Factory<>(ExampleSavedData::create, ExampleSavedData::load), "example");
```

---

#### 附加到主世界（Overworld）

如果 SD 不特定于某个世界，则应将其附加到主世界（Overworld），可以通过 `MinecraftServer#overworld`​ 获取主世界。主世界是唯一一个永远不会完全卸载的维度，因此非常适合存储多世界数据。

例如：

```java
// 在主世界中附加保存数据
ServerLevel overworld = minecraftServer.overworld();
overworld.getDataStorage().computeIfAbsent(new Factory<>(ExampleSavedData::create, ExampleSavedData::load), "example");
```
