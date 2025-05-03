# Mixin 文档

## 1. 准备工作和配置开发环境

Mixin 是一种在运行时修改已编译 Java 类的方法，允许你注入、修改或替换目标类的方法和字段，而不需要直接修改源代码。

**请在idea中下载Minecraft Development插件，他对Mixin开发有很大帮助！**

在neoforge中本身就集成了mixin,所以我们只需要开启就可以了

找的resources/META-INF/neoforge.mods.toml文件并打开找到

```java
# The [[mixins]] block allows you to declare your mixin config to FML so that it gets loaded.
#[[mixins]]
#config="${mod_id}.mixins.json"
```

把第二，三行的#号删除，就算开启成功了，很简单吧！

然后找到resources/[你的mod名].mixins.json，如果没有创建一个（或者改名）,打开后是这样(没有复制一样的)

```java
{
  "required": true,
  "package": "",
  "mixins": [
  ],
  "client": [
  ],
  "server": [
  ],
  "compatibilityLevel": "JAVA_21",
  "injectors": {
    "defaultRequire": 1
  }
}
```

在package填上你想让mixin文件放到的文件夹，比如我的是`com.example.examplemod.Mixin`​这个可以自己指定文件夹，指定了后mixin文件就都要放在那个文件夹中。当然，每个mixin类都要在这写上类名，比如我写了一个`ItemEntityMixin`​类，就要在文件中这么写

```java
  "mixins": [
    "ItemEntityMixin"
  ]
```

mixins,client,server分别代表双端，仅客户端，仅服务端，看你注入的类来填（插件可以自动帮你填）

‍

|注解类别|注解名称|功能描述|适用场景|
| ----------| ----------| --------------------------------| ------------------------------------------------|
|**核心注解**|​`@Mixin`​|声明当前类是目标类的混入类|所有Mixin类的必须注解|
||​`@Shadow`​|引用目标类中已存在的字段或方法|需要访问目标类私有成员时|
|**注入注解**|​`@Inject`​|在方法指定位置注入代码|方法执行前后插入逻辑（最常用）|
||​`@At`​|定义代码注入点的精确位置|需要精确定位指令时（必须配合其他注入注解使用）|
||​`@Redirect`​|完全重定向方法调用或字段访问|需要修改第三方库行为时|
||​`@ModifyArg`​|修改方法调用的指定参数|需要改变方法参数值时|
||​`@ModifyArgs`​|批量修改方法调用的多个参数|需要同时修改多个相关参数|
||​`@ModifyVariable`​|修改局部变量的值|需要拦截方法中间计算结果时|
||​`@ModifyConstant`​|修改方法中的常量值|需要调整硬编码数值时|
|**覆盖注解**|​`@Overwrite`​|完全覆盖原方法实现（慎用）|没有其他替代方案时的最后手段|
|**访问控制**|​`@Accessor`​|生成字段的getter/setter|需要访问私有字段但不想用反射时|
||​`@Invoker`​|调用私有方法|需要调用目标类私有方法时|
|**辅助注解**|​`@Unique`​|声明Mixin独有的字段/方法|避免与目标类或其他Mixin冲突|
||​`@Final`​|标记shadow字段为final|需要处理final字段时|
||​`@Mutable`​|允许修改final字段|需要修改目标类final字段时|
||​`@Implements`​|让目标类实现额外接口|需要为目标类添加接口支持时|
|**条件注解**|​`@Pseudo`​|标记目标类可能不存在|处理可选依赖的类时|
||​`@Dynamic`​|标记动态或原生方法|处理JNI方法或运行时生成的方法时|
|**调试注解**|​`@Debug`​|调试Mixin应用过程|开发阶段排查注入问题时|
|**配置注解**|​`@Group`​|定义注入点分组约束|需要确保多个注入点协同工作时|

## 2. 注解全解

### 2.1 核心注解

#### `@Mixin`​

```java
@Mixin(TargetClass.class)
public class MyMixin {
    // 混入内容
}
```

* 标识这是一个 Mixin 类
* 参数是目标类

