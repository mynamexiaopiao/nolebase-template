# 屏幕系统（Screens）

### 屏幕系统（Screens）

屏幕系统通常是 Minecraft 中所有图形用户界面（GUI）的基础：接收用户输入，在服务器上验证输入，并将结果操作同步回客户端。它们可以与菜单结合使用，创建类似库存视图的通信网络，或者它们可以是独立的，模组开发者可以通过自己的网络实现来处理。

屏幕由许多部分组成，因此很难完全理解 Minecraft 中的“屏幕”到底是什么。因此，本文将逐一介绍屏幕的各个组件及其应用方式，然后再讨论屏幕本身。

---

#### 相对坐标（Relative Coordinates）

每当渲染任何内容时，都需要一些标识符来指定它出现的位置。在 Minecraft 中，大多数渲染调用都使用坐标平面中的 x、y 和 z 值。X 值从左到右增加，y 值从上到下增加，z 值从远到近增加。然而，这些坐标并不固定在一个特定的范围内。它们的范围可以根据屏幕的大小和游戏选项中指定的“GUI 缩放比例”而变化。因此，必须特别注意确保传递给渲染调用的坐标值能够正确缩放（即相对化）以适应可变的屏幕大小。

有关如何相对化坐标的信息将在屏幕部分中介绍。

**注意**：如果你选择使用固定坐标或不正确地缩放屏幕，渲染的对象可能会看起来奇怪或错位。检查你是否正确相对化坐标的一个简单方法是点击视频设置中的“GUI 缩放”按钮。该值用于在确定 GUI 渲染比例时除以显示器的宽度和高度。

---

#### GuiGraphics（GUI 图形）

Minecraft 中渲染的任何 GUI 通常都是使用 `GuiGraphics`​ 完成的。`GuiGraphics`​ 是几乎所有渲染方法的第一个参数；它包含渲染常用对象的基本方法。这些方法分为五类：彩色矩形、字符串、纹理、物品和工具提示。还有一个额外的方法用于渲染组件的片段（`#enableScissor`​ / `#disableScissor`​）。`GuiGraphics`​ 还暴露了 `PoseStack`​，它应用了正确渲染组件所需的变换。此外，颜色采用 ARGB 格式。

---

#### 渲染对象（Renderable）

渲染对象本质上是需要渲染的对象。这些对象包括屏幕、按钮、聊天框、列表等。渲染对象只有一个方法：`#render`​。该方法接受用于渲染到屏幕的 `GuiGraphics`​、鼠标的 x 和 y 位置（缩放为相对屏幕大小）以及帧间时间差（自上一帧以来经过的刻数）。

一些常见的渲染对象包括屏幕和“小部件”（Widgets）：通常是屏幕上渲染的可交互元素，例如 `Button`​、其子类 `ImageButton`​ 和用于在屏幕上输入文本的 `EditBox`​。

---

#### GuiEventListener（GUI 事件监听器）

Minecraft 中渲染的任何屏幕都实现了 `GuiEventListener`​。`GuiEventListener`​ 负责处理用户与屏幕的交互。这些交互包括来自鼠标的输入（移动、点击、释放、拖动、滚动、悬停）和键盘的输入（按下、释放、键入）。每个方法返回关联的操作是否成功影响了屏幕。按钮、聊天框、列表等小部件也实现了此接口。

---

#### ContainerEventHandler（容器事件处理器）

与 `GuiEventListener`​ 几乎同义的是其子类型：`ContainerEventHandler`​。这些处理器负责处理包含小部件的屏幕上的用户交互，管理当前聚焦的元素以及如何应用关联的交互。`ContainerEventHandler`​ 添加了三个额外功能：可交互的子元素、拖动和聚焦。

事件处理器持有子元素，用于确定元素的交互顺序。在鼠标事件处理器期间（不包括拖动），鼠标悬停的第一个子元素将执行其逻辑。

