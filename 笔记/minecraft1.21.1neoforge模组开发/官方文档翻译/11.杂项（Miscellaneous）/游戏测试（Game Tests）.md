# 游戏测试（Game Tests）

### 游戏测试（Game Tests）

游戏测试是一种在游戏内运行单元测试的方式。该系统设计为可扩展且并行运行，能够高效地执行大量不同的测试。测试对象交互和行为是该框架的众多应用之一。

---

#### 创建游戏测试

标准的游戏测试遵循三个基本步骤：

1. 加载一个结构或模板，该模板包含用于测试交互或行为的场景。
2. 编写一个方法来执行场景中的逻辑。
3. 执行方法逻辑。如果达到成功状态，则测试成功；否则，测试失败，结果存储在场景附近的讲台（lectern）中。

因此，要创建游戏测试，必须有一个包含场景初始状态的模板，以及一个提供执行逻辑的方法。

---

#### 测试方法

游戏测试方法是一个 `Consumer<GameTestHelper>`​ 引用，这意味着它接收一个 `GameTestHelper`​ 并返回空。为了使游戏测试方法被识别，必须使用 `@GameTest`​ 注解：

```java
public class ExampleGameTests {
    @GameTest
    public static void exampleTest(GameTestHelper helper) {
        // 执行逻辑
    }
}
```

​`@GameTest`​ 注解还包含配置游戏测试运行方式的成员：

```java
@GameTest(
    setupTicks = 20L, // 测试花费 20 Tick 进行设置
    required = false  // 失败会被记录，但不会影响批处理的执行
)
public static void exampleConfiguredTest(GameTestHelper helper) {
    // 执行逻辑
}
```

---

#### 相对定位

所有 `GameTestHelper`​ 方法都将结构模板场景中的相对坐标转换为绝对坐标。为了简化相对坐标和绝对坐标之间的转换，可以使用 `GameTestHelper#absolutePos`​ 和 `GameTestHelper#relativePos`​。

在游戏中，可以通过 `/test`​ 命令加载结构模板，将玩家放置在所需位置，然后运行 `/test pos`​ 命令来获取结构的相对坐标。该命令会将相对坐标作为可复制的文本组件输出到聊天中。

**提示**：
可以通过在命令末尾附加变量名来指定引用名称：

```plaintext
/test pos <var> # 输出 'final BlockPos <var> = new BlockPos(...);'
```

---

#### 成功完成

游戏测试方法负责一件事：在有效完成时标记测试成功。如果在超时（由 `GameTest#timeoutTicks`​ 定义）之前未达到成功状态，则测试自动失败。

​`GameTestHelper`​ 中有许多抽象方法可用于定义成功状态，但以下四个方法尤为重要：

|方法|描述|
| ------| ----------------------------------------------------------------------------------------------|
|​`#succeed`​|标记测试成功。|
|​`#succeedIf`​|立即测试提供的 `Runnable`​，如果不抛出 `GameTestAssertException`​，则成功。如果测试未在立即的 Tick 中成功，则标记为失败。|
|​`#succeedWhen`​|每 Tick 测试提供的 `Runnable`​，直到超时。如果某个 Tick 的检查未抛出 `GameTestAssertException`​，则成功。|
|​`#succeedOnTickWhen`​|在指定的 Tick 上测试提供的 `Runnable`​，如果不抛出 `GameTestAssertException`​，则成功。如果在其他 Tick 上成功，则标记为失败。|

**注意**：
游戏测试每 Tick 执行，直到测试标记为成功。因此，在指定 Tick 上调度成功的方法必须确保在之前的 Tick 上始终失败。

---

#### 调度操作

并非所有操作都会在测试开始时执行。可以调度操作在特定时间或间隔执行：

|方法|描述|
| ------| --------------------------------------|
|​`#runAtTickTime`​|在指定的 Tick 上执行操作。|
|​`#runAfterDelay`​|在当前 Tick 后延迟 x Tick 执行操作。|
|​`#onEachTick`​|每 Tick 执行操作。|

---

#### 断言

在游戏测试的任何时候，都可以进行断言以检查给定条件是否为真。`GameTestHelper`​ 中有许多断言方法，但核心思想是在适当状态未满足时抛出 `GameTestAssertException`​。

---

#### 生成的测试方法

如果需要动态生成游戏测试方法，可以创建测试方法生成器。这些方法不接收参数并返回 `TestFunction`​ 集合。为了使测试方法生成器被识别，必须使用 `@GameTestGenerator`​ 注解：

```java
public class ExampleGameTests {
    @GameTestGenerator
    public static Collection<TestFunction> exampleTests() {
        // 返回 TestFunction 集合
    }
}
```

​`TestFunction`​ 是 `@GameTest`​ 注解和运行测试的方法所包含的装箱信息。

**提示**：
任何使用 `@GameTest`​ 注解的方法都会通过 `GameTestRegistry#turnMethodIntoTestFunction`​ 转换为 `TestFunction`​。该方法可用作创建 `TestFunction`​ 的参考，而无需使用注解。

