# 自定义模型加载器（Custom Model Loaders）

### 自定义模型加载器（Custom Model Loaders）

模型只是一个形状。它可以是一个立方体、一组立方体、一组三角形或任何其他几何形状（或几何形状的集合）。在大多数情况下，模型的定义方式并不重要，因为最终所有内容都会在内存中转换为 `BakedModel`​。因此，NeoForge 添加了注册自定义模型加载器的功能，这些加载器可以将任何你想要的模型转换为游戏使用的 `BakedModel`​。

方块模型的入口点仍然是模型 JSON 文件。然而，你可以在 JSON 的根目录中指定一个 `loader`​ 字段，该字段将替换默认加载器为你自己的加载器。自定义模型加载器可以忽略默认加载器所需的所有字段。

---

### 内置模型加载器

除了默认的模型加载器外，NeoForge 还提供了几个内置加载器，每个加载器都有不同的用途。

#### 复合模型（Composite Model）

复合模型可用于在父模型中指定不同的模型部分，并仅在子模型中应用其中一些部分。这最好通过一个示例来说明。考虑以下位于 `examplemod:example_composite_model`​ 的父模型：

```json
{
    "loader": "neoforge:composite",
    // 指定模型部分。
    "children": {
        "part_1": {
            "parent": "examplemod:some_model_1"
        },
        "part_2": {
            "parent": "examplemod:some_model_2"
        }
    },
    "visibility": {
        // 默认禁用 part_2。
        "part_2": false
    }
}
```

然后，我们可以在 `examplemod:example_composite_model`​ 的子模型中禁用和启用各个部分：

```json
{
    "parent": "examplemod:example_composite_model",
    // 覆盖可见性。如果缺少某个部分，它将使用父模型的可见性值。
    "visibility": {
        "part_1": false,
        "part_2": true
    }
}
```

要生成此模型的数据，请使用自定义加载器类 `CompositeModelBuilder`​。

---

#### 动态流体容器模型（Dynamic Fluid Container Model）

动态流体容器模型（也称为动态桶模型，以其最常见的用例命名）用于表示流体容器（如桶或罐）并希望在模型中显示流体的物品。这仅在流体数量固定（例如只有熔岩和粉雪）时有效，如果流体是任意的，请改用 `BlockEntityWithoutLevelRenderer`​。

```json
{
    "loader": "neoforge:fluid_container",
    // 必需。必须是有效的流体 ID。
    "fluid": "minecraft:water",
    // 加载器通常需要两个纹理：base 和 fluid。
    "textures": {
        // 基础容器纹理，即空桶的等效物。
        "base": "examplemod:item/custom_container",
        // 流体纹理，即桶中水的等效物。
        "fluid": "examplemod:item/custom_container_fluid"
    },
    // 可选，默认为 false。是否将模型倒置，用于气态流体。
    "flip_gas": true,
    // 可选，默认为 true。是否将盖子用作遮罩。
    "cover_is_mask": false,
    // 可选，默认为 true。是否将流体的亮度应用于物品模型。
    "apply_fluid_luminosity": false,
}
```

通常情况下，动态流体容器模型会直接使用桶模型。这是通过指定 `neoforge:item_bucket`​ 父模型来完成的，如下所示：

```json
{
    "loader": "neoforge:fluid_container",
    "parent": "neoforge:item/bucket",
    // 替换为你自己的流体。
    "fluid": "minecraft:water"
    // 可选属性。请注意，纹理由父模型处理。
}
```

要生成此模型的数据，请使用自定义加载器类 `DynamicFluidContainerModelBuilder`​。请注意，出于遗留支持的原因，此类还提供了设置 `apply_tint`​ 属性的方法，该属性不再使用。

---

#### 元素模型（Elements Model）

元素模型由方块模型元素和一个可选的根变换组成。主要用于常规模型渲染之外的用途，例如在方块实体渲染器（BER）中。

```json
{
    "loader": "neoforge:elements",
    "elements": [...],
    "transform": {...}
}
```

---

