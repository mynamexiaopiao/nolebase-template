# ItemColor使用教程

原版的刷怪蛋，每个生物只是换了其中的颜色，那有必要为每个生物蛋创建专用贴图吗？显然没有必要，那原版的生物蛋是怎么实现自动生成的呢，其实用的是`ItemColor`系统来处理的

在 NeoForge 1.21.1 中，`ItemColor` 用于为物品（Item）或方块（Block）的动态纹理上色。它通常用于以下场景：

- 为不同状态的物品（例如不同耐久度、不同 NBT（1.21.1已废弃） 数据）提供不同的颜色。
- 为方块的物品形式（例如草方块、树叶）提供动态颜色。

以下是 `ItemColor` 的使用方法，适用于 NeoForge 1.21.1。

## ItemColor接口

源码：

```java
@OnlyIn(Dist.CLIENT)  
public interface ItemColor {  
    int getColor(ItemStack stack, int tintIndex);  
}
```

参数解释:

- `stack`: 要染色的物品堆
- `tintIndex`: 物品模型中的染色层索引
- 返回值: 一个表示ARGB颜色的整数（格式为0xAARRGGBB）

## 监听registerItemColors事件

类型：客户端事件
示例代码：
这个代码是我们注册了一个剑，它的剑身的剑柄是分开用`ItemColor `渲染的，然后拼在一起

```java
@SubscribeEvent  
@OnlyIn(Dist.CLIENT)  
public static void registerItemColors(RegisterColorHandlersEvent.Item event) {  
    ItemColor itemColor = new ItemColor() {  
  
        @Override  
        public int getColor(ItemStack stack, int tintIndex) {  
        //tintIndex类似于图层,数字是我们json文件 layer+数字 中的数字
            if (tintIndex==0 ){  
                return 0xFF000000;
            }else if (tintIndex==1){  
                return 0xFF553322;
            }  
            
        }  
    };  
    //添加进我们的物品
    event.register(itemColor, ModItems.MAGIC_SWORD.get());  
}
```

## 物品模型

我们可以实现渲染的核心就在json中，在其中可以指定很多东西
这里就要涉及json的写法了

#### 默认的样式（简单物品模型）

首先，我们要定义一些默认的物品样式
![[color_changing_sword_blade.png]]
![[color_changing_sword_handle.png]]
上面两个就是剑柄和剑身，只不过被分离开来，并且转为了灰度图
它们的路径src\main\resources\assets\yourmod\textures\item\两个图片的.png

json文件src\main\resources\assets\yourmod\models\item\物品注册名.json

```json
{ 
	"parent": "item/generated", 
	"textures": { 
		"layer0": "modid:item/my_colored_item_base",
		"layer1": "modid:item/my_colored_item_overlay" 
		}
}
```

这里就将两个部分拼接了起来

在这种情况下，`tintindex` 是隐式分配的：

- `layer0` 自动获得 `tintindex=0`
- `layer1` 自动获得 `tintindex=1`
- 以此类推...

#### 模型变体和覆盖规则

物品可以根据不同条件显示不同的模型，这是通过 `overrides` 属性实现的：

```json
{
  "parent": "item/generated",
  "textures": {
    "layer0": "modid:item/my_colored_item_base"
  },
  "overrides": [
    {
      "predicate": {
        "modid:custom_property": 0.5
      },
      "model": "modid:item/my_colored_item_variant_1"
    },
    {
      "predicate": {
        "modid:custom_property": 1.0
      },
      "model": "modid:item/my_colored_item_variant_2"
    },
    {
      "predicate": {
        "minecraft:damage": 0.5
      },
      "model": "modid:item/my_colored_item_damaged"
    }
  ]
}
```

`overrides` 详解：

- `predicate`: 定义触发条件
  - 可以使用原版谓词如 `minecraft:damage`、`minecraft:broken` 等
  - 也可以定义自定义谓词 (需要在代码中注册)
- `model`: 满足条件时使用的替代模型

## ItemColor的原理[​](https://arkdust-tutorials.kituin.fun/render/item/color.html#itemcolor的原理)

以下内容引用自https://arkdust-tutorials.kituin.fun/render/item/color.html

事件中保有一个`ItemColors`变量，这一实例用于总控所有材质调色内容。在使用事件注册了颜色后，内容都将在这里进行存储。

在`ItemRenderer#renderQuadList`方法中，也就是物品渲染阶段，将会从其中拉取颜色进行运算处理，统一烘培。这涉及到底层的着色原理，因此在此不进行升入研讨。

```java
public void renderQuadList(PoseStack pPoseStack, VertexConsumer pBuffer, List<BakedQuad> pQuads, ItemStack pItemStack, int pCombinedLight, int pCombinedOverlay) {
    boolean flag = !pItemStack.isEmpty();
    PoseStack.Pose posestack$pose = pPoseStack.last();

    for (BakedQuad bakedquad : pQuads) {
        int i = -1;
        if (flag && bakedquad.isTinted()) {
            i = this.itemColors.getColor(pItemStack, bakedquad.getTintIndex());
        }

        float f = (float) (i >> 16 & 0xFF) / 255.0F;
        float f1 = (float) (i >> 8 & 0xFF) / 255.0F;
        float f2 = (float) (i & 0xFF) / 255.0F;
        pBuffer.putBulkData(posestack$pose, bakedquad, f, f1, f2, 1.0F, pCombinedLight, pCombinedOverlay, true);
    }
}
```

好了，ItemColor的教程就到这里了，其实方块，模型也可以这样，大家可以自己探索。