---

#### 批处理

游戏测试可以按批处理执行，而不是按注册顺序执行。通过为 `GameTest#batch`​ 提供相同的字符串，可以将测试添加到批处理中。

批处理本身并不提供任何有用的功能。但是，批处理可用于在当前测试运行的级别上执行设置和拆卸状态。这是通过使用 `@BeforeBatch`​ 进行设置或使用 `@AfterBatch`​ 进行拆卸来实现的。`#batch`​ 方法必须与游戏测试提供的字符串匹配。

批处理方法是 `Consumer<ServerLevel>`​ 引用，这意味着它们接收 `ServerLevel`​ 并返回空：

```java
public class ExampleGameTests {
    @BeforeBatch(batch = "firstBatch")
    public static void beforeTest(ServerLevel level) {
        // 执行设置
    }

    @GameTest(batch = "firstBatch")
    public static void exampleTest2(GameTestHelper helper) {
        // 执行逻辑
    }
}
```

---

#### 注册游戏测试

游戏测试必须注册才能在游戏中运行。有两种方法可以做到这一点：通过 `@GameTestHolder`​ 注解或 `RegisterGameTestsEvent`​。两种注册方法仍然要求测试方法使用 `@GameTest`​、`@GameTestGenerator`​、`@BeforeBatch`​ 或 `@AfterBatch`​ 注解。

1. **GameTestHolder**
    `@GameTestHolder`​ 注解注册类型（类、接口、枚举或记录）内的所有测试方法。`@GameTestHolder`​ 包含一个方法，其值必须是模组的 ID，否则测试将不会在默认配置下运行。

    ```java
    @GameTestHolder(MODID)
    public class ExampleGameTests {
        // ...
    }
    ```
2. **RegisterGameTestsEvent**
    `RegisterGameTestsEvent`​ 也可以使用 `#register`​ 注册类或方法。事件监听器必须添加到模组事件总线。以这种方式注册的测试方法必须在每个使用 `@GameTest`​ 注解的方法上提供模组 ID 到 `GameTest#templateNamespace`​。

    ```java
    public void registerTests(RegisterGameTestsEvent event) {
        event.register(ExampleGameTests.class);
    }

    @GameTest(templateNamespace = MODID)
    public static void exampleTest3(GameTestHelper helper) {
        // 执行设置
    }
    ```

---

#### 结构模板

游戏测试在由结构或模板加载的场景中执行。所有模板定义场景的尺寸和将加载的初始数据（方块和实体）。模板必须存储为 `data/<namespace>/structures`​ 中的 `.nbt`​ 文件。

**提示**：
可以使用结构块创建和保存结构模板。

模板的位置由以下几个因素决定：

1. 如果指定了模板的命名空间。
2. 如果类名应附加到模板名称之前。
3. 如果指定了模板的名称。

---

#### 运行游戏测试

可以使用 `/test`​ 命令运行游戏测试。`/test`​ 命令高度可配置，但只有少数几个对运行测试很重要：

|子命令|描述|
| --------| --------------------------------------|
|​`run`​|运行指定的测试：`run <test_name>`​。|
|​`runall`​|运行所有可用的测试。|
|​`runclosest`​|运行距离玩家 15 个方块内的最近测试。|
|​`runthese`​|运行玩家 200 个方块内的测试。|
|​`runfailed`​|运行上次运行中失败的所有测试。|

**注意**：
子命令跟在 `/test`​ 命令后面：`/test <subcommand>`​。

---

#### 构建脚本配置

游戏测试在构建脚本（`build.gradle`​ 文件）中提供了额外的配置设置，以运行并集成到不同的设置中。

1. **启用其他命名空间**
    如果构建脚本按推荐设置，则仅启用当前模组 ID 下的游戏测试。要启用其他命名空间以加载游戏测试，运行配置必须将属性 `neoforge.enabledGameTestNamespaces`​ 设置为以逗号分隔的每个命名空间的字符串。如果属性为空或未设置，则将加载所有命名空间。

    ```groovy
    property 'neoforge.enabledGameTestNamespaces', 'modid1,modid2,modid3'
    ```
    **注意**：
    命名空间之间不能有空格，否则命名空间将无法正确加载。
2. **游戏测试服务器运行配置**
    游戏测试服务器是一种特殊的配置，用于运行构建服务器。构建服务器返回所需失败游戏测试的退出代码。所有失败的测试（无论是必需的还是可选的）都会被记录。可以使用 `gradlew runGameTestServer`​ 运行此服务器。
3. **在其他运行配置中启用游戏测试**
    默认情况下，只有客户端、服务器和 `gameTestServer`​ 运行配置启用了游戏测试。如果其他运行配置应运行游戏测试，则必须将 `neoforge.enableGameTest`​ 属性设置为 `true`​。

    ```groovy
    property 'neoforge.enableGameTest', 'true'
    ```