所有Mixin类都必须要有的注解，括号中的参数为你要注入的类+.class，当你需要使用注入类的父类方法或字段时，你需要继承和注入类一样的父类。这时如果父类为接口或抽象类，你可以选择将自己的类变成抽象类来简化代码

#### `@Shadow`​

```java
@Shadow
private int someField;
@Shadow
public void someMethod() {} //可加上abstract

void a(){
	someMethod() //调用的是注入类的方法，并不是上面那个空方法体的方法
}
```

* 引用目标类中已有的字段或方法
* 必须与目标类中的签名完全匹配（Minecraft Development会自动帮你生成@Shadow注解）
* 可以用于字段、方法

你可能需要调用注入类的方法或使用其中的字段，这是一个可以让你获取父类的字段、方法的注解。你可以在获取的方法前加入abstract，这样可以不写方法体。即使你的方法的方法体是空的（如上面的someMethod()），当你在调用的时候也没关系，因为这其实只是个样子，真正调用的是你注入类的方法。

### 2.2 方法注入注解

#### ​`@Inject`​

```java
@Inject(method = "targetMethod", at = @At("HEAD"))
private void injectAtMethodHead(CallbackInfo ci) {
    // 在目标方法开头注入
}
```

* 在目标方法的特定位置注入代码
* 常用参数：

  * ​`method`​: 目标方法描述符
  * ​`at`​: 指定注入位置
  * ​`cancellable`​: 是否可取消（使用 `ci.cancel()`​）

这是Mixin中的核心注解，也是我们最常用的注解之一，通过它可以实现对我们注入类中的方法进行操作。其中，`method`​参数指定了你要注入的方法的名称（必须和注入的类中的方法一样）,`at`​指定了在哪个位置注入（详细如下表，不全，列举了主要的），`cancellable`​限定是否可以取消或者对方法返回值进行操作（下面会讲解）

|Value|描述|典型用途|
| -------| --------------------------| ----------------|
|​`HEAD`​|方法开头第一行|前置条件检查|
|​`RETURN`​|return语句前|结果处理|
|​`TAIL`​|方法最后一行（return后）|收尾清理|
|​`INVOKE`​|方法调用处|拦截特定调用|
|​`FIELD`​|字段访问处|监控字段读写|
|​`NEW`​|对象创建处|替换实例化逻辑|

##### 2.2.1 `cancellable`​

如果你的方法没有返回值，在`@Inject`​注入时开启了`cancellable`​，那么根据插件的提示，你会补全（或者在最后添加）`CallbackInfo`​这个参数

```java
@Inject(method = "updateSwimming", at = @At("HEAD"), cancellable = true)
private void onGetMaxHealth(CallbackInfo ci) {
    ci.cancel();//取消玩家游泳
}
```

这是一个控制返回的参数如上通过`ci.cancel()`​，可以在某些部分进行返回，类似`return`​，它不能设置返回值，只能取消。我们的例子是取消玩家游泳。

如果你的方法有返回值，那么则会补全`CallbackInfoReturnable<返回值> cir`​这个参数，它可以对返回值进行操作

```java
@Inject(method = "getTotalCookTime", at = @At("RETURN"),cancellable = true)
private static void getTotalCookTime(Level level, AbstractFurnaceBlockEntity blockEntity, CallbackInfoReturnable<Integer> cir) {
    SuperFurnaceBlockEntityMixin mixin = (SuperFurnaceBlockEntityMixin) (Object) blockEntity;
    if (mixin.isFast)  //这个参数原版没有
        cir.setReturnValue(cir.getReturnValueI()/4);
    }
}
```

如上这个例子（多方块视频中的），可以通过`getReturnValue()`​获取原本返回值,`setReturnValue(T value)`​设置原本返回值，当然也可以用`cir.cancel()`​来取消

##### 2.2.2 INVOKE详解

前面我们都是在方法的前面后面注入我们的逻辑，但如果我们想在特定的地方，如某个方法被调用后来注入我们的逻辑该怎么办呢，我们可以用INVOKE