#### 空模型（Empty Model）

空模型完全不渲染任何内容。

```json
{
    "loader": "neoforge:empty"
}
```

---

#### 物品层模型（Item Layer Model）

物品层模型是标准 `item/generated`​ 模型的变体，提供以下附加功能：

* 无限数量的层（而不是默认的 5 层）
* 每层的渲染类型

```json
{
    "loader": "neoforge:item_layers",
    "textures": {
        "layer0": "...",
        "layer1": "...",
        "layer2": "...",
        "layer3": "...",
        "layer4": "...",
        "layer5": "...",
    },
    "render_types": {
        // 将渲染类型映射到层号。例如，层 0、2 和 4 将使用 cutout。
        "minecraft:cutout": [0, 2, 4],
        "minecraft:cutout_mipped": [1, 3],
        "minecraft:translucent": [5]
    },
    // 默认加载器允许的其他内容
}
```

要生成此模型的数据，请使用自定义加载器类 `ItemLayerModelBuilder`​。

---

#### OBJ 模型（OBJ Model）

OBJ 模型加载器允许你在游戏中使用 Wavefront `.obj`​ 3D 模型，从而允许在模型中包含任意形状（包括三角形、圆形等）。`.obj`​ 模型必须放置在 `models`​ 文件夹（或其子文件夹）中，并且必须提供具有相同名称的 `.mtl`​ 文件（或手动设置），例如，位于 `models/block/example.obj`​ 的 OBJ 模型必须具有位于 `models/block/example.mtl`​ 的相应 MTL 文件。

```json
{
    "loader": "neoforge:obj",
    // 必需。引用模型文件。请注意，这是相对于命名空间根目录的，而不是模型文件夹。
    "model": "examplemod:models/example.obj",
    // 通常，.mtl 文件必须放置在与 .obj 文件相同的位置，只有文件扩展名不同。
    // 这将使加载器自动拾取它们。但是，如果需要，你也可以手动设置 .mtl 文件的位置。
    "mtl_override": "examplemod:models/example_other_name.mtl",
    // 这些纹理可以在 .mtl 文件中引用为 #texture0、#particle 等。
    // 这通常需要手动编辑 .mtl 文件。
    "textures": {
        "texture0": "minecraft:block/cobblestone",
        "particle": "minecraft:block/stone"
    },
    // 启用或禁用模型的自动剔除。可选，默认为 true。
    "automatic_culling": false,
    // 是否对模型进行着色。可选，默认为 true。
    "shade_quads": false,
    // 某些建模程序会假设 V=0 为底部而不是顶部。此属性将 V 上下翻转。
    // 可选，默认为 false。
    "flip_v": true,
    // 是否启用自发光。可选，默认为 true。
    "emissive_ambient": false
}
```

要生成此模型的数据，请使用自定义加载器类 `ObjModelBuilder`​。

---

#### 分离变换模型（Separate Transforms Model）

分离变换模型可用于根据视角在不同模型之间切换。视角与普通模型中的 `display`​ 块相同。这是通过指定一个基础模型（作为回退），然后指定每个视角的覆盖模型来实现的。请注意，如果你愿意，每个模型都可以是完全成熟的模型，但通常最简单的方法是使用该模型的子模型来引用另一个模型，如下所示：

```json
{
    "loader": "neoforge:separate_transforms",
    // 通常使用圆石模型。
    "base": {
        "parent": "minecraft:block/cobblestone"
    },
    // 仅在掉落时使用石头模型。
    "perspectives": {
        "ground": {
            "parent": "minecraft:block/stone"
        }
    }
}
```

要生成此模型的数据，请使用自定义加载器类 `SeparateTransformsModelBuilder`​。

---

### 创建自定义模型加载器

要创建自己的模型加载器，你需要三个类，以及一个事件处理程序：

1. 几何加载器类
2. 几何类
3. 动态烘焙模型类
4. 用于 `ModelEvent.RegisterGeometryLoaders`​ 的客户端事件处理程序，用于注册几何加载器

