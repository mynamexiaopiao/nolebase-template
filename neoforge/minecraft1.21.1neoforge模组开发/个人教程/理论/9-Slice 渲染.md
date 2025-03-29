# 9-Slice 渲染

## 九宫格缩放

9-Slice 渲染（又称 **九宫格缩放**）是一种智能的图像缩放技术，特别适合 GUI 元素的动态尺寸适配。在 Minecraft Mod 开发中，它用于 Tooltip、按钮、边框等需要保持边角不变形的 UI 元素。

基础原理

9-Slice 将一张纹理划分为 9 个区域：

+-----+-------+-----+  
|  1  |   2   |  3  |  ← 顶部边缘（横向拉伸）  
+-----+-------+-----+  
|  4  |   5   |  6  |  ← 中心区域（双向拉伸）  
+-----+-------+-----+  
|  7  |   8   |  9  |  ← 底部边缘（横向拉伸）  
↑     ↑       ↑     ↑  
左侧  中间    右侧  垂直  
边缘  区域    边缘  拉伸  
（纵向拉伸）

## 技术优势

* **边角保真**：四个角落(1,3,7,9)保持原始像素不变
* **边缘平滑**：上下边缘(2,8)仅水平拉伸，左右边缘(4,6)仅垂直拉伸
* **中心灵活**：中间区域(5)可拉伸或平铺
* **动态适配**：完美适应任意目标尺寸

‍

## Minecraft中的实现细节

### 纹理设计规范

* **推荐尺寸**：16x16或32x32像素
* **边框比例**：通常取纹理的1/4作为边框（如4px）
* **设计要点**：

  * 四角包含完整装饰元素
  * 边缘使用可平铺的图案
  * 中心区域尽量简单

‍

相信大家已经理解了，我们可以通过9-Slice 渲染，来达到我们要的效果

示例（来自deepseek）：

```java
private static final int BORDER = 4;//边框宽度
private static final int TEXTURE_SIZE = 16; // 原图尺寸
```

```java
public static void render9Slice(GuiGraphics guiGraphics, ResourceLocation texture, 
                              int x, int y, int width, int height) {
    // 绑定纹理
    RenderSystem.setShaderTexture(0, texture);
    
    // ► 渲染四个角（不缩放）
    // 左上角
    guiGraphics.blit(texture, x, y, 
        0, 0, BORDER, BORDER, TEXTURE_SIZE, TEXTURE_SIZE);
    // 右上角
    guiGraphics.blit(texture, x + width - BORDER, y, 
        TEXTURE_SIZE - BORDER, 0, BORDER, BORDER, TEXTURE_SIZE, TEXTURE_SIZE);
    // 左下角
    guiGraphics.blit(texture, x, y + height - BORDER, 
        0, TEXTURE_SIZE - BORDER, BORDER, BORDER, TEXTURE_SIZE, TEXTURE_SIZE);
    // 右下角
    guiGraphics.blit(texture, x + width - BORDER, y + height - BORDER, 
        TEXTURE_SIZE - BORDER, TEXTURE_SIZE - BORDER, BORDER, BORDER, TEXTURE_SIZE, TEXTURE_SIZE);

    // ► 渲染四条边（单向拉伸）
    // 上边（横向拉伸）
    guiGraphics.blit(texture, 
        x + BORDER, y, width - 2 * BORDER, BORDER, // 目标尺寸
        BORDER, 0, TEXTURE_SIZE - 2 * BORDER, BORDER, // 源纹理区域
        TEXTURE_SIZE, TEXTURE_SIZE);
    // 下边（横向拉伸）
    guiGraphics.blit(texture, 
        x + BORDER, y + height - BORDER, width - 2 * BORDER, BORDER,
        BORDER, TEXTURE_SIZE - BORDER, TEXTURE_SIZE - 2 * BORDER, BORDER,
        TEXTURE_SIZE, TEXTURE_SIZE);
    // 左边（纵向拉伸）
    guiGraphics.blit(texture, 
        x, y + BORDER, BORDER, height - 2 * BORDER,
        0, BORDER, BORDER, TEXTURE_SIZE - 2 * BORDER,
        TEXTURE_SIZE, TEXTURE_SIZE);
    // 右边（纵向拉伸）
    guiGraphics.blit(texture, 
        x + width - BORDER, y + BORDER, BORDER, height - 2 * BORDER,
        TEXTURE_SIZE - BORDER, BORDER, BORDER, TEXTURE_SIZE - 2 * BORDER,
        TEXTURE_SIZE, TEXTURE_SIZE);

    // ► 渲染中心区域（双向拉伸/平铺）
    guiGraphics.blit(texture, 
        x + BORDER, y + BORDER, width - 2 * BORDER, height - 2 * BORDER,
        BORDER, BORDER, TEXTURE_SIZE - 2 * BORDER, TEXTURE_SIZE - 2 * BORDER,
        TEXTURE_SIZE, TEXTURE_SIZE);
}
```

‍
