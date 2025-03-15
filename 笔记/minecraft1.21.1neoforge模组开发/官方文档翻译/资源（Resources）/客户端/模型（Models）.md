# 模型（Models）

### 模型（Models）

模型是 JSON 文件，用于确定方块或物品的视觉形状和纹理。模型由立方体元素组成，每个元素都有自己的大小，并为每个面分配纹理。

每个物品都会通过其注册名称分配一个物品模型。例如，注册名称为 `examplemod:example_item`​ 的物品将分配位于 `assets/examplemod/models/item/example_item.json`​ 的模型。对于方块来说，这稍微复杂一些，因为它们首先会分配一个方块状态文件。更多信息见下文。

---

### 规范

参见：[Minecraft Wiki 上的模型](https://minecraft.fandom.com/wiki/Model)

模型是一个 JSON 文件，根标签中包含以下可选属性：

* ​**​`loader`​**​：NeoForge 添加。设置自定义模型加载器。参见 **自定义模型加载器** 了解更多信息。
* ​**​`parent`​**​：设置父模型，形式为相对于 `models`​ 文件夹的资源位置。所有父属性将被应用，然后由声明模型中的属性覆盖。常见的父模型包括：

  * ​`minecraft:block/block`​：所有方块模型的公共父模型。
  * ​`minecraft:block/cube`​：所有使用 1x1x1 立方体模型的父模型。
  * ​`minecraft:block/cube_all`​：立方体模型的变体，所有六个面使用相同的纹理，例如圆石或木板。
  * ​`minecraft:block/cube_bottom_top`​：立方体模型的变体，所有四个水平面使用相同的纹理，顶部和底部使用单独的纹理。常见的例子包括砂岩或錾制石英。
  * ​`minecraft:block/cube_column`​：立方体模型的变体，具有侧面纹理和底部/顶部纹理。例子包括原木、石英柱和紫珀柱。
  * ​`minecraft:block/cross`​：使用两个相同纹理的平面模型，一个顺时针旋转 45°，另一个逆时针旋转 45°，从上方看形成一个 X（因此得名）。例子包括大多数植物，例如草、树苗和花。
  * ​`minecraft:item/generated`​：经典 2D 平面物品模型的父模型。游戏中大多数物品使用此模型。忽略 `elements`​ 块，因为其四边形是从纹理生成的。
  * ​`minecraft:item/handheld`​：看起来像是玩家实际持有的 2D 平面物品模型的父模型。主要用于工具。是 `item/generated`​ 的子模型，因此也会忽略 `elements`​ 块。
  * ​`minecraft:builtin/entity`​：指定除粒子纹理外没有其他纹理。如果这是父模型，`BakedModel#isCustomRenderer()`​ 返回 `true`​，以允许使用 `BlockEntityWithoutLevelRenderer`​。
  * 方块物品通常（但不总是）使用其对应的方块模型作为父模型。例如，圆石物品模型使用父模型 `minecraft:block/cobblestone`​。
* ​**​`ambientocclusion`​**​：是否启用环境光遮蔽。仅对方块模型有效。默认为 `true`​。如果你的自定义方块模型有奇怪的阴影，尝试将其设置为 `false`​。
* ​**​`render_type`​**​：参见 **渲染类型**。
* ​**​`gui_light`​**​：可以是 `"front"`​ 或 `"side"`​。如果为 `"front"`​，光线将从前方照射，适用于平面 2D 模型。如果为 `"side"`​，光线将从侧面照射，适用于 3D 模型（尤其是方块模型）。默认为 `"side"`​。仅对物品模型有效。
* ​**​`textures`​**​：一个子对象，将名称（称为纹理变量）映射到纹理位置。纹理变量可以在元素中使用。它们也可以在元素中指定，但留空以便子模型指定它们。

  * 方块模型应额外指定一个粒子纹理。此纹理用于方块掉落、跑步或破坏时的效果。
  * 物品模型还可以使用层纹理，命名为 `layer0`​、`layer1`​ 等，其中索引较高的层渲染在索引较低的层之上（例如 `layer1`​ 渲染在 `layer0`​ 之上）。仅在父模型为 `item/generated`​ 时有效，并且最多支持 5 层（`layer0`​ 到 `layer4`​）。
* ​**​`elements`​**​：立方体元素的列表。
* ​**​`overrides`​**​：覆盖模型的列表。仅对物品模型有效。
* ​**​`display`​**​：一个子对象，包含不同视角的显示选项，参见链接文章了解可能的键。仅对物品模型有效，但通常在方块模型中指定，以便物品模型可以继承显示选项。每个视角是一个可选的子对象，可能包含以下选项（按顺序应用）：

  * ​**​`translation`​**​：模型的平移，指定为 `[x, y, z]`​。
  * ​**​`rotation`​**​：模型的旋转，指定为 `[x, y, z]`​。
  * ​**​`scale`​**​：模型的缩放，指定为 `[x, y, z]`​。
  * ​**​`right_rotation`​**​：NeoForge 添加。缩放后应用的第二次旋转，指定为 `[x, y, z]`​。
  * ​**​`transform`​**​：参见 **根变换**。

**提示**
如果你不确定如何指定某些内容，可以参考一个实现类似功能的原版模型。

---

### 渲染类型（Render Types）

使用可选的 NeoForge 添加的 `render_type`​ 字段，你可以为模型设置渲染类型。如果未设置（所有原版模型都是如此），游戏将回退到 `ItemBlockRenderTypes`​ 中硬编码的渲染类型。如果 `ItemBlockRenderTypes`​ 中也没有该方块的渲染类型，则回退到 `minecraft:solid`​。原版提供了以下默认渲染类型：

* ​**​`minecraft:solid`​**​：用于完全实心的方块，例如石头。
* ​**​`minecraft:cutout`​**​：用于任何像素要么完全实心要么完全透明的方块，例如玻璃。
* ​**​`minecraft:cutout_mipped`​**​：`minecraft:cutout`​ 的变体，会在远距离缩小纹理以避免视觉伪影（Mipmapping）。不适用于物品渲染，因为通常在物品上不需要 Mipmapping 并且可能导致伪影。例如树叶使用此类型。
* ​**​`minecraft:cutout_mipped_all`​**​：`minecraft:cutout_mipped`​ 的变体，同时将 Mipmapping 应用于物品模型。
* ​**​`minecraft:translucent`​**​：用于任何像素可能部分透明的方块，例如染色玻璃。
* ​**​`minecraft:tripwire`​**​：用于需要渲染到天气目标的特殊方块，例如绊线。

选择正确的渲染类型在一定程度上是性能问题。实心渲染比剪切渲染快，剪切渲染比半透明渲染快。因此，你应该为你的用例指定“最严格”的渲染类型，因为它也是最快的。

如果你愿意，还可以添加自己的渲染类型。为此，订阅模组总线事件 `RegisterNamedRenderTypesEvent`​ 并 `#register`​ 你的渲染类型。`#register`​ 有三个或四个参数：

1. 渲染类型的名称。将前缀为你的模组 ID。例如，使用 `"my_cutout"`​ 将提供 `examplemod:my_cutout`​ 作为新的渲染类型（假设你的模组 ID 是 `examplemod`​）。
2. 区块渲染类型。可以使用 `RenderType.chunkBufferLayers()`​ 返回的列表中的任何类型。
3. 实体渲染类型。必须是具有 `DefaultVertexFormat.NEW_ENTITY`​ 顶点格式的渲染类型。
4. 可选：Fabulous 渲染类型。必须是具有 `DefaultVertexFormat.NEW_ENTITY`​ 顶点格式的渲染类型。如果图形模式设置为 Fabulous!，则将使用此渲染类型而不是常规渲染类型。如果省略，则回退到常规渲染类型。如果渲染类型以某种方式使用透明度，通常建议设置此参数。

---

### 元素（Elements）

元素是立方体对象的 JSON 表示。它具有以下属性：

* ​**​`from`​**​：立方体起始角的坐标，指定为 `[x, y, z]`​。以 1/16 方块单位指定。例如，`[0, 0, 0]`​ 是“左下”角，`[8, 8, 8]`​ 是中心，`[16, 16, 16]`​ 是方块的“右上”角。
* ​**​`to`​**​：立方体结束角的坐标，指定为 `[x, y, z]`​。与 `from`​ 一样，以 1/16 方块单位指定。

**提示**
Minecraft 将 `from`​ 和 `to`​ 中的值限制在 `[-16, 32]`​ 范围内。然而，强烈建议不要超过 `[0, 16]`​，因为这会导致光照和/或剔除问题。

* ​**​`neoforge_data`​**​：参见 **额外面数据**。
* ​**​`faces`​**​：一个对象，包含最多 6 个面的数据，分别命名为 `north`​、`south`​、`east`​、`west`​、`up`​ 和 `down`​。每个面具有以下数据：

  * ​**​`uv`​**​：面的 UV 坐标，指定为 `[u1, v1, u2, v2]`​，其中 `u1, v1`​ 是左上角 UV 坐标，`u2, v2`​ 是右下角 UV 坐标。
  * ​**​`texture`​**​：用于面的纹理。必须是以 `#`​ 前缀的纹理变量。例如，如果你的模型有一个名为 `wood`​ 的纹理，你将使用 `#wood`​ 来引用该纹理。技术上可选，如果不存在则使用缺失纹理。
  * ​**​`rotation`​**​：可选。将纹理顺时针旋转 90、180 或 270 度。
  * ​**​`cullface`​**​：可选。告诉渲染引擎在指定方向有完整方块接触时跳过渲染该面。方向可以是 `north`​、`south`​、`east`​、`west`​、`up`​ 或 `down`​。
  * ​**​`tintindex`​**​：可选。指定一个颜色处理器可能使用的色调索引，参见 **着色** 了解更多信息。默认为 `-1`​，表示不着色。
  * ​**​`neoforge_data`​**​：参见 **额外面数据**。

此外，它还可以指定以下可选属性：

* ​**​`shade`​**​：仅用于方块模型。可选。是否对此元素的面应用方向相关的阴影。默认为 `true`​。
* ​**​`rotation`​**​：对象的旋转，指定为包含以下数据的子对象：

  * ​**​`angle`​**​：旋转角度，以度为单位。可以是 -45 到 45 度，步长为 22.5 度。
  * ​**​`axis`​**​：旋转轴。目前无法围绕多个轴旋转对象。
  * ​**​`origin`​**​：可选。旋转的原点，指定为 `[x, y, z]`​。注意这些是绝对值，即不相对于立方体的位置。如果未指定，则使用 `[0, 0, 0]`​。

---

### 额外面数据（Extra Face Data）

额外面数据（`neoforge_data`​）可以应用于元素和元素的单个面。在所有可用上下文中都是可选的。如果同时指定了元素级别和面级别的额外面数据，则面级别的数据将覆盖元素级别的数据。额外数据可以指定以下内容：

* ​**​`color`​**​：用给定颜色为面着色。必须是 ARGB 值。可以指定为字符串或十进制整数（JSON 不支持十六进制字面量）。默认为 `0xFFFFFFFF`​。如果颜色值是常量，可以用作着色的替代。
* ​**​`block_light`​**​：覆盖用于此面的方块光照值。默认为 `0`​。
* ​**​`sky_light`​**​：覆盖用于此面的天空光照值。默认为 `0`​。
* ​**​`ambient_occlusion`​**​：禁用或启用此面的环境光遮蔽。默认为模型中设置的值。

使用自定义的 `neoforge:item_layers`​ 加载器，你还可以指定额外面数据以应用于 `item/generated`​ 模型中的所有几何体。在以下示例中，第 1 层将被着色为红色并以全亮度发光：

```json
{
    "loader": "neoforge:item_layers",
    "parent": "minecraft:item/generated",
    "textures": {
        "layer0": "minecraft:item/stick",
        "layer1": "minecraft:item/glowstone_dust"
    },
    "neoforge_data": {
        "layers": {
            "1": {
                "color": "0xFFFF0000",
                "block_light": 15,
                "sky_light": 15,
                "ambient_occlusion": false
            }
        }
    }
}
```

---

### 覆盖（Overrides）

物品覆盖可以根据浮点值（称为覆盖值）为物品分配不同的模型。例如，弓和弩使用此功能根据拉弓时间更改纹理。覆盖既有模型部分，也有代码部分。

模型可以指定一个或多个覆盖模型，当覆盖值等于或大于给定阈值时使用。例如，弓使用两个不同的属性 `pulling`​ 和 `pull`​。`pulling`​ 被视为布尔值，`1`​ 表示拉弓，`0`​ 表示未拉弓，而 `pull`​ 表示当前拉弓的程度。然后，它使用这些属性来指定在拉弓到 65% 以下（`pulling 1`​，无 `pull`​ 值）、65%（`pulling 1`​，`pull 0.65`​）和 90%（`pulling 1`​，`pull 0.9`​）时使用三种替代模型。如果多个模型适用（因为值不断增加），列表的最后一个元素匹配，因此请确保顺序正确。覆盖如下所示：

```json
{
    // 其他内容
    "overrides": [
        {
            // pulling = 1
            "predicate": {
                "pulling": 1
            },
            "model": "item/bow_pulling_0"
        },
        {
            // pulling = 1, pull >= 0.65
            "predicate": {
                "pulling": 1,
                "pull": 0.65
            },
            "model": "item/bow_pulling_1"
        },
        // pulling = 1, pull >= 0.9
        {
            "predicate": {
                "pulling": 1,
                "pull": 0.9
            },
            "model": "item/bow_pulling_2"
        }
    ]
}
```

代码部分非常简单。假设我们要为物品添加一个名为 `examplemod:property`​ 的属性，我们将在客户端事件处理程序中使用以下代码：

```java
@SubscribeEvent
public static void onClientSetup(FMLClientSetupEvent event) {
    event.enqueueWork(() -> { // ItemProperties#register 不是线程安全的，因此我们需要在主线程上调用它
        ItemProperties.register(
            // 要应用属性的物品。
            ExampleItems.EXAMPLE_ITEM,
            // 属性的 ID。
            ResourceLocation.fromNamespaceAndPath("examplemod", "property"),
            // 计算覆盖值的方法引用。
            // 参数是使用的物品堆叠、关卡上下文、使用物品的玩家和一个随机种子。
            (stack, level, player, seed) -> someMethodThatReturnsAFloat()
        );
    });
}
```

**注意**
原版 Minecraft 只允许浮点值在 0 到 1 之间。NeoForge 修补了这一点，允许任意浮点值。

---

### 根变换（Root Transforms）

在模型的顶层添加 `transform`​ 属性会告诉加载器在应用方块状态文件中的旋转（对于方块模型）或显示块中的变换（对于物品模型）之前，应对所有几何体应用变换。这是由 NeoForge 添加的。

根变换可以通过两种方式指定。第一种方式是作为名为 `matrix`​ 的单个属性，包含一个 3x4 变换矩阵（行主序，省略最后一行），形式为嵌套的 JSON 数组。矩阵是按顺序组合的平移、左旋转、缩放、右旋转和变换原点。示例如下：

```json
{
    // ...
    "transform": {
        "matrix": [
            [0, 0, 0, 0],
            [0, 0, 0, 0],
            [0, 0, 0, 0]
        ]
    }
}
```

第二种方式是指定一个 JSON 对象，包含以下任意组合的条目，按顺序应用：

* ​**​`translation`​**​：相对平移。指定为三维向量 `[x, y, z]`​，如果不存在则默认为 `[0, 0, 0]`​。
* ​**​`rotation`​**​ **或** **​`left_rotation`​**​：在缩放之前应用于平移原点的旋转。默认为无旋转。可以通过以下方式指定：

  * 一个 JSON 对象，包含单个轴到旋转的映射，例如 `{"x": 90}`​。
  * 一个 JSON 对象数组，每个对象包含单个轴到旋转的映射，按指定顺序应用，例如 `[{"x": 90}, {"y": 45}, {"x": -22.5}]`​。
  * 一个包含三个值的数组，每个值指定围绕每个轴的旋转，例如 `[90, 45, -22.5]`​。
  * 一个包含四个值的数组，直接指定四元数，例如 `[0.38268346, 0, 0, 0.9238795]`​（= 围绕 X 轴旋转 45 度）。
* ​**​`scale`​**​：相对于平移原点的缩放。指定为三维向量 `[x, y, z]`​，如果不存在则默认为 `[1, 1, 1]`​。
* ​**​`post_rotation`​**​ **或** **​`right_rotation`​**​：在缩放之后应用于平移原点的旋转。默认为无旋转。指定方式与 `rotation`​ 相同。
* ​**​`origin`​**​：用于旋转和缩放的原点。变换也作为最后一步移动到此点。可以指定为三维向量 `[x, y, z]`​，或使用三个内置值之一：`"corner"`​（= `[0, 0, 0]`​）、`"center"`​（= `[0.5, 0.5, 0.5]`​）或 `"opposing-corner"`​（= `[1, 1, 1]`​，默认）。

---

### 方块状态文件（Blockstate Files）

参见：[Minecraft Wiki 上的方块状态文件](https://minecraft.fandom.com/wiki/Block_states)

方块状态文件用于为不同的方块状态分配不同的模型。每个注册到游戏中的方块必须有一个方块状态文件。为