为了说明这些类是如何连接的，我们将跟踪一个模型的加载过程：

1. 在模型加载期间，将带有 `loader`​ 属性的模型 JSON 传递给你的几何加载器。几何加载器然后读取模型 JSON 并使用模型 JSON 的属性返回一个几何对象。
2. 在模型烘焙期间，几何对象被烘焙，返回一个动态烘焙模型。
3. 在模型渲染期间，动态烘焙模型用于渲染。

让我们通过一个基本的类设置进一步说明这一点。几何加载器类名为 `MyGeometryLoader`​，几何类名为 `MyGeometry`​，动态烘焙模型类名为 `MyDynamicModel`​：

```java
public class MyGeometryLoader implements IGeometryLoader<MyGeometry> {
    // 强烈建议对几何加载器使用单例模式，因为所有模型都可以通过一个加载器加载。
    public static final MyGeometryLoader INSTANCE = new MyGeometryLoader();
    // 我们将用于注册此加载器的 ID。也用于加载器数据生成类。
    public static final ResourceLocation ID = ResourceLocation.fromNamespaceAndPath("examplemod", "my_custom_loader");
    
    // 根据单例模式，将构造函数设为私有。        
    private MyGeometryLoader() {}
    
    @Override
    public MyGeometry read(JsonObject jsonObject, JsonDeserializationContext context) throws JsonParseException {
        // 使用给定的 JsonObject 和（如果需要）JsonDeserializationContext 从模型 JSON 中获取属性。
        // MyGeometry 构造函数可以有构造函数参数（见下文）。
        return new MyGeometry();
    }
}

public class MyGeometry implements IUnbakedGeometry<MyGeometry> {
    // 构造函数可以有你需要的任何参数，并将它们存储在字段中以供下面使用。
    // 如果构造函数有参数，则 MyGeometryLoader#read 中的构造函数调用必须匹配它们。
    public MyGeometry() {}

    // 负责模型烘焙的方法，返回我们的动态模型。此方法中的参数是：
    // - 几何烘焙上下文。包含我们将传递给模型的许多属性，例如光照和环境光遮蔽值。
    // - 模型烘焙器。可用于烘焙子模型。
    // - 精灵获取器。将材质（= 纹理变量）映射到 TextureAtlasSprites。可以从上下文中获取材质。
    //   例如，要获取模型的粒子纹理，请调用 spriteGetter.apply(context.getMaterial("particle"));
    // - 模型状态。这包含来自方块状态文件的属性，例如旋转和 uvlock 布尔值。
    // - 物品覆盖。这是物品模型中 "overrides" 块的代码表示。
    @Override
    public BakedModel bake(IGeometryBakingContext context, ModelBaker baker, Function<Material, TextureAtlasSprite> spriteGetter, ModelState modelState, ItemOverrides overrides) {
        // 有关参数的更多信息，请参见下文。
        return new MyDynamicModel(context.useAmbientOcclusion(), context.isGui3d(), context.useBlockLight(),
            spriteGetter.apply(context.getMaterial("particle")), overrides);
    }

    // 负责正确解析父属性的方法。如果此模型加载任何嵌套模型或在其自身上重用原版加载器（见下文），则需要此方法。
    @Override
    public void resolveParents(Function<ResourceLocation, UnbakedModel> modelGetter, IGeometryBakingContext context) {
        // UnbakedModel#resolveParents
    }
}

// 也可以使用 BakedModelWrapper 来返回大多数方法的默认值，从而允许你仅覆盖实际需要覆盖的内容。
public class MyDynamicModel implements IDynamicBakedModel {
    // 缺失纹理的材质。其精灵可以在需要时用作回退。
    private static final Material MISSING_TEXTURE = 
        new Material(TextureAtlas.LOCATION_BLOCKS, MissingTextureAtlasSprite.getLocation());

    // 用于下面方法的属性。可选，如果适用，方法也可以使用常量值。
    private final boolean useAmbientOcclusion;
    private final boolean isGui3d;
    private final boolean usesBlockLight;
    private final TextureAtlasSprite particle;
    private final ItemOverrides overrides;

    // 构造函数不需要除实例化最终字段之外的任何参数。
    // 它可以指定任何其他参数以存储在你认为模型工作所需的字段中。
    public MyDynamicModel(boolean useAmbientOcclusion, boolean isGui3d, boolean usesBlockLight, TextureAtlasSprite particle, ItemOverrides overrides) {
        this.useAmbientOcclusion = useAmbientOcclusion;
        this.isGui3d = isGui3d;
        this.usesBlockLight = usesBlockLight;
        this.particle = particle;
        this.overrides = overrides;
    }

    // 使用我们的属性。有关方法效果的更多信息，请参见烘焙模型文章。
    @Override
    public boolean useAmbientOcclusion() {
        return useAmbientOcclusion;
    }

    @Override
    public boolean isGui3d() {
        return isGui3d;
    }

    @Override
    public boolean usesBlockLight() {
        return usesBlockLight;
    }

    @Override
    public TextureAtlasSprite getParticleIcon() {
        // 如果不需要粒子（例如在物品模型上下文中），则返回 MISSING_TEXTURE.sprite()。
        return particle;
    }

    @Override
    public ItemOverrides getOverrides() {
        // 在方块模型上下文中返回 ItemOverrides.EMPTY。
        return overrides;
    }

    // 如果要使用自定义方块实体渲染器而不是默认渲染器，请覆盖此方法为 true。
    @Override
    public boolean isCustomRenderer() {
        return false;
    }

    // 这是魔法发生的地方。在此处返回要渲染的四边形列表。参数是：
    // - 正在渲染的方块状态。如果渲染物品，则可能为 null。
    // - 正在剔除的面。可能为 null，这意味着应返回无法被遮挡的四边形。
    // - 客户端绑定的随机源，可用于随机化内容。
    // - 要使用的额外数据。源自方块实体（如果存在）或 BakedModel#getModelData()。
    // - 请求四边形的渲染类型。
    // 注意：这可能会在短时间内多次调用，每个方块最多调用几次。
    // 这应尽可能快，并在适用的情况下使用缓存。
    @Override
    public List<BakedQuad> getQuads(@Nullable BlockState state, @Nullable Direction side, RandomSource rand, ModelData extraData, @Nullable RenderType renderType) {
        List<BakedQuad> quads = new ArrayList<>();
        // 根据需要在此处将元素添加到四边形列表
        return quads;
    }
}
```

