# 模型数据生成（Model Datagen）

### 模型数据生成（Model Datagen）

与大多数 JSON 数据一样，方块和物品模型可以通过数据生成器生成。由于物品和方块模型之间有一些共同点，因此它们的生成代码也有一些共同之处。

---

### 模型数据生成类

#### ModelBuilder

每个模型都从某种 `ModelBuilder`​ 开始——通常是 `BlockModelBuilder`​ 或 `ItemModelBuilder`​，具体取决于你要生成的内容。它包含模型的所有属性：父模型、纹理、元素、变换、加载器等。每个属性都可以通过方法设置：

|方法|作用|
| ------| -----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|​`#texture(String key, ResourceLocation texture)`​|添加一个具有给定键和纹理位置的纹理变量。有一个重载，第二个参数是 `String`​。|
|​`#renderType(ResourceLocation renderType)`​|设置渲染类型。有一个重载，参数是 `String`​。有效值列表请参见 `RenderType`​ 类。|
|​`#ao(boolean ao)`​|设置是否使用环境光遮蔽。|
|​`#guiLight(GuiLight light)`​|设置 GUI 光照。可以是 `GuiLight.FRONT`​ 或 `GuiLight.SIDE`​。|
|​`#element()`​|添加一个新的 `ElementBuilder`​（相当于向模型添加一个新元素）。返回该 `ElementBuilder`​ 以进行进一步修改。|
|​`#transforms()`​|返回构建器的 `TransformVecBuilder`​，用于设置模型的显示。|
|​`#customLoader(BiFunction customLoaderFactory)`​|使用给定的工厂，使此模型使用自定义加载器，从而使用自定义加载器构建器。这会改变构建器类型，因此可能会使用不同的方法，具体取决于加载器的实现。NeoForge 提供了一些开箱即用的自定义加载器，参见链接文章了解更多信息（包括数据生成）。|

**提示**
虽然可以通过数据生成器创建复杂和精细的模型，但建议使用建模软件（如 Blockbench）创建更复杂的模型，然后直接使用导出的模型或将其作为其他模型的父模型。

---

### ModelProvider

方块和物品模型的数据生成都使用 `ModelProvider`​ 的子类，分别命名为 `BlockModelProvider`​ 和 `ItemModelProvider`​。虽然物品模型数据生成直接扩展 `ItemModelProvider`​，但方块模型数据生成使用 `BlockStateProvider`​ 基类，它有一个内部的 `BlockModelProvider`​，可以通过 `BlockStateProvider#models()`​ 访问。此外，`BlockStateProvider`​ 还有一个内部的 `ItemModelProvider`​，可以通过 `BlockStateProvider#itemModels()`​ 访问。`ModelProvider`​ 最重要的部分是 `getBuilder(String path)`​ 方法，它返回给定位置的 `BlockModelBuilder`​（或 `ItemModelBuilder`​）。

然而，`ModelProvider`​ 还包含各种辅助方法。最重要的辅助方法可能是 `withExistingParent(String name, ResourceLocation parent)`​，它返回一个新的构建器（通过 `getBuilder(name)`​），并将给定的 `ResourceLocation`​ 设置为模型父模型。另外两个非常常见的辅助方法是 `mcLoc(String name)`​，它返回一个命名空间为 `minecraft`​ 且路径为给定名称的 `ResourceLocation`​，以及 `modLoc(String name)`​，它执行相同的操作，但使用提供者的模组 ID（通常是你的模组 ID）而不是 `minecraft`​。此外，它还提供了各种辅助方法，这些方法是常见事物的 `#withExistingParent`​ 快捷方式，例如台阶、楼梯、栅栏、门等。

---

### ModelFile

