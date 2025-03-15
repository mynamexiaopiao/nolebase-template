# 烘焙模型（Baked Models）

### 烘焙模型（Baked Models）

烘焙模型是带有纹理的形状的代码表示。它们可以来自多个来源，例如通过调用 `UnbakedModel#bake`​（默认模型加载器）或 `IUnbakedGeometry#bake`​（自定义模型加载器）。一些方块实体渲染器也使用烘焙模型。模型的复杂程度没有限制。

模型存储在 `ModelManager`​ 中，可以通过 `Minecraft.getInstance().modelManager`​ 访问。然后，你可以调用 `ModelManager#getModel`​ 通过 `ResourceLocation`​ 或 `ModelResourceLocation`​ 获取某个模型。模组通常会重用之前自动加载和烘焙的模型。

---

### BakedModel 的方法

#### getQuads

烘焙模型中最重要的方法是 `getQuads`​。该方法负责返回一个 `BakedQuad`​ 列表，这些 `BakedQuad`​ 可以被发送到 GPU 进行渲染。一个四边形（quad）类似于建模程序（以及大多数其他游戏）中的三角形，但由于 Minecraft 通常专注于方块，开发者选择使用四边形（4 个顶点）而不是三角形（3 个顶点）进行渲染。`getQuads`​ 有五个参数可以使用：

* ​**​`BlockState`​**​：正在渲染的方块状态。可能为 `null`​，表示正在渲染物品。
* ​**​`Direction`​**​：正在剔除的面的方向。可能为 `null`​，表示应返回无法被遮挡的四边形。
* ​**​`RandomSource`​**​：客户端绑定的随机源，可用于随机化。
* ​**​`ModelData`​**​：要使用的额外模型数据。这可能包含来自方块实体的渲染所需的额外数据。由 `BakedModel#getModelData`​ 提供。
* ​**​`RenderType`​**​：用于渲染方块的渲染类型。可能为 `null`​，表示应返回此模型使用的所有渲染类型的四边形。否则，它是 `BakedModel#getRenderTypes`​ 返回的渲染类型之一（见下文）。

模型应大量缓存。这是因为即使区块仅在其中的方块发生变化时才会重建，此方法中的计算仍然需要尽可能快，并且理想情况下应大量缓存，因为每个区块部分（最多 4096 个方块）会多次调用此方法（每个渲染类型最多调用 7 次 * 模型使用的渲染类型数量 * 每个区块部分的方块数量）。此外，方块实体渲染器或实体渲染器实际上可能会每帧多次调用此方法。

#### applyTransform 和 getTransforms

​`applyTransform`​ 允许在应用透视变换到模型时自定义逻辑，包括返回完全独立的模型。此方法由 NeoForge 添加，用于替代原版的 `getTransforms()`​ 方法，后者仅允许自定义变换本身，而不能自定义应用变换的方式。然而，`applyTransform`​ 的默认实现依赖于 `getTransforms`​，因此如果你只需要自定义变换，也可以覆盖 `getTransforms`​ 并完成工作。`applyTransforms`​ 提供三个参数：

* ​**​`ItemDisplayContext`​**​：模型正在变换到的视角。
* ​**​`PoseStack`​**​：用于渲染的姿势堆栈。
* ​**​`boolean`​**​：是否使用左手渲染的修改值而不是默认的右手渲染值；如果渲染的手是左手（副手，或选项启用了左手模式时的主手），则为 `true`​。

**注意**
`applyTransform`​ 和 `getTransforms`​ 仅适用于物品模型。

#### 其他方法

​`BakedModel`​ 中你可能覆盖或查询的其他方法包括：

|方法签名|作用|
| ----------| ---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|​`TriState useAmbientOcclusion()`​|是否使用环境光遮蔽。接受 `BlockState`​、`RenderType`​ 和 `ModelData`​ 参数，并返回一个 `TriState`​，允许不仅强制禁用 AO，还可以强制启用 AO。有两个重载，每个重载返回一个布尔参数，并接受仅 `BlockState`​ 或无参数；这两个重载已被弃用，建议使用第一个变体。|
|​`boolean isGui3d()`​|此模型在 GUI 槽中是渲染为 3D 还是平面。|
|​`boolean usesBlockLight()`​|在光照模型时是否使用 3D 光照（`true`​）或来自前方的平面光照（`false`​）。|
|​`boolean isCustomRenderer()`​|如果为 `true`​，则跳过正常渲染并调用关联的 `BlockEntityWithoutLevelRenderer`​ 的 `renderByItem`​ 方法。如果为 `false`​，则通过默认渲染器渲染。|
|​`ItemOverrides getOverrides()`​|返回与此模型关联的 `ItemOverrides`​。这仅与物品模型相关。|
|​`ModelData getModelData(BlockAndTintGetter, BlockPos, BlockState, ModelData)`​|返回用于模型的模型数据。此方法传递一个现有的 `ModelData`​，如果方块有关联的方块实体，则它是 `BlockEntity#getModelData()`​ 的结果，否则为 `ModelData.EMPTY`​。此方法可用于需要模型数据但没有方块实体的方块，例如具有连接纹理的方块。|
|​`TextureAtlasSprite getParticleIcon(ModelData)`​|返回用于模型的粒子图标。可以使用模型数据为不同的模型数据值使用不同的粒子图标。NeoForge 添加，替代了无参数的原版 `getParticleIcon()`​ 重载。|
|​`ChunkRenderTypeSet getRenderTypes(BlockState, RandomSource, ModelData)`​|返回一个 `ChunkRenderTypeSet`​，包含用于渲染方块模型的渲染类型。`ChunkRenderTypeSet`​ 是一个基于集合的有序 `Iterable<RenderType>`​。默认情况下回退到从模型 JSON 获取渲染类型。仅用于方块模型，物品模型使用下面的重载。|
|​`List<RenderType> getRenderTypes(ItemStack, boolean)`​|返回一个 `List<RenderType>`​，包含用于渲染物品模型的渲染类型。默认情况下回退到正常的模型绑定渲染类型查找，该查找始终生成一个包含一个元素的列表。仅用于物品模型，方块模型使用上面的重载。|

