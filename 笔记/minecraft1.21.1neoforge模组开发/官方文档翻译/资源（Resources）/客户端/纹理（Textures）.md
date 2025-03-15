# 纹理（Textures）

### 纹理（Textures）

Minecraft 中的所有纹理都是 PNG 文件，位于命名空间的 `textures`​ 文件夹中。不支持 JPG、GIF 和其他图像格式。引用纹理的资源位置的路径通常相对于 `textures`​ 文件夹，因此例如，资源位置 `examplemod:block/example_block`​ 指的是位于 `assets/examplemod/textures/block/example_block.png`​ 的纹理文件。

纹理的大小通常应为 2 的幂次方，例如 16x16 或 32x32。与旧版本不同，现代 Minecraft 原生支持大于 16x16 的方块和物品纹理大小。对于不是 2 的幂次方的纹理（例如 GUI 背景），你可以在下一个可用的 2 的幂次方大小（通常是 256x256）中创建一个空文件，并将你的纹理放在该文件的左上角，其余部分留空。然后可以在使用纹理的代码中设置实际绘制的纹理大小。

---

### 纹理元数据（Texture Metadata）

纹理元数据可以在与纹理文件同名且带有 `.mcmeta`​ 后缀的文件中指定。例如，位于 `textures/block/example.png`​ 的动画纹理需要一个伴随的 `textures/block/example.png.mcmeta`​ 文件。`.mcmeta`​ 文件的格式如下（所有内容都是可选的）：

```json
{
    // 纹理在需要时是否会被模糊处理。默认为 false。
    // 目前由编解码器指定，但在文件和代码中均未使用。
    "blur": true,
    // 纹理在需要时是否会被夹紧。默认为 false。
    // 目前由编解码器指定，但在文件和代码中均未使用。
    "clamp": true,
    "gui": {
        // 指定纹理在需要时如何缩放。可以是以下三种之一：
        "scaling": {
            "type": "stretch" // 默认
        },
        "scaling": {
            "type": "tile",
            "width": 16,
            "height": 16
        },
        "scaling": {
            // 类似于 "tile"，但允许指定边框偏移。
            "type": "nine_slice",
            "width": 16,
            "height": 16,
            // 也可以是一个整数，用作所有四个边的值。
            "border": {
                "left": 0,
                "top": 0,
                "right": 0,
                "bottom": 0
            }
        }
    },
    // 见下文。
    "animation": {}
}
```

---

### 动画纹理（Animated Textures）

Minecraft 原生支持方块和物品的动画纹理。动画纹理由一个纹理文件组成，其中不同的动画阶段位于彼此下方（例如，一个 16x16 的动画纹理，有 8 个阶段，将表示为一个 16x128 的 PNG 文件）。

为了实际显示为动画而不仅仅是扭曲的纹理，纹理元数据中必须有一个动画对象。该子对象可以为空，但可以包含以下可选条目：

```json
{
    "animation": {
        // 自定义帧播放顺序。如果省略，帧将从顶部到底部播放。
        "frames": [1, 0],
        // 一帧在切换到下一个动画阶段之前停留的时间，以帧为单位。默认为 1。
        "frametime": 5,
        // 是否在动画阶段之间插值。默认为 false。
        "interpolate": true,
        // 一个动画阶段的宽度和高度。如果省略，则使用纹理宽度作为这两个值。
        "width": 12,
        "height": 12
    }
}
```
