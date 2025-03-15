# 资源（Resources）

### 资源（Resources）

资源是游戏使用的外部文件，但不是代码。最突出的资源类型是纹理，然而，Minecraft 生态系统中还存在许多其他类型的资源。当然，所有这些资源都需要在代码端有一个消费者，因此消费系统也被归类在本节中。

Minecraft 通常有两种资源：用于逻辑客户端的资源，称为**资产（Assets）** ，以及用于逻辑服务器的资源，称为**数据（Data）** 。资产主要是显示信息，例如纹理、显示模型、翻译或声音，而数据包括影响游戏玩法的各种内容，例如战利品表、配方或世界生成信息。它们分别从资源包和数据包中加载。NeoForge 为每个模组生成了一个内置的资源包和数据包。

资源包和数据包通常需要一个`pack.mcmeta`​文件；然而，现代 NeoForge 在运行时为你生成这些文件，因此你不需要担心它。

如果你对某些内容的格式感到困惑，可以查看原版资源。你的 NeoForge 开发环境不仅包含原版代码，还包含原版资源。它们可以在 External Resources 部分（IntelliJ）/Project Libraries 部分（Eclipse）中找到，名称为`ng_dummy_ng.net.minecraft:client:client-extra:<minecraft_version>`​（用于 Minecraft 资源）或`ng_dummy_ng.net.neoforged:neoforge:<neoforge_version>`​（用于 NeoForge 资源）。

---

### 资产（Assets）

资产，或客户端资源，是仅在客户端相关的所有资源。它们从资源包中加载，有时也称为纹理包（源自旧版本，当时它们只能影响纹理）。资源包基本上是一个`assets`​文件夹。`assets`​文件夹包含资源包中包含的各种命名空间的子文件夹；每个命名空间都是一个子文件夹。例如，一个名为`coolmod`​的模组的资源包可能包含一个`coolmod`​命名空间，但也可能包含其他命名空间，例如`minecraft`​。

NeoForge 自动将所有模组资源包收集到**模组资源包**中，该包位于资源包菜单的**已选包**侧底部。目前无法禁用模组资源包。然而，位于模组资源包上方的资源包会覆盖下方资源包中定义的资源。这种机制允许资源包制作者覆盖你的模组资源，也允许模组开发者根据需要覆盖 Minecraft 资源。

资源包可以包含模型、方块状态文件、纹理、声音、粒子定义和翻译文件。

---

### 数据（Data）

与资产相比，数据是所有服务器资源的术语。与资源包类似，数据通过数据包（或数据包）加载。与资源包一样，数据包由一个`pack.mcmeta`​文件和一个名为`data`​的根文件夹组成。然后，再次与资源包一样，`data`​文件夹包含资源包中包含的各种命名空间的子文件夹；每个命名空间都是一个子文件夹。例如，一个名为`coolmod`​的模组的数据包可能包含一个`coolmod`​命名空间，但也可能包含其他命名空间，例如`minecraft`​。

NeoForge 在创建新世界时自动应用所有模组数据包。目前无法禁用模组数据包。然而，大多数数据文件可以通过优先级更高的数据包覆盖（从而通过替换为空文件来移除）。可以通过将其他数据包放置在世界的`datapacks`​子文件夹中，然后通过`/datapack`​命令启用或禁用它们来启用或禁用其他数据包。

**注意**：目前没有内置的方法将一组自定义数据包应用于每个世界。然而，有许多模组可以实现这一点。

数据包可能包含以下文件夹及其文件：

|文件夹名称|内容|
| ----------------------| ------------------|
|​`advancement`​|进度|
|​`damage_type`​|伤害类型|
|​`loot_table`​|战利品表|
|​`recipe`​|配方|
|​`tags`​|标签|
|​`neoforge/data_maps`​|数据映射|
|​`neoforge/loot_modifiers`​|全局战利品修改器|
|​`dimension`​, `dimension_type`​, `structure`​, `worldgen`​, `neoforge/biome_modifier`​|世界生成文件|