---

### 视角（Perspectives）

Minecraft 的渲染引擎识别总共 8 种视角类型（如果包括代码中的回退，则为 9 种）用于物品渲染。这些用于模型 JSON 的 `display`​ 块，并在代码中通过 `ItemDisplayContext`​ 枚举表示。

|枚举值|JSON 键|用途|
| --------| ---------| ----------------------------------------------------------------|
|​`THIRD_PERSON_RIGHT_HAND`​|​`"thirdperson_righthand"`​|第三人称中的右手（F5 视图或其他玩家）|
|​`THIRD_PERSON_LEFT_HAND`​|​`"thirdperson_lefthand"`​|第三人称中的左手（F5 视图或其他玩家）|
|​`FIRST_PERSON_RIGHT_HAND`​|​`"firstperson_righthand"`​|第一人称中的右手|
|​`FIRST_PERSON_LEFT_HAND`​|​`"firstperson_lefthand"`​|第一人称中的左手|
|​`HEAD`​|​`"head"`​|当在玩家的头部盔甲槽中时（通常只能通过命令实现）|
|​`GUI`​|​`"gui"`​|库存、玩家快捷栏|
|​`GROUND`​|​`"ground"`​|掉落的物品；注意掉落物品的旋转由掉落物品渲染器处理，而不是模型|
|​`FIXED`​|​`"fixed"`​|物品展示框|
|​`NONE`​|​`"none"`​|代码中的回退用途，不应在 JSON 中使用|

---

### ItemOverrides

​`ItemOverrides`​ 是一个类，提供了一种方式让烘焙模型处理 `ItemStack`​ 的状态并通过 `#resolve`​ 方法返回一个新的烘焙模型。`#resolve`​ 有五个参数：

* ​**​`BakedModel`​**​：原始模型。
* ​**​`ItemStack`​**​：正在渲染的物品堆叠。
* ​**​`ClientLevel`​**​：渲染模型的关卡。这应仅用于查询关卡，而不是以任何方式修改它。可能为 `null`​。
* ​**​`LivingEntity`​**​：渲染模型的实体。可能为 `null`​，例如从方块实体渲染器渲染时。
* ​**​`int`​**​：用于随机化的种子。

​`ItemOverrides`​ 还持有模型的覆盖选项作为 `BakedOverrides`​。`BakedOverride`​ 对象是模型 `overrides`​ 块的代码表示。烘焙模型可以使用它根据其内容返回不同的模型。可以通过 `ItemOverrides#getOverrides()`​ 获取 `ItemOverrides`​ 实例的所有 `BakedOverrides`​ 列表。

---

### BakedModelWrapper

​`BakedModelWrapper`​ 可用于修改现有的 `BakedModel`​。`BakedModelWrapper`​ 是 `BakedModel`​ 的子类，它接受另一个 `BakedModel`​（“原始”模型）作为构造函数参数，并默认将所有方法重定向到原始模型。然后，你的实现可以仅覆盖选定的方法，如下所示：

```java
// 泛型参数可以选择性地是 BakedModel 的更具体的子类。
// 如果是，则构造函数参数必须匹配该类型。
public class MyBakedModelWrapper extends BakedModelWrapper<BakedModel> {
    // 将原始模型传递给 super。
    public MyBakedModelWrapper(BakedModel originalModel) {
        super(originalModel);
    }
    
    // 在此覆盖你想要的任何方法。如果需要，你也可以访问 originalModel。
}
```

编写模型包装器类后，你必须将包装器应用于它应影响的模型。在客户端事件处理程序 `ModelEvent.ModifyBakingResult`​ 中执行此操作：

```java
@SubscribeEvent
public static void modifyBakingResult(ModelEvent.ModifyBakingResult event) {
    // 对于方块模型
    event.getModels().computeIfPresent(
        // 要修改的模型的模型资源位置。使用 BlockModelShaper#stateToModelLocation 获取，
        // 参数为要影响的方块状态。
        BlockModelShaper.stateToModelLocation(MyBlocksClass.EXAMPLE_BLOCK.defaultBlockState()),
        // 一个 BiFunction，参数为位置和原始模型，返回新模型。
        (location, model) -> new MyBakedModelWrapper(model)
    );
    // 对于物品模型
    event.getModels().computeIfPresent(
        // 要修改的模型的模型资源位置。
        new ModelResourceLocation(
            ResourceLocation.fromNamespaceAndPath("examplemod", "example_item"),
            "inventory"
        ),
        // 一个 BiFunction，参数为位置和原始模型，返回新模型。
        (location, model) -> new MyBakedModelWrapper(model)
    );
}
```

**警告**
通常建议尽可能使用自定义模型加载器而不是在 `ModelEvent.ModifyBakingResult`​ 中包装烘焙模型。如果需要，自定义模型加载器也可以使用 `BakedModelWrapper`​。