最后一个重要的类是 `ModelFile`​。`ModelFile`​ 是磁盘上模型 JSON 的代码表示。`ModelFile`​ 是一个抽象类，有两个内部子类 `ExistingModelFile`​ 和 `UncheckedModelFile`​。`ExistingModelFile`​ 的存在性通过 `ExistingFileHelper`​ 验证，而 `UncheckedModelFile`​ 则假定存在而不进行进一步检查。此外，`ModelBuilder`​ 也被视为 `ModelFile`​。

---

### 方块模型数据生成

现在，为了实际生成方块状态和方块模型文件，扩展 `BlockStateProvider`​ 并覆盖 `registerStatesAndModels()`​ 方法。请注意，方块模型将始终放置在 `models/block`​ 子文件夹中，但引用是相对于 `models`​ 的（即它们必须始终以 `block/`​ 为前缀）。在大多数情况下，从许多预定义的辅助方法中选择一个是有意义的：

```java
public class MyBlockStateProvider extends BlockStateProvider {
    // 参数值由 GatherDataEvent 提供。
    public MyBlockStateProvider(PackOutput output, ExistingFileHelper existingFileHelper) {
        // 将 "examplemod" 替换为你自己的模组 ID。
        super(output, "examplemod", existingFileHelper);
    }
    
    @Override
    protected void registerStatesAndModels() {
        // 占位符，它们的用法应替换为实际值。有关如何使用模型构建器的信息，请参见上文，
        // 有关模型构建器提供的辅助方法的信息，请参见下文。
        ModelFile exampleModel = models().withExistingParent("example_model", this.mcLoc("block/cobblestone"));
        Block block = MyBlocksClass.EXAMPLE_BLOCK.get();
        ResourceLocation exampleTexture = modLoc("block/example_texture");
        ResourceLocation bottomTexture = modLoc("block/example_texture_bottom");
        ResourceLocation topTexture = modLoc("block/example_texture_top");
        ResourceLocation sideTexture = modLoc("block/example_texture_front");
        ResourceLocation frontTexture = modLoc("block/example_texture_front");

        // 创建一个简单的方块模型，每个面使用相同的纹理。
        // 纹理必须位于 assets/<namespace>/textures/block/<path>.png，其中
        // <namespace> 和 <path> 分别是方块注册名称的命名空间和路径。
        // 大多数（完整）方块使用此方法，例如木板、圆石或砖块。
        simpleBlock(block);
        // 接受模型文件的重载。
        simpleBlock(block, exampleModel);
        // 接受一个或多个（可变参数）ConfiguredModel 对象的重载。
        // 有关 ConfiguredModel 的更多信息，请参见下文。
        simpleBlock(block, ConfiguredModel.builder().build());
        // 添加一个物品模型文件，使用方块的名称，并将给定的模型文件作为父模型，以便方块物品拾取。
        simpleBlockItem(block, exampleModel);
        // 调用 #simpleBlock()（模型文件重载）和 #simpleBlockItem 的简写。
        simpleBlockWithItem(block, exampleModel);
        
        // 添加一个原木方块模型。需要在 assets/<namespace>/textures/block/<path>.png 和
        // assets/<namespace>/textures/block/<path>_top.png 处有两个纹理，分别引用侧面和顶部纹理。
        // 请注意，此处的方块输入仅限于 RotatedPillarBlock，这是原版原木使用的类。
        logBlock(block);
        // 类似于 #logBlock，但纹理命名为 <path>_side.png 和 <path>_end.png，而不是
        // <path>.png 和 <path>_top.png。用于石英柱和类似方块。
        // 有一个重载允许你指定不同的纹理基础名称，然后根据需要添加 _side 和 _end 后缀，
        // 一个重载允许你指定侧面和顶部纹理的两个资源位置，
        // 以及一个重载允许指定侧面和顶部模型文件。
        axisBlock(block);
        // #logBlock 和 #axisBlock 的变体，还允许指定渲染类型。
        // 有字符串和资源位置变体用于渲染类型，
        // 与 #logBlock 和 #axisBlock 的所有变体组合。
        logBlockWithRenderType(block, "minecraft:cutout");
        axisBlockWithRenderType(block, mcLoc("cutout_mipped"));
        
        // 指定一个可水平旋转的方块模型，具有侧面纹理、正面纹理和顶部纹理。
        // 底部也将使用侧面纹理。如果你不需要正面或顶部纹理，
        // 只需传递侧面纹理两次。例如，熔炉和类似方块使用此方法。
        horizontalBlock(block, sideTexture, frontTexture, topTexture);
        // 指定一个可水平旋转的方块模型，使用一个模型文件，该文件将根据需要旋转。
        // 有一个重载，接受一个 Function<BlockState, ModelFile> 而不是模型文件，
        // 允许不同的旋转使用不同的模型。例如，切石机使用此方法。
        horizontalBlock(block, exampleModel);
        // 指定一个可水平旋转的方块模型，该模型附加到一个面上，例如按钮或拉杆。
        // 考虑将方块放置在地面和天花板上，并相应地旋转它们。
        // 类似于 #horizontalBlock，有一个重载接受 Function<BlockState, ModelFile>。
        horizontalFaceBlock(block, exampleModel);
        // 类似于 #horizontalBlock，但适用于可朝所有方向旋转的方块，包括上下。
        // 同样，有一个重载接受 Function<BlockState, ModelFile>。
        directionalBlock(block, exampleModel);
    }
}
```