拖动元素（通过 `#mouseClicked`​ 和 `#mouseReleased`​ 实现）提供了更精确的执行逻辑。

聚焦允许在事件执行期间首先检查并处理特定的子元素，例如在键盘事件或拖动鼠标期间。通常通过 `#setFocused`​ 设置聚焦。此外，可以使用 `#nextFocusPath`​ 循环可交互的子元素，根据传入的 `FocusNavigationEvent`​ 选择子元素。

**注意**：屏幕通过 `AbstractContainerEventHandler`​ 实现 `ContainerEventHandler`​，它添加了用于拖动和聚焦子元素的设置器和获取器逻辑。

---

#### NarratableEntry（可叙述条目）

​`NarratableEntry`​ 是可以通过 Minecraft 的辅助功能叙述功能讲述的元素。每个元素可以根据悬停或选择的内容提供不同的叙述，通常按聚焦、悬停和其他情况的优先级排序。

​`NarratableEntry`​ 有三个方法：一个确定元素的优先级（`#narrationPriority`​），一个确定是否讲述叙述（`#isActive`​），最后一个提供叙述内容（`#updateNarration`​）。

**注意**：Minecraft 中的所有小部件都是 `NarratableEntry`​，因此如果使用可用的子类型，通常不需要手动实现。

---

#### 屏幕子类型（The Screen Subtype）

有了上述知识，就可以构建一个基本的屏幕。为了更容易理解，屏幕的组件将按通常遇到的顺序进行介绍。

首先，所有屏幕都接受一个 `Component`​，它表示屏幕的标题。此组件通常由其子类型绘制到屏幕上。它仅用于基本屏幕的叙述消息。

```java
// 在某个 Screen 子类中
public MyScreen(Component title) {
    super(title);
}
```

---

#### 初始化（Initialization）

一旦屏幕初始化完成，就会调用 `#init`​ 方法。`#init`​ 方法从 Minecraft 实例中设置屏幕的初始设置，包括相对宽度和高度（根据游戏缩放比例）。任何设置（例如添加小部件或预计算相对坐标）都应在此方法中完成。如果游戏窗口调整大小，屏幕将重新初始化，调用 `#init`​ 方法。

有三种方法可以向屏幕添加小部件，每种方法都有不同的用途：

|方法|描述|
| ------| ----------------------------------------|
|​`#addWidget`​|添加可交互和可叙述的小部件，但不渲染。|
|​`#addRenderableOnly`​|添加仅渲染的小部件；不可交互或叙述。|
|​`#addRenderableWidget`​|添加可交互、可叙述和可渲染的小部件。|

通常，`#addRenderableWidget`​ 会最常用。

```java
// 在某个 Screen 子类中
@Override
protected void init() {
    super.init();

    // 添加小部件和预计算值
    this.addRenderableWidget(new EditBox(/* ... */));
}
```

---

#### 屏幕的刻更新（Ticking Screens）

屏幕还使用 `#tick`​ 方法进行刻更新，以执行一些客户端逻辑用于渲染目的。

```java
// 在某个 Screen 子类中
@Override
public void tick() {
    super.tick();

    // 每帧执行一些逻辑
}
```

---

#### 输入处理（Input Handling）

由于屏幕是 `GuiEventListener`​ 的子类型，因此可以重写输入处理器，例如处理特定按键按下的逻辑。

---

#### 渲染屏幕（Rendering the Screen）

最后，屏幕通过 `Renderable`​ 子类型提供的 `#render`​ 方法进行渲染。如前所述，`#render`​ 方法每帧绘制屏幕上需要渲染的所有内容，例如背景、小部件、工具提示等。默认情况下，`#render`​ 方法仅将小部件渲染到屏幕上。

屏幕中最常见的两个渲染内容（通常不由子类型处理）是背景和工具提示。

背景可以使用 `#renderBackground`​ 渲染，其中一个方法接受一个 v 偏移量，用于在无法渲染背景时渲染选项背景。