```java
@Inject(method = "attack",at = @At(value = "INVOKE",
		target = "Lnet/minecraft/world/entity/player/Player;getAttackStrengthScale(F)F",),
        cancellable = true)
private void attack(Entity target, CallbackInfo ci) {

}
```

这就是INVOKE，可以看见突然多了很大一串`"Lnet/minecraft/world/entity/player/Player;getAttackStrengthScale(F)F"`​这是干什么的呢？

这其实是域描述符，它指定了注入的位置，我们的代码会在这个位置下面运行。

有了插件的帮助，其实只要输入名称就会自动帮你补全，下面可供了解（来自[Mixin注入 [Fabric Wiki]](https://wiki.fabricmc.net/zh_cn:tutorial:mixin_injects)）

|描述符|原名|描述|
| ----------| -----------| --------------------------------------------------------------------|
|B|byte|带符号的字节|
|C|char|Basic Multilingual Plane 中的 Unicode 字符代码点，使用 UTF-16 编码|
|D|double|双精度浮点值|
|F|float|单精度浮点值|
|I|int|整型|
|J|long|长整型|
|L类名称;|reference|类名称的实例|
|S|short|带符号的短整型|
|Z|boolean|true 或 false|
|[|reference|单数组维度|

方法描述符包括方法名称，接着一系列包含输入类型的括号，以及输出类型。Java 中定义的像 `Object m(int i, double[] d, Thread t)`​ 这样的方法会有 `m(I[DLjava/lang/Thread;)Ljava/lang/Object;`​ 这样的方法描述符。

在这个返回类型为 void 的例子中，你需要使用 V（空描述符）作为，例如，`void foo(String bar)`​ 就会是 `foo(Ljava/lang/String;)V`​。

泛型将会移除，因为泛型在运行的时候不存在，因为像 `Pair<Integer, ? extends Task<? super VillagerEntity>‍>`​ 这样的就会变成 `Lcom/mojang/datafixers/util/Pair`​。

##### 2.2.3构造函数和静态初始化块注入

如果遇到要注入构造函数和静态初始化块的情况，可以用一下方法

```java
@Inject(method = "<init>", at = @At("TAIL")) // 构造函数
@Inject(method = "<clinit>", at = @At("TAIL")) // 静态初始化块
```

##### 2.2.4注入偏移

默认情况我们的注入都在下面，如果我们想在上面指定几行来注入，可以使用`shift`​和`by`​

```java
@At(
	method = ""
    target = "目标描述符",  // 部分类型需要
    shift = At.Shift.XXX,  // 位置偏移
    by = 偏移量,           // 配合shift=BY使用
)
```

以下是shift可选位置偏移

|位置偏移|作用|
| ----------| ------------------|
|​`NONE`​|不返回|
|​`BEFORE`​|向后移动一条指令|
|​`AFTER`​|向前移动一条指令|
|​`BY`​|通过by参数移动|

当你使用`BY`​的时候，向下移用正数，向上移用负数，`by`​的值不应小于-3或大于3，否则可能会出现未知问题。

##### 2.2.5抓取方法内的局部变量

如果想获取方法内的局部变量，可以在`@Inject`​中加入locals来获取

```java
@Inject(method = "targetMethod",at = ....,
        locals = LocalCapture.CAPTURE_FAILHARD // 强制捕获局部变量
    )
private void captureLocals(CallbackInfo ci, 
                             int var1, 
                             String var2, 
                             ItemStack var3) {
        // 变量顺序必须与原方法完全一致
    System.out.println("捕获到变量: " + var2);
}
```

正如上面的代码，方法内的参数会变成这个其中的变量，它还有几种模式可以选择

|​`LocalCapture`​ 类型|行为|
| -----------| ------------------------------------|
|​`CAPTURE_FAILHARD`​|严格匹配，失败则报错（推荐调试用）|
|​`CAPTURE_FAILSOFT`​|失败时静默跳过（生产环境适用）|
|​`PRINT`​|失败时打印警告|

##### 2.2.6高级参数

高级参数的用法全部翻译自代码中的文档

|参数|作用|值|值作用解释|
| ------| ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------| ------| --------------------------------------------------------|
|​**​`remap`​**​|默认情况下，注解处理器将尝试为所有 Inject 方法定位混淆映射，因为一般来说，Inject 注解的目标将是目标类中的混淆方法。然而，由于也可以将混入应用于非混淆目标（或混淆目标中的非混淆方法，例如 Forge 添加的方法），因此可能需要抑制否则会生成的编译器错误。将此值设置为 false 将导致注解处理器在尝试为混入构建混淆表时跳过此注解。返回值：True 表示指示注解处理器搜索此注解的混淆映射。|​`Boolean`​|将注释处理器设置为“true”，以便搜索此注释的混淆映射。|
|​`id`​|标识符可以通过 `CallbackInfo.getId`​ 访问器获取。如果未指定，ID 默认为目标方法名称。|​`String`​|使用的注入器ID|
|​`require`​|一般来说，注入器旨在“软失败”，即未能在目标方法中找到注入点不被视为错误条件。另一个转换器可能已更改方法结构，或由于任何原因可能导致注入失败。这也使得可以定义多个注入以在目标类预期变异的情况下实现相同的任务，而失败的注入器则被简单忽略。然而，这种行为并不总是理想的。例如，如果您的应用程序依赖于特定注入的成功，您可能希望将注入失败检测为错误条件。因此，提供此参数以允许您规定此回调处理程序所需的最小成功注入次数。如果未达到指定的注入次数，则在应用程序运行时抛出InjectionError。谨慎使用此选项。返回值：<br />所需的最小注入回调次数，默认由包含的配置指定。|​`int`​|最小 预期 回调数，默认 1|
|​`allow`​|注入点通常预期会匹配目标方法或代码片段中的每条候选指令，除非指定了诸如At. ordinal等选项，这些选项会自然限制匹配结果的数量。<br />此选项允许通过指定最大允许匹配数对注入点结果进行合理性检查，类似于Group. max提供的功能。例如，若您的注入预期匹配目标方法的4次调用，但实际上匹配了5次，通过将此值设为4，即可检测到这种篡改情况。<br />允许设置任何大于等于1的值。小于1或小于require的值将被忽略。require优先级高于此参数，因此若allow小于require，则始终使用require的值。<br />请注意，此选项并非对此注入点查询行为的限制，仅作为确保匹配数量不过高的合理性检查。<br /><br />|​`int`​|此注入点允许的最大注入次数|
|​`constraints`​|返回必须验证的约束，以便此注入器成功。有关约束格式的详细信息，请参见 org.spongepowered.asm.util.ConstraintParser.Constraint。|​`String`​|此注解的约束。|
|​`order`​|默认情况下，几乎所有针对目标类的注入器都会同时应用它们的注入。换句话说，如果多个混入类（mixin）针对同一个类，那么注入器会按照优先级顺序应用（因为混入类本身是按照优先级顺序合并的，注入器按照合并的顺序运行）。例外的是重定向注入器，它们会在稍后的阶段应用。注入器的默认顺序是1000，而重定向注入器使用10000。指定一个顺序值会改变这种默认行为，使注入器比通常情况更早或更晚注入。例如，指定900会使注入器在其他注入器之前应用，而1100会在之后应用。具有相同顺序的注入器仍会按照其混入类的优先级顺序应用。|​`int`​|此注入器的应用顺序，如果未指定则使用默认值（1000）|

#### `@Redirect`​

```java
@Redirect(
    method = "目标方法",
    at = @At(
        value = "INVOKE",
        target = "目标描述符"
    ),
    require = 1 // 推荐设置以确保注入
)
[返回类型] 自定义方法名([参数列表]) {
    // 新逻辑
}
```

* 重定向方法，字段或者构造器

##### 1.方法调用重定向

```java
@Redirect(
    method = "目标方法",
    at = @At(
        value = "INVOKE",
        target = "目标方法描述符"
    )
)
private 返回类型 自定义方法名(原方法参数) {
    // 新逻辑
}
```

可能不好理解如何进行重定向

假如有这样一个方法

```java
void x(){
	a(1)	
}
```

我们进行重定向

```java
@Redirect(method = "x", at = @At(value = "INVOKE", target = "......"))
private int b(int x) {
    // 自己的代码
}
```

那么他就变成这样了

```java
void x(){
	b(1)	
}
```

其他的也是同理

##### 2. 字段访问重定向

```java
@Redirect(
    method = "目标方法",
    at = @At(
        value = "FIELD",
        target = "目标字段描述符",
        opcode = Opcodes.GETFIELD 或 Opcodes.PUTFIELD
    )
)
private 字段类型 自定义方法名(可选实例参数) {
    // 新逻辑
}
```

‍

##### 3. 构造方法重定向

```java
@Redirect(
    method = "目标方法",
    at = @At(
        value = "NEW",
        target = "目标类全限定名"
    )
)
private 目标类型 自定义方法名(构造方法参数) {
    // 返回替代实例
}
```

‍

#### `@ModifyArg`​

```java
@ModifyArg(method = "targetMethod", at = @At(value = "INVOKE", target = "Lnet/minecraft/...;someMethod(I)V"), index = 0)
private int modifyArg(int original) {
    return original + 1; // 修改参数值
}
```

* 修改方法调用的参数
* ​`index`​ 指定要修改的参数索引

在代码中，会经常用到方法调用，如果你想该方法调用的值，就需要用到这个，下面举个例子

```java
@ModifyArg(
        method = "attack", // 目标方法
        at = @At(
                value = "INVOKE",
                // 定位到计算基础伤害的方法调用
                target = "Lnet/minecraft/world/entity/player/Player;getAttackStrengthScale(F)F"
        ),
        index = 0 // 修改第一个参数（也是唯一参数）
)
private float modifyBaseDamage(float originalDamage) {
    Player player = (Player)(Object)this;
    ItemStack weapon = player.getMainHandItem();

    // 如果是钻石剑则提升伤害
    if (weapon.is(Items.DIAMOND_SWORD)) {
        return originalDamage * 10f;
    }
    return originalDamage;
}
```

在`@ModifyArg`​中，我们基本上用INVOKE，其中index参数标记你该的是第几个参数（从0开始计）使用`@ModifyArg`​注解方法就会有返回值，返回值和你要修改的值一样，经过操作修改后用`return`​返回就修改成功了。

#### `@ModifyArgs`​

```java
@ModifyArgs(method = "targetMethod", at = @At(value = "INVOKE", target = "Lnet/minecraft/...;someMethod(II)V"))
private void modifyArgs(Args args) {
    args.set(0, (int)args.get(0) + 1); // 修改第一个参数
    args.set(1, (int)args.get(1) * 2); // 修改第二个参数
}
```

* 同时修改多个参数
* 通过 `Args`​ 对象操作

上面介绍了`@ModifyArg`​，当你需要修改方法中多个参数时，你就需要`@ModifyArgs`​，它可以一次性操作多个参数，我们方法会有`args`​参数，通过它的`.set`​来修改，因此它没有`index`​参数

#### `@ModifyVariable`​

```java
@ModifyVariable(method = "targetMethod", at = @At("STORE"), ordinal = 0)
private int modifyLocalVariable(int original) {
    return original * 2; // 修改变量值
}
```

* 修改局部变量
* ​`ordinal`​ 指定变量的序号（同名变量时）

这个注解是用来捕获并修改方法局部变量的，其中`ordinal`​表示同名变量的出现序号（从0开始），这可能不好理解，下面举个例子

```java
public void attack(Entity target) {
    float damage = getBaseDamage(); // <- 第一个damage（ordinal=0）
    damage += getEnchantBonus();    // 修改值后重新存储
    
    if (isCritical()) {
        float damage = damage * 1.5f; // <- 第二个damage（ordinal=1）
        showCriticalParticles();
    }
    target.hurt(damage);
}
```

相信这样就好理解了，那它是如何进行修改的呢，再举一个例子

```java
@Mixin(Player.class)
public abstract class PlayerHungerMixin {
    
    @ModifyVariable(
        method = "addExhaustion", 
        at = @At("STORE"), // 捕获存储到局部变量的操作
        ordinal = 0
    )
    private float modifyExhaustion(float original) {
        // 如果玩家有特殊效果，减少饥饿消耗
        if (((Player)(Object)this).hasEffect(MobEffects.SATURATION)) {
            return original * 0.3f;
        }
        return original;
    }
}
```

```java
// 原方法：
float exhaustion = amount; // ← 被捕获的STORE操作
this.foodData.addExhaustion(exhaustion);

// 修改后：
float exhaustion = amount * 0.3f;
this.foodData.addExhaustion(exhaustion);
```

这其实和@Injec+locals的效果一样

#### ​`@ModifyConstant`​

```java
@ModifyConstant(
    method = "目标方法",
    constant = @Constant(
        // 常量类型选择（任选其一）:
        intValue = 10,          // 匹配int
        floatValue = 0.5F,      // 匹配float  
        doubleValue = 1.0,      // 匹配double
        longValue = 100L,       // 匹配long
        stringValue = "text",   // 匹配字符串
        classValue = String.class, // 匹配类字面量
        nullValue = true        // 匹配null
    ),
    ordinal = 0,    // 同类型常量的序号（可选）
    expandZeroConditions = true // 扩展0值匹配（可选）
)
private [返回类型] 方法名([原类型] original) {
    return modifiedValue; // 返回修改后的值
}
```

* 修改方法中的常量值
* 支持 int, float, long, double, String, class 常量

​`@ModifyConstant`​ 是 Mixin 中用于修改方法内常量值的专用注解，其中不同常量的值取决于注入方法中的值

### 2.3 类修改注解

#### `@Overwrite`​

```java
@Overwrite
public void targetMethod() {
    // 完全覆盖原方法
}
```

* **慎用**：完全替换原方法，可能与其他模组冲突
* 仅在没有其他替代方案时使用

正如提示的一样，这个可以重写类中的方法，但请注意，非必要请不要使用！！！会导致mod的兼容性变差

#### `@Accessor`​和`@Invoker`​

```java
@Accessor("fieldName")
Type getFieldName();
```

* 生成字段的 getter/setter
* 用于访问私有字段

```java
@Invoker("methodName")
void invokeMethodName(Args... args);
```

* 调用私有方法
* 方法签名必须匹配

​`@Accessor`​可以建立一个接口通过set和get方法获取改变类中私有变量的值,有人可能会说，`@Accessor`​这个有啥用，`@Shadow`​也可以访问私有的字段，en..的确如此,但它有这几个优点()

|特性|​`@Shadow`​|​`@Accessor`​|
| ------| -------------------------| ---------------------------|
|**作用域**|仅在当前Mixin类内部可用|生成公共方法供外部调用|
|**代码生成**|直接引用目标字段/方法|自动生成getter/setter方法|
|**使用场景**|Mixin内部实现逻辑|提供给其他类使用的API|

​`@Invoker`​可以访问私有方法

#### `@Mutable`​

```java
@Mutable
@Shadow private static final SomeType FIELD = null;
```

* 允许修改 final 字段

### 2.4 辅助注解

#### `@Final`​

```java
@Final
@Shadow private int field;
```

* 表示 shadow 字段是 final 的

#### `@Unique`​

```java
@Unique
private int myUniqueField;
```

* 添加 Mixin 类独有的字段/方法
* 避免与目标类或其他 Mixin 冲突

#### `@Implements`​

```java
@Implements({
    @Interface(
        iface = 目标接口.class,
        prefix = "自定义前缀$" // 可选
    )
})
@Mixin(目标类.class)
public abstract class MyMixin {
    // 必须实现接口要求的方法
    @Unique
    public void 自定义前缀$接口方法() {
        // 实现逻辑
    }
}
```

* 让目标类实现额外接口

### 2.5 条件注解

#### `@Pseudo`​

```java
@Pseudo
@Mixin(SomeClass.class)
public class MyMixin {}
```

* 用于混入可能不存在的类

翻译自文档：标记为 @Pseudo 的 Mixin 可以针对在 编译时 不可用且在运行时可能不可用的类。这可用于以下情况 - 由于某种原因 - 目标类对正在编译的项目不可用，因此无法通过 AP 验证。目标类由 AP 仅使用目标的超类（或至少使用目标的任何 可用 已知超类）的知识进行模拟。这意味着存在某些限制：
如果目标的层次结构中有一个混淆的类，那么伪 mixin 的超类要求就非常重要。例如，假设我们将来自扩展 GuiScreen 的另一方的 CustomGuiScreen 混入到一个类中，其中 GuiScreen 是一个经过混淆处理的类。GuiScreen 包含一个经过混淆处理的方法 initGui，（@Pseudo） 目标类将覆盖该方法。尝试注入 initGui 会在开发时成功，但在生产时会失败，因为引用被混淆了。通常，当目标以这种方式覆盖混淆方法时，AP 可以通过遍历目标的超类层次结构来解决混淆问题，以便发现映射。但是，当目标在编译时不可用时，AP 无法执行此作，并且必须仅依赖来自 mixin 本身的信息。我们可以通过确保 mixin 继承自同一个超类（或者至少是包含 mixin 中使用的混淆方法或字段的超类）来克服这个问题，从而允许我们的示例 initGui 方法在超类层次结构中解析。此行为不适用于普通 mixin，因为 AP 始终通过目标类元数据解析层次结构（当它可用时）。
如果目标类包含混入需要的Overwrite混淆方法，或者Shadow不是从超类继承的混淆方法（例如，目标被混淆），Overwrite则必须使用别名手动修饰 orShadow，因为 AP 没有自动解析映射的机制。

#### `@Dynamic`​

```java
@Dynamic
@Shadow private native void method();
```

* 用于动态或原生方法

翻译自文档:

`@Dynamic`​ 注解用于修饰那些目标在原始类中不可用，并且在运行时被动态创建或转换的混入（mixin）元素。该注解纯粹用于装饰目的，没有任何语义上的含义。

在混入元素上使用该注解是鼓励的，因为它带来了以下好处：

1. **为混入维护者提供上下文**：对于那些看似无意义的注入，可以通过使用该注解为其提供上下文。例如，一个目标似乎不存在的注入或覆盖可以通过使用该注解来提供解释。
2. **允许混入工具（如IDE插件）识别特定目标**：在静态分析中，工具可能会将目标标记为无效，而该注解可以提供上下文来抑制或增强生成的警告。
3. **在注入失败时提供额外上下文**：例如，如果一个带有 `@Dynamic`​ 注解的注入器未能找到有效目标，混入框架会将 `@Dynamic`​ 注解的内容包含在错误信息中，为最终用户提供额外的调试信息。这可以用于识别上游转换是否被禁用或更改。

由于该注解可能被包含在错误信息中，建议通过 `@Dynamic`​ 提供的描述尽可能简洁地提供尽可能多的信息。良好的描述示例可能包括：

* "方法 `Foo.bar`​ 在运行时由模组 X 添加"
* "所有对 `Foo.bar`​ 的调用在运行时由模组 Y 替换为对 `Baz.flop`​ 的调用"
* "由 `SomeOtherUpstreamMixin`​ 添加"

换句话说，`@Dynamic`​ 的内容应尽量为修饰的方法提供有用的上下文，而不过于冗长。

如果目标方法或字段是由上游混入添加的，并且该混入在项目的类路径中，还可以使用 `mixin`​ 值来指定贡献目标成员的混入。这为IDE提供了可导航的链接，并为混入工具（如IDE插件）提供了有用的上下文。

如果目标成员是由上游混入贡献的，但该混入不在当前项目的类路径中，使用上游混入的完全限定名作为值是一个有用的后备方案，因为开发者在阅读源代码时仍然可以在他们的IDE或Github上使用该字符串值进行搜索。

## 3.Mixin 小提示

### 访问this

当你需要访问this时，不妨这么做

​`((TargetClass)(Object)this)`​

