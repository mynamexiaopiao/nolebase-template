# blockstates

### 方块状态（Blockstates）

在开发过程中，你可能会遇到需要为方块定义不同状态的情况。例如，小麦作物有八个生长阶段，如果为每个阶段都创建一个独立的方块显然是不合适的。或者，你有一个类似台阶的方块——一个底部状态、一个顶部状态，以及一个两者兼具的状态。

这时，**方块状态（Blockstates）** 就派上用场了。方块状态是一种简单的方式，用于表示方块可能具有的不同状态，比如生长阶段或台阶的放置类型。

---

### 方块状态属性（Blockstate Properties）

方块状态使用属性系统。一个方块可以拥有多个不同类型的属性。例如，末地传送门框架有两个属性：是否拥有末影之眼（`eye`，2种选项）以及它的朝向（`facing`，4种选项）。因此，末地传送门框架总共有8种（2 * 4）不同的方块状态：

```
minecraft:end_portal_frame[facing=north,eye=false]
minecraft:end_portal_frame[facing=east,eye=false]
minecraft:end_portal_frame[facing=south,eye=false]
minecraft:end_portal_frame[facing=west,eye=false]
minecraft:end_portal_frame[facing=north,eye=true]
minecraft:end_portal_frame[facing=east,eye=true]
minecraft:end_portal_frame[facing=south,eye=true]
minecraft:end_portal_frame[facing=west,eye=true]
```

这种`blockid[property1=value1,property2=value,...]`的表示方式是方块状态的标准文本形式，在原版游戏中的某些地方（例如命令）也会使用。

如果你的方块没有定义任何方块状态属性，它仍然有一个唯一的方块状态——即没有任何属性的状态，因为没有属性需要指定。这可以表示为`minecraft:oak_planks[]`或简写为`minecraft:oak_planks`。

与方块一样，每个`BlockState`在内存中只存在一个实例。这意味着可以使用`==`来比较`BlockState`，并且应该这样做。`BlockState`是一个`final`类，意味着它不能被扩展。任何功能都应放在对应的`Block`类中！

---

### 何时使用方块状态

#### 方块状态 vs. 独立方块

一个经验法则是：**如果它有不同的名称，它应该是一个独立的方块**。例如，制作椅子方块时，椅子的方向应该是一个属性，而不同类型的木材应该分成不同的方块。因此，每种木材类型都有一个椅子方块，每个椅子方块有四个方块状态（每个方向一个）。

#### 方块状态 vs. 方块实体

另一个经验法则是：**如果你有有限数量的状态，使用方块状态；如果你有无限或接近无限数量的状态，使用方块实体**。方块实体可以存储任意数量的数据，但比方块状态慢。

方块状态和方块实体可以结合使用。例如，箱子使用方块状态属性来表示方向、是否被水淹没或是否变成双箱，而存储物品、是否打开或与漏斗交互则由方块实体处理。

对于“方块状态可以有多少状态？”这个问题，没有明确的答案，但我们建议如果你需要超过8-9位的数据（即超过几百种状态），应该使用方块实体。

---

### 实现方块状态

要实现方块状态属性，首先在你的方块类中创建或引用一个`public static final Property<?>`常量。虽然你可以自由地创建自己的`Property<?>`实现，但原版代码提供了几种便捷的实现，涵盖了大多数用例：

- **​`IntegerProperty`​**
  实现`Property<Integer>`。定义一个保存整数值的属性。注意，不支持负值。
  通过调用`IntegerProperty#create(String propertyName, int minimum, int maximum)`创建。
- **​`BooleanProperty`​**
  实现`Property<Boolean>`。定义一个保存`true`或`false`值的属性。
  通过调用`BooleanProperty#create(String propertyName)`创建。
- **​`EnumProperty<E extends Enum<E>>`​** 
  实现`Property<E>`。定义一个可以接受枚举类值的属性。
  通过调用`EnumProperty#create(String propertyName, Class<E> enumClass)`创建。
  也可以只使用枚举值的子集（例如16种染料颜色中的4种），参见`EnumProperty#create`的重载。
- **​`DirectionProperty`​**
  继承`EnumProperty<Direction>`。定义一个可以接受`Direction`值的属性。
  通过调用`DirectionProperty#create(String propertyName)`创建。
  提供了几个便捷的谓词。例如，要获取表示基本方向的属性，调用`DirectionProperty.create("<name>", Direction.Plane.HORIZONTAL)`；要获取X轴方向，调用`DirectionProperty.create("<name>", Direction.Axis.X)`。

`BlockStateProperties`类包含了原版共享的属性，应尽可能使用或引用这些属性，而不是创建自己的属性。