此外，它们还可能包含一些与命令集成的系统的子文件夹。这些系统很少与模组结合使用，但仍然值得一提：

|文件夹名称|内容|
| ------------| ------------|
|​`chat_type`​|聊天类型|
|​`function`​|函数|
|​`item_modifier`​|物品修改器|
|​`predicate`​|谓词|

---

### pack.mcmeta

​`pack.mcmeta`​文件保存资源包或数据包的元数据。对于模组，NeoForge 使此文件过时，因为`pack.mcmeta`​是合成的。如果你仍然需要`pack.mcmeta`​文件，完整的规范可以在链接的 Minecraft Wiki 文章中找到。

---

### 数据生成（Data Generation）

数据生成，俗称**datagen**，是一种以编程方式生成 JSON 资源文件的方法，以避免手动编写这些文件的繁琐且容易出错的过程。这个名字有点误导，因为它既适用于资产，也适用于数据。

Datagen 通过**Data**运行配置运行，该配置与**Client**和**Server**运行配置一起为你生成。数据运行配置遵循模组生命周期，直到注册事件触发后。然后它触发`GatherDataEvent`​，在该事件中你可以以数据提供者的形式注册要生成的对象，将所述对象写入磁盘，并结束该过程。

所有数据提供者都扩展了`DataProvider`​接口，并且通常需要重写一个方法。以下是 Minecraft 和 NeoForge 提供的一些值得注意的数据生成器（链接的文章提供了更多信息，例如辅助方法）：

|类|方法|生成内容|端|备注|
| -----------------| ------| ----------------------------------------| --------| --------------------------------------------------------------------|
|​`BlockStateProvider`​|​`registerStatesAndModels()`​|方块状态文件，方块模型|客户端||
|​`ItemModelProvider`​|​`registerModels()`​|物品模型|客户端||
|​`LanguageProvider`​|​`addTranslations()`​|翻译|客户端|还需要在构造函数中传递语言。|
|​`ParticleDescriptionProvider`​|​`addDescriptions()`​|粒子定义|客户端||
|​`SoundDefinitionsProvider`​|​`registerSounds()`​|声音定义|客户端||
|​`SpriteSourceProvider`​|​`gather()`​|精灵源 / 图集|客户端||
|​`AdvancementProvider`​|​`generate()`​|进度|服务器|确保使用 NeoForge 变体，而不是 Minecraft 的。|
|​`LootTableProvider`​|​`generate()`​|战利品表|服务器|需要额外的方法和类才能正常工作，详见链接文章。|
|​`RecipeProvider`​|​`buildRecipes(RecipeOutput)`​|配方|服务器||
|​`TagsProvider`​ 的各种子类|​`addTags(HolderLookup.Provider)`​|标签|服务器|存在多个专门的子类，详见链接文章。|
|​`DataMapProvider`​|​`gather()`​|数据映射条目|服务器||
|​`GlobalLootModifierProvider`​|​`start()`​|全局战利品修改器|服务器||
|​`DatapackBuiltinEntriesProvider`​|N/A|数据包内置条目，例如世界生成和伤害类型|服务器|无需重写方法，而是在构造函数中的 lambda 中添加条目。详见链接文章。|
|​`JsonCodecProvider`​（抽象类）|​`gather()`​|具有编解码器的对象|两者|可以扩展以用于任何具有编解码器以编码数据的对象。|

所有这些提供者都遵循相同的模式。首先，你创建一个子类并添加你自己的资源以生成。然后，你将提供者添加到事件处理程序中的事件中。以下是使用`RecipeProvider`​的示例：