工具提示通过 `GuiGraphics#renderTooltip`​ 或 `GuiGraphics#renderComponentTooltip`​ 渲染，可以接受渲染的文本组件、可选的自定义工具提示组件以及工具提示在屏幕上渲染的 x/y 相对坐标。

```java
// 在某个 Screen 子类中

// mouseX 和 mouseY 表示光标在屏幕上的缩放坐标
@Override
public void render(GuiGraphics graphics, int mouseX, int mouseY, float partialTick) {
    // 背景通常首先渲染
    this.renderBackground(graphics);

    // 在小部件之前渲染内容（背景纹理）

    // 然后渲染小部件（如果这是屏幕的直接子类）
    super.render(graphics, mouseX, mouseY, partialTick);

    // 在小部件之后渲染内容（工具提示）
}
```

---

#### 关闭屏幕（Closing the Screen）

当屏幕关闭时，两个方法处理清理工作：`#onClose`​ 和 `#removed`​。

​`#onClose`​ 在用户输入关闭当前屏幕时调用。此方法通常用作回调，以销毁并保存屏幕本身的任何内部进程。这包括向服务器发送数据包。

​`#removed`​ 在屏幕更改之前调用，并释放给垃圾回收器。它处理在屏幕打开之前未重置回初始状态的任何内容。

```java
// 在某个 Screen 子类中

@Override
public void onClose() {
    // 在此处停止任何处理器

    // 最后调用，以防干扰重写
    super.onClose();
}

@Override
public void removed() {
    // 在此处重置初始状态

    // 最后调用，以防干扰重写
    super.removed();
}
```

---

#### AbstractContainerScreen（抽象容器屏幕）

如果屏幕直接附加到菜单，则应改为子类化 `AbstractContainerScreen`​。`AbstractContainerScreen`​ 充当菜单的渲染器和输入处理器，并包含与槽位同步和交互的逻辑。因此，通常只需要重写或实现两个方法即可拥有一个可用的容器屏幕。再次为了更容易理解，容器屏幕的组件将按通常遇到的顺序进行介绍。

​`AbstractContainerScreen`​ 通常需要三个参数：打开的容器菜单（由泛型 `T`​ 表示）、玩家库存（仅用于显示名称）和屏幕本身的标题。在此处可以设置一些定位字段：

|字段|描述|
| ------| --------------------------------------------------------------|
|​`imageWidth`​|用于背景的纹理宽度。通常在 256 x 256 的 PNG 中，默认为 176。|
|​`imageHeight`​|用于背景的纹理高度。通常在 256 x 256 的 PNG 中，默认为 166。|
|​`titleLabelX`​|屏幕标题渲染的相对 x 坐标。|
|​`titleLabelY`​|屏幕标题渲染的相对 y 坐标。|
|​`inventoryLabelX`​|玩家库存名称渲染的相对 x 坐标。|
|​`inventoryLabelY`​|玩家库存名称渲染的相对 y 坐标。|

**注意**：在上一节中提到，预计算的相对坐标应在 `#init`​ 方法中设置。这仍然成立，因为此处提到的值不是预计算的坐标，而是静态值和相对化坐标。

图像值是静态且不变的，因为它们表示背景纹理的大小。为了在渲染时更容易，`#init`​ 方法中预计算了两个额外的值（`leftPos`​ 和 `topPos`​），它们标记了背景渲染的左上角。标签坐标相对于这些值。

​`leftPos`​ 和 `topPos`​ 也用作渲染背景的便捷方式，因为它们已经表示传递给 `#blit`​ 方法的位置。

```java
// 在某个 AbstractContainerScreen 子类中
public MyContainerScreen(MyMenu menu, Inventory playerInventory, Component title) {
    super(menu, playerInventory, title);

    this.titleLabelX = 10;
    this.inventoryLabelX = 10;

    /*
     * 如果更改了 'imageHeight'，则还必须更改 'inventoryLabelY'，
     * 因为该值依赖于 'imageHeight' 值。
     */
}
```