一旦你有了属性常量，在你的方块类中重写`Block#createBlockStateDefinition(StateDefinition$Builder)`方法。在该方法中，调用`StateDefinition.Builder#add(YOUR_PROPERTY)`。`StateDefinition.Builder#add`是一个可变参数方法，因此如果你有多个属性，可以一次性添加它们。

每个方块还会有一个默认状态。如果没有特别指定，默认状态会使用每个属性的默认值。你可以通过在构造函数中调用`Block#registerDefaultState(BlockState)`方法来更改默认状态。

如果你希望在放置方块时更改使用的`BlockState`，重写`Block#getStateForPlacement(BlockPlaceContext)`方法。这可以用于根据玩家放置方块时的位置或视角来设置方块的方向。

以下是一个示例，展示了`EndPortalFrameBlock`类的相关部分：

```java
public class EndPortalFrameBlock extends Block {
    public static final DirectionProperty FACING = BlockStateProperties.FACING;
    public static final BooleanProperty EYE = BlockStateProperties.EYE;

    public EndPortalFrameBlock(BlockBehaviour.Properties pProperties) {
        super(pProperties);
        registerDefaultState(stateDefinition.any()
                .setValue(FACING, Direction.NORTH)
                .setValue(EYE, false)
        );
    }

    @Override
    protected void createBlockStateDefinition(StateDefinition.Builder<Block, BlockState> pBuilder) {
        pBuilder.add(FACING, EYE);
    }

    @Override
    @Nullable
    public BlockState getStateForPlacement(BlockPlaceContext pContext) {
        // 根据BlockPlaceContext确定放置方块时使用的状态
    }
}
```

---

### 使用方块状态

要从`Block`获取`BlockState`，调用`Block#defaultBlockState()`。默认方块状态可以通过`Block#registerDefaultState`更改，如上所述。

你可以通过调用`BlockState#getValue(Property<?>)`来获取属性的值，传入你想要获取值的属性。以末地传送门框架为例：

```java
Direction direction = endPortalFrameBlockState.getValue(EndPortalFrameBlock.FACING);
```

如果你想获取具有不同属性值的`BlockState`，只需在现有的方块状态上调用`BlockState#setValue(Property<T>, T)`。以末地传送门框架为例：

```java
endPortalFrameBlockState = endPortalFrameBlockState.setValue(EndPortalFrameBlock.FACING, Direction.SOUTH);
```

**注意**：`BlockState`是不可变的。这意味着当你调用`#setValue(Property<T>, T)`时，你并没有修改方块状态，而是在内部执行查找，并返回你请求的方块状态对象，这是唯一具有这些属性值的对象。因此，仅仅调用`state#setValue`而不将其保存到变量中（例如保存回`state`）是没有效果的。

要从世界中获取`BlockState`，使用`Level#getBlockState(BlockPos)`。

---

### 设置方块状态

要在世界中设置`BlockState`，使用`Level#setBlock(BlockPos, BlockState, int)`。

`int`参数需要额外解释，因为它的含义并不直观。它表示所谓的**更新标志（update flags）** 。

为了帮助正确设置更新标志，`Block`类中有一些以`UPDATE_`为前缀的`int`常量。这些常量可以通过按位或（`|`）组合（例如`Block.UPDATE_NEIGHBORS | Block.UPDATE_CLIENTS`）。

- `Block.UPDATE_NEIGHBORS`：向相邻方块发送更新。具体来说，它会调用`Block#neighborChanged`，进而调用一系列方法，其中大多数与红石相关。
- `Block.UPDATE_CLIENTS`：将方块更新同步到客户端。
- `Block.UPDATE_INVISIBLE`：明确不在客户端更新。这会覆盖`Block.UPDATE_CLIENTS`，导致更新不同步。方块始终在服务器端更新。
- `Block.UPDATE_IMMEDIATE`：强制在客户端的主线程上重新渲染。
- `Block.UPDATE_KNOWN_SHAPE`：停止邻居更新递归。
- `Block.UPDATE_SUPPRESS_DROPS`：禁用该位置旧方块的掉落物。
- `Block.UPDATE_MOVE_BY_PISTON`：仅由活塞代码使用，表示方块被活塞移动。这主要用于延迟光照引擎更新。
- `Block.UPDATE_ALL`：`Block.UPDATE_NEIGHBORS | Block.UPDATE_CLIENTS`的别名。
- `Block.UPDATE_ALL_IMMEDIATE`：`Block.UPDATE_NEIGHBORS | Block.UPDATE_CLIENTS | Block.UPDATE_IMMEDIATE`的别名。
- `Block.NONE`：`Block.UPDATE_INVISIBLE`的别名。

还有一个便捷方法`Level#setBlockAndUpdate(BlockPos pos, BlockState state)`，它内部调用`setBlock(pos, state, Block.UPDATE_ALL)`。