完成后，别忘了实际注册你的加载器，否则所有工作都将白费：

```java
// 客户端模组总线事件处理程序
@SubscribeEvent
public static void registerGeometryLoaders(ModelEvent.RegisterGeometryLoaders event) {
    event.register(MyGeometryLoader.ID, MyGeometryLoader.INSTANCE);
}
```

---

### 数据生成（Datagen）

当然，我们也可以生成模型的数据。为此，我们需要一个扩展 `CustomLoaderBuilder`​ 的类：

```java
// 这假设是一个方块模型。如果你正在制作自定义物品模型，请改用 ItemModelBuilder 作为泛型参数。
public class MyLoaderBuilder extends CustomLoaderBuilder<BlockModelBuilder> {
    public MyLoaderBuilder(BlockModelBuilder parent, ExistingFileHelper existingFileHelper) {
        super(
            // 你的模型加载器的 ID。
            MyGeometryLoader.ID,
            // 我们使用的父构建器。这始终是第一个构造函数参数。
            parent,
            // 我们使用的现有文件助手。这始终是第二个构造函数参数。
            existingFileHelper,
            // 如果加载器不存在，是否允许内联原版元素作为回退。
            false
        );
    }
    
    // 在此处添加字段和设置器。然后可以在下面使用这些字段。
    
    // 将模型序列化为 JSON。
    @Override
    public JsonObject toJson(JsonObject json) {
        // 将你的字段添加到给定的 JsonObject。
        // 然后调用 super，它添加 loader 属性和一些其他内容。
        return super.toJson(json);
    }
}
```