---

#### 菜单访问（Menu Access）

由于菜单传递到屏幕中，因此可以通过 `menu`​ 字段访问菜单中的任何值（无论是通过槽位、数据槽位还是自定义系统同步的）。

---

#### 容器刻更新（Container Tick）

当玩家存活并查看屏幕时，容器屏幕在 `#tick`​ 方法中进行刻更新，通过 `#containerTick`​ 实现。这基本上取代了容器屏幕中的 `#tick`​，其最常见的用途是更新配方书。

```java
// 在某个 AbstractContainerScreen 子类中
@Override
protected void containerTick() {
    super.containerTick();

    // 在此处进行刻更新
}
```

---

#### 渲染容器屏幕（Rendering the Container Screen）

容器屏幕通过三个方法进行渲染：`#renderBg`​（渲染背景纹理）、`#renderLabels`​（渲染背景上的任何文本）和 `#render`​（包含前两个方法，并提供灰色背景和工具提示）。

从 `#render`​ 开始，最常见的重写（通常是唯一的情况）是添加背景，调用父类以渲染容器屏幕，最后在其上渲染工具提示。

```java
// 在某个 AbstractContainerScreen 子类中
@Override
public void render(GuiGraphics graphics, int mouseX, int mouseY, float partialTick) {
    this.renderBackground(graphics);
    super.render(graphics, mouseX, mouseY, partialTick);

    /*
     * 此方法由容器屏幕添加，用于渲染悬停槽位的工具提示。
     */
    this.renderTooltip(graphics, mouseX, mouseY);
}
```

在父类中，调用 `#renderBg`​ 以渲染屏幕的背景。最标准的表示使用三个方法调用：两个用于设置，一个用于绘制背景纹理。

```java
// 在某个 AbstractContainerScreen 子类中

// 背景纹理的位置（assets/<命名空间>/<路径>）
private static final ResourceLocation BACKGROUND_LOCATION = ResourceLocation.fromNamespaceAndPath(MOD_ID, "textures/gui/container/my_container_screen.png");

@Override
protected void renderBg(GuiGraphics graphics, float partialTick, int mouseX, int mouseY) {
    /*
     * 将背景纹理渲染到屏幕上。'leftPos' 和 'topPos' 应已表示纹理应渲染的左上角，
     * 因为它们是从 'imageWidth' 和 'imageHeight' 预计算的。两个零表示 256 x 256 PNG 文件中的整数 u/v 坐标。
     */
    graphics.blit(BACKGROUND_LOCATION, this.leftPos, this.topPos, 0, 0, this.imageWidth, this.imageHeight);
}
```

最后，调用 `#renderLabels`​ 以渲染背景上的任何文本，但在工具提示下方。这只需使用字体绘制关联的组件。

```java
// 在某个 AbstractContainerScreen 子类中
@Override
protected void renderLabels(GuiGraphics graphics, int mouseX, int mouseY) {
    super.renderLabels(graphics, mouseX, mouseY);

    // 假设我们有一些 Component 'label'
    // 'label' 在 'labelX' 和 'labelY' 处绘制
    graphics.drawString(this.font, this.label, this.labelX, this.labelY, 0x404040);
}
```

**注意**：渲染标签时，无需指定 `leftPos`​ 和 `topPos`​ 偏移量。这些已经在 `PoseStack`​ 中进行了变换，因此此方法中的所有内容都相对于这些坐标绘制。

---

#### 注册 AbstractContainerScreen（Registering an AbstractContainerScreen）

要将 `AbstractContainerScreen`​ 与菜单一起使用，需要注册它。可以通过在模组事件总线上调用 `RegisterMenuScreensEvent`​ 中的 `register`​ 来完成。

```java
// 事件在模组事件总线上监听
private void registerScreens(RegisterMenuScreensEvent event) {
    event.register(MY_MENU.get(), MyContainerScreen::new);
}
```