此外，`BlockStateProvider`​ 中还存在以下常见方块模型的辅助方法：

* 楼梯
* 台阶
* 按钮
* 压力板
* 标志
* 栅栏
* 栅栏门
* 墙
* 玻璃板
* 门
* 活板门

在某些情况下，方块状态不需要特殊处理，但模型需要。为此，可以通过 `BlockStateProvider#models()`​ 访问的 `BlockModelProvider`​ 提供了一些额外的辅助方法，所有这些方法都接受名称作为第一个参数，并且大多数方法与完整立方体有关。它们通常用作 `simpleBlock`​ 等方法的模型文件参数。辅助方法包括支持 `BlockStateProvider`​ 中的方法，以及：

* ​`withExistingParent`​：之前已经提到过，此方法返回一个具有给定父模型的新模型构建器。父模型必须已经存在或在模型之前创建。
* ​`getExistingFile`​：在模型提供者的 `ExistingFileHelper`​ 中执行查找，如果存在则返回相应的 `ModelFile`​，否则抛出 `IllegalStateException`​。
* ​`singleTexture`​：接受一个父模型和一个纹理位置，返回一个具有给定父模型并将纹理变量 `texture`​ 设置为给定纹理位置的模型。
* ​`sideBottomTop`​：接受一个父模型和三个纹理位置，返回一个具有给定父模型并将侧面、底部和顶部纹理设置为三个纹理位置的模型。
* ​`cube`​：接受六个纹理资源位置用于六个面，返回一个完整立方体模型，六个面设置为六个纹理。
* ​`cubeAll`​：接受一个纹理位置，返回一个完整立方体模型，将给定纹理应用于所有六个面。可以看作是 `singleTexture`​ 和 `cube`​ 的结合。
* ​`cubeTop`​：接受两个纹理位置，返回一个完整立方体模型，第一个纹理应用于侧面和底部，第二个纹理应用于顶部。
* ​`cubeBottomTop`​：接受三个纹理位置，返回一个完整立方体模型，将侧面、底部和顶部纹理设置为三个纹理位置。可以看作是 `cube`​ 和 `sideBottomTop`​ 的结合。
* ​`cubeColumn`​ 和 `cubeColumnHorizontal`​：接受两个纹理位置，返回一个“直立”或“横放”的柱状立方体模型，将侧面和顶部纹理设置为两个纹理位置。由 `BlockStateProvider#logBlock`​、`BlockStateProvider#axisBlock`​ 及其变体使用。
* ​`orientable`​：接受三个纹理位置，返回一个具有“正面”纹理的立方体。三个纹理位置分别是侧面、正面和顶部纹理。
* ​`orientableVertical`​：`orientable`​ 的变体，省略顶部参数，而是使用侧面参数。
* ​`orientableWithBottom`​：`orientable`​ 的变体，在正面和顶部参数之间有一个底部纹理的第四个参数。
* ​`crop`​：接受一个纹理位置，返回一个类似作物的模型，使用给定的纹理，如四种原版作物使用。
* ​`cross`​：接受一个纹理位置，返回一个十字模型，使用给定的纹理，如花、树苗和许多其他植物使用。
* ​`torch`​：接受一个纹理位置，返回一个火把模型，使用给定的纹理。
* ​`wall_torch`​：接受一个纹理位置，返回一个墙上火把模型，使用给定的纹理（墙上火把是与站立火把分开的方块）。
* ​`carpet`​：接受一个纹理位置，返回一个地毯模型，使用给定的纹理。