```java
public class MyRecipeProvider extends RecipeProvider {
    public MyRecipeProvider(PackOutput output, CompletableFuture<HolderLookup.Provider> lookupProvider) {
        super(output, lookupProvider);
    }

    @Override
    protected void buildRecipes(RecipeOutput output) {
        // 在这里注册你的配方。
    }
}

@EventBusSubscriber(bus = EventBusSubscriber.Bus.MOD, modid = "examplemod")
public class MyDatagenHandler {
    @SubscribeEvent
    public static void gatherData(GatherDataEvent event) {
        // 数据生成器可能需要其中一些作为构造函数参数。
        // 有关每个参数的更多详细信息，请参阅下文。
        DataGenerator generator = event.getGenerator();
        PackOutput output = generator.getPackOutput();
        ExistingFileHelper existingFileHelper = event.getExistingFileHelper();
        CompletableFuture<HolderLookup.Provider> lookupProvider = event.getLookupProvider();

        // 注册提供者。
        generator.addProvider(
                // 一个布尔值，确定是否应实际生成数据。
                // 事件提供了确定这一点的方法：
                // event.includeClient(), event.includeServer(),
                // event.includeDev() 和 event.includeReports()。
                // 由于配方是服务器数据，我们仅在服务器数据生成中运行它们。
                event.includeServer(),
                // 我们的提供者。
                new MyRecipeProvider(output, lookupProvider)
        );
        // 其他数据提供者在这里。
    }
}
```

事件提供了一些上下文供你使用：

* ​`event.getGenerator()`​返回你注册提供者的`DataGenerator`​。
* ​`event.getPackOutput()`​返回一个`PackOutput`​，某些提供者使用它来确定其文件输出位置。
* ​`event.getExistingFileHelper()`​返回一个`ExistingFileHelper`​，提供者使用它来处理可以引用其他文件的内容（例如方块模型，可以指定父文件）。
* ​`event.getLookupProvider()`​返回一个`CompletableFuture<HolderLookup.Provider>`​，主要用于标签和数据生成注册表以引用其他可能尚未存在的元素。
* ​`event.includeClient()`​、`event.includeServer()`​、`event.includeDev()`​和`event.includeReports()`​是布尔方法，允许你检查是否启用了特定的命令行参数（见下文）。

---

### 命令行参数

数据生成器可以接受几个命令行参数：

* ​`--mod examplemod`​：告诉数据生成器为此模组运行数据生成。NeoGradle 自动为所属模组 ID 添加此参数，如果你在一个项目中有多个模组，请添加此参数。
* ​`--output path/to/folder`​：告诉数据生成器输出到给定文件夹。建议使用 Gradle 的`file(...).getAbsolutePath()`​为你生成绝对路径（相对于项目根目录的路径）。默认为`file('src/generated/resources').getAbsolutePath()`​。
* ​`--existing path/to/folder`​：告诉数据生成器在检查现有文件时考虑给定文件夹。与输出一样，建议使用 Gradle 的`file(...).getAbsolutePath()`​。
* ​`--existing-mod examplemod`​：告诉数据生成器在检查现有文件时考虑给定模组的 JAR 文件中的资源。
* 生成器模式（所有这些参数都是布尔参数，不需要任何额外的参数）：

  * ​`--includeClient`​：是否生成客户端资源（资产）。在运行时使用`GatherDataEvent#includeClient()`​检查。
  * ​`--includeServer`​：是否生成服务器资源（数据）。在运行时使用`GatherDataEvent#includeServer()`​检查。
  * ​`--includeDev`​：是否运行开发工具。通常不应由模组使用。在运行时使用`GatherDataEvent#includeDev()`​检查。
  * ​`--includeReports`​：是否转储已注册对象的列表。在运行时使用`GatherDataEvent#includeReports()`​检查。
  * ​`--all`​：启用所有生成器模式。

所有参数都可以通过将以下内容添加到你的`build.gradle`​中来添加到运行配置中：

```groovy
runs {
    // 其他运行配置在这里

    data {
        programArguments.addAll '--arg1', 'value1', '--arg2', 'value2', '--all' // 布尔参数没有值
    }
}
```

例如，要复制默认参数，你可以指定以下内容：

```groovy
runs {
    // 其他运行配置在这里

    data {
        programArguments.addAll '--mod', 'examplemod', // 插入你自己的模组 ID
                '--output', file('src/generated/resources').getAbsolutePath(),
                '--includeClient',
                '--includeServer'
    }
}
```