要使用此加载器构建器，请在方块（或物品）模型数据生成期间执行以下操作：

```java
// 这假设是一个 BlockStateProvider。在 ItemModelProvider 中直接使用 getBuilder("my_cool_block")。
// customLoader() 的参数是一个 BiFunction。BiFunction 的参数
// 是 getBuilder() 调用的结果和提供者的 ExistingFileHelper。
MyLoaderBuilder loaderBuilder = models().getBuilder("my_cool_block").customLoader(MyLoaderBuilder::new);
```

然后，在 `loaderBuilder`​ 上调用你的字段设置器。

---

### 可见性（Visibility）

​`CustomLoaderBuilder`​ 的默认实现包含应用可见性的方法。你可以选择在模型加载器中使用或忽略可见性属性。目前，只有复合模型加载器使用此属性。

---

### 重用默认模型加载器

在某些情况下，重用原版模型加载器并在其基础上构建模型逻辑而不是完全替换它是有意义的。我们可以使用一个巧妙的技巧来实现这一点：在模型加载器中，我们只需删除 `loader`​ 属性并将其发送回模型反序列化器，使其认为这是一个常规模型。然后我们将其传递给几何对象，在那里烘焙模型几何（就像默认几何处理程序那样），并将其传递给动态模型，在那里我们可以以任何我们想要的方式使用模型的四边形：

```java
public class MyGeometryLoader implements IGeometryLoader<MyGeometry> {
    public static final MyGeometryLoader INSTANCE = new MyGeometryLoader();
    public static final ResourceLocation ID = ResourceLocation.fromNamespaceAndPath(...);
    
    private MyGeometryLoader() {}
    
    @Override
    public MyGeometry read(JsonObject jsonObject, JsonDeserializationContext context) throws JsonParseException {
        // 通过删除 loader 字段并将其传递回反序列化器，使反序列化器认为这是一个普通模型。
        jsonObject.remove("loader");
        BlockModel base = context.deserialize(jsonObject, BlockModel.class);
        // 如果需要，在此处添加其他内容
        return new MyGeometry(base);
    }
}

public class MyGeometry implements IUnbakedGeometry<MyGeometry> {
    private final BlockModel base;

    // 存储方块模型以供下面使用。            
    public MyGeometry(BlockModel base) {
        this.base = base;
    }

    @Override
    public BakedModel bake(IGeometryBakingContext context, ModelBaker baker, Function<Material, TextureAtlasSprite> spriteGetter, ModelState modelState, ItemOverrides overrides) {
        BakedModel bakedBase = new ElementsModel(base.getElements()).bake(context, baker, spriteGetter, modelState, overrides);
        return new MyDynamicModel(bakedBase, /* 其他参数在此 */);
    }

    @Override
    public void resolveParents(Function<ResourceLocation, UnbakedModel> modelGetter, IGeometryBakingContext context) {
        base.resolveParents(modelGetter);
    }
}

public class MyDynamicModel implements IDynamicBakedModel {
    private final BakedModel base;
    // 其他字段在此

    public MyDynamicModel(BakedModel base, /* 其他参数在此 */) {
        this.base = base;
        // 在此设置其他字段
    }

    // 其他覆盖方法在此

    @Override
    public List<BakedQuad> getQuads(@Nullable BlockState state, @Nullable Direction side, RandomSource rand, ModelData extraData, @Nullable RenderType renderType) {
        List<BakedQuad> quads = new ArrayList<>();
        // 添加基础模型的四边形。也可以根据需要在此处对四边形执行其他操作。
        quads.addAll(base.getQuads(state, side, rand, extraData, renderType));
        // 根据需要在此处将其他元素添加到四边形列表
        return quads;
    }
    
    // 将基础模型的变换也应用于我们的模型。
    @Override
    public BakedModel applyTransform(ItemDisplayContext transformType, PoseStack poseStack, boolean applyLeftHandTransform) {
        return base.applyTransform(transformType, poseStack, applyLeftHandTransform);
    }
}
```