最后，别忘了将你的方块状态提供者注册到事件中：

```java
@SubscribeEvent
public static void gatherData(GatherDataEvent event) {
    DataGenerator generator = event.getGenerator();
    PackOutput output = generator.getPackOutput();
    ExistingFileHelper existingFileHelper = event.getExistingFileHelper();

    // 其他提供者在这里
    generator.addProvider(
        event.includeClient(),
        new MyBlockStateProvider(output, existingFileHelper)
    );
}
```

---

### ConfiguredModel.Builder

如果默认的辅助方法不能满足你的需求，你还可以直接使用 `ConfiguredModel.Builder`​ 构建模型对象，然后在 `VariantBlockStateBuilder`​ 中使用它们来构建变体方块状态文件，或在 `MultiPartBlockStateBuilder`​ 中使用它们来构建多部分方块状态文件：

```java
// 创建一个 ConfiguredModel.Builder。或者，你可以在适用的情况下使用下面演示的方法之一
// （VariantBlockStateBuilder.PartialBlockstate#modelForState 或 MultiPartBlockStateBuilder#part）。
ConfiguredModel.Builder<?> builder = ConfiguredModel.builder()
// 使用一个模型文件。如前所述，可以是 ExistingModelFile、UncheckedModelFile，
// 或某种 ModelBuilder。有关如何使用 ModelBuilder 的信息，请参见上文。
        .modelFile(models().withExistingParent("example_model", this.mcLoc("block/cobblestone")))
        // 设置绕 x 和 y 轴的旋转。
        .rotationX(90)
        .rotationY(180)
        // 设置 uvlock。
        .uvlock(true)
        // 设置权重。
        .weight(5);
// 构建配置的模型。返回类型是一个数组
// 以考虑同一方块状态中的多个可能模型。
ConfiguredModel[] model = builder.build();

// 获取一个变体方块状态构建器。
VariantBlockStateBuilder variantBuilder = getVariantBuilder(MyBlocksClass.EXAMPLE_BLOCK.get());
// 创建一个部分状态并设置其属性。
VariantBlockStateBuilder.PartialBlockstate partialState = variantBuilder.partialState();
// 为部分方块状态添加一个或多个模型。模型是一个可变参数。
variantBuilder.addModels(partialState,
    // 指定至少一个 ConfiguredModel.Builder，如上所示。通过 #modelForState() 创建。
    partialState.modelForState()
        .modelFile(models().withExistingParent("example_variant_model", this.mcLoc("block/cobblestone")))
        .uvlock(true)
);
// 或者，forAllStates(Function<BlockState, ConfiguredModel[]>) 为每个状态创建一个模型。
// 传递的函数将为每个可能的状态调用一次。
variantBuilder.forAllStates(state -> {
    // 根据状态的属性返回一个 ConfiguredModel。
    // 例如，以下代码将根据方块的水平旋转旋转模型。
    return ConfiguredModel.builder()
        .modelFile(models().withExistingParent("example_variant_model", this.mcLoc("block/cobblestone")))
        .rotationY((int) state.getValue(BlockStateProperties.HORIZONTAL_FACING).toYRot())
        .build();
});

// 获取一个多部分方块状态构建器。
MultiPartBlockStateBuilder multipartBuilder = getMultipartBuilder(MyBlocksClass.EXAMPLE_BLOCK.get());
// 添加一个新部分。以 .part() 开始，以 .end() 结束。
multipartBuilder.part()
    // 第一步：构建模型。multipartBuilder.part() 返回一个 ConfiguredModel.Builder，
    // 意味着上面看到的所有方法都可以在这里使用。
    .modelFile(models().withExistingParent("example_multipart_model", this.mcLoc("block/cobblestone")))
    // 调用 .addModel()。现在模型已构建，我们可以进行第二步：添加部分数据。
    .addModel()
    // 为部分添加条件。需要一个属性
    // 和至少一个属性值；属性值是一个可变参数。
    .condition(BlockStateProperties.FACING, Direction.NORTH, Direction.SOUTH)
    // 将多部分条件设置为 OR 而不是默认的 AND。
    .useOr()
    // 创建一个嵌套条件组。
    .nestedGroup()
    // 向嵌套组添加条件。
    .condition(BlockStateProperties.FACING, Direction.NORTH)
    // 仅将此条件组设置为 OR 而不是 AND。
    .useOr()
    // 创建另一个嵌套条件组。嵌套组的数量没有限制。
    .nestedGroup()
    // 结束嵌套条件组，返回到拥有部分构建器或条件组级别。
    // 这里调用两次，因为我们目前有两个嵌套组。
    .endNestedGroup()
    .endNestedGroup()
    // 结束部分构建器并将结果部分添加到多部分构建器中。
    .end();
```

---

### 物品模型数据生成

生成物品模型要简单得多，这主要是因为我们直接操作 `ItemModelProvider`​，而不是使用像 `BlockStateProvider`​ 这样的中间类，这当然是因为物品模型没有类似于方块状态文件的等效物，而是直接使用。

与上面类似，我们创建一个类并让它扩展基础提供者，在这种情况下是 `ItemModelProvider`​。由于我们直接在 `ModelProvider`​ 的子类中，所有 `models()`​ 调用都变成了 `this`​（或被省略）。

```java
public class MyItemModelProvider extends ItemModelProvider {
    public MyItemModelProvider(PackOutput output, ExistingFileHelper existingFileHelper) {
        super(output, "examplemod", existingFileHelper);
    }
    
    @Override
    protected void registerModels() {
        // 方块物品通常使用其对应的方块模型作为父模型。
        withExistingParent(MyItemsClass.EXAMPLE_BLOCK_ITEM.getId().toString(), modLoc("block/example_block"));
        // 物品通常使用一个简单的父模型和一个纹理。最常见的父模型是 item/generated 和 item/handheld。
        // 在此示例中，物品纹理将位于 assets/examplemod/textures/item/example_item.png。
        // 如果你想要更复杂的模型，可以使用 getBuilder()，然后像处理方块模型一样处理。
        withExistingParent(MyItemsClass.EXAMPLE_ITEM.getId().toString(), mcLoc("item/generated")).texture("layer0", "item/example_item");
        // 上面一行非常常见，因此有一个快捷方式。请注意，物品注册名称和
        // 纹理路径（相对于 textures/item）必须匹配。
        basicItem(MyItemsClass.EXAMPLE_ITEM.get());
    }
}
```

与所有数据提供者一样，别忘了将你的提供者注册到事件中：

```java
@SubscribeEvent
public static void gatherData(GatherDataEvent event) {
    DataGenerator generator = event.getGenerator();
    PackOutput output = generator.getPackOutput();
    ExistingFileHelper existingFileHelper = event.getExistingFileHelper();

    // 其他提供者在这里
    generator.addProvider(
        event.includeClient(),
        new MyItemModelProvider(output, existingFileHelper)
    );
}
```
