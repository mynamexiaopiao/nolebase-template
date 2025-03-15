# 自定义战利品对象(Custom Loot Entry Types)

### 自定义战利品对象（Custom Loot Entry Types）

由于战利品表系统的复杂性，系统中有多个注册表，模组开发者可以利用这些注册表来添加更多行为。

所有与战利品表相关的注册表都遵循类似的模式。要添加一个新的注册表条目，通常需要扩展某个类或实现某个接口来定义功能。然后，定义一个用于序列化的编解码器（codec），并使用 `DeferredRegister`​ 将其注册到相应的注册表中。这与大多数注册表（例如方块/方块状态和物品/物品堆栈）使用的“一个基础对象，多个实例”方法一致。

---

### 自定义战利品条目类型

要创建自定义战利品条目类型，可以扩展 `LootPoolEntryContainer`​ 或其两个直接子类之一：`LootPoolSingletonContainer`​ 或 `CompositeEntryBase`​。以下是一个示例，创建一个返回实体掉落物的战利品条目类型（仅为示例，实际应用中更推荐直接引用其他战利品表）：

```java
// 我们扩展 LootPoolSingletonContainer，因为我们有一组“有限”的掉落物。
public class EntityLootEntry extends LootPoolSingletonContainer {
    private final Holder<EntityType<?>> entity;

    private EntityLootEntry(Holder<EntityType<?>> entity, int weight, int quality, List<LootItemCondition> conditions, List<LootItemFunction> functions) {
        super(weight, quality, conditions, functions);
        this.entity = entity;
    }

    public static LootPoolSingletonContainer.Builder<?> entityLoot(Holder<EntityType<?>> entity) {
        return simpleBuilder((weight, quality, conditions, functions) -> new EntityLootEntry(entity, weight, quality, conditions, functions));
    }

    @Override
    public void createItemStack(Consumer<ItemStack> consumer, LootContext context) {
        LootTable table = context.getLevel().reloadableRegistries().getLootTable(entity.value().getDefaultLootTable());
        table.getRandomItemsRaw(context, consumer);
    }
}
```

接下来，我们为战利品条目创建一个 `MapCodec`​：

```java
public static final MapCodec<EntityLootEntry> CODEC = RecordCodecBuilder.mapCodec(inst ->
    inst.group(
        BuiltInRegistries.ENTITY_TYPE.holderByNameCodec().fieldOf("entity").forGetter(e -> e.entity)
    )
    .and(singletonFields(inst))
    .apply(inst, EntityLootEntry::new)
);
```

然后，使用此编解码器进行注册：

```java
public static final DeferredRegister<LootPoolEntryType> LOOT_POOL_ENTRY_TYPES =
    DeferredRegister.create(Registries.LOOT_POOL_ENTRY_TYPE, ExampleMod.MOD_ID);

public static final Supplier<LootPoolEntryType> ENTITY_LOOT =
    LOOT_POOL_ENTRY_TYPES.register("entity_loot", () -> new LootPoolEntryType(EntityLootEntry.CODEC));
```

最后，在战利品条目类中重写 `getType()`​：

```java
@Override
public LootPoolEntryType getType() {
    return ENTITY_LOOT.get();
}
```

---

### 自定义数字提供器

要创建自定义数字提供器，可以实现 `NumberProvider`​ 接口。以下是一个示例，创建一个反转数字符号的数字提供器：

```java
public record InvertedSignProvider(NumberProvider base) implements NumberProvider {
    public static final MapCodec<InvertedSignProvider> CODEC = RecordCodecBuilder.mapCodec(inst -> inst.group(
        NumberProviders.CODEC.fieldOf("base").forGetter(InvertedSignProvider::base)
    ).apply(inst, InvertedSignProvider::new));

    @Override
    public float getFloat(LootContext context) {
        return -this.base.getFloat(context);
    }

    @Override
    public int getInt(LootContext context) {
        return -this.base.getInt(context);
    }

    @Override
    public Set<LootContextParam<?>> getReferencedContextParams() {
        return this.base.getReferencedContextParams();
    }
}
```

注册数字提供器：

```java
public static final DeferredRegister<LootNumberProviderType> LOOT_NUMBER_PROVIDER_TYPES =
    DeferredRegister.create(Registries.LOOT_NUMBER_PROVIDER_TYPE, ExampleMod.MOD_ID);

public static final Supplier<LootNumberProviderType> INVERTED_SIGN =
    LOOT_NUMBER_PROVIDER_TYPES.register("inverted_sign", () -> new LootNumberProviderType(InvertedSignProvider.CODEC));
```

在数字提供器类中重写 `getType()`​：

```java
@Override
public LootNumberProviderType getType() {
    return INVERTED_SIGN.get();
}
```

---

### 自定义基于等级的值

可以通过实现 `LevelBasedValue`​ 接口来创建自定义基于等级的值。以下是一个示例，创建一个反转另一个 `LevelBasedValue`​ 输出的值：

```java
public record InvertedSignLevelBasedValue(LevelBasedValue base) implements LevelBasedValue {
    public static final MapCodec<InvertedSignLevelBasedValue> CODEC = RecordCodecBuilder.mapCodec(inst -> inst.group(
        LevelBasedValue.CODEC.fieldOf("base").forGetter(InvertedSignLevelBasedValue::base)
    ).apply(inst, InvertedSignLevelBasedValue::new));

    @Override
    public float calculate(int level) {
        return -this.base.calculate(level);
    }

    @Override
    public MapCodec<InvertedSignLevelBasedValue> codec() {
        return CODEC;
    }
}
```

注册基于等级的值：

```java
public static final DeferredRegister<MapCodec<? extends LevelBasedValue>> LEVEL_BASED_VALUES =
    DeferredRegister.create(Registries.ENCHANTMENT_LEVEL_BASED_VALUE_TYPE, ExampleMod.MOD_ID);

public static final Supplier<MapCodec<? extends LevelBasedValue>> INVERTED_SIGN =
    LEVEL_BASED_VALUES.register("inverted_sign", () -> InvertedSignLevelBasedValue.CODEC);
```

---

### 自定义战利品条件

要创建自定义战利品条件，可以实现 `LootItemCondition`​ 接口。以下是一个示例，创建一个仅在玩家具有特定经验等级时通过的条件：

```java
public record HasXpLevelCondition(int level) implements LootItemCondition {
    public static final MapCodec<HasXpLevelCondition> CODEC = RecordCodecBuilder.mapCodec(inst -> inst.group(
        Codec.INT.fieldOf("level").forGetter(HasXpLevelCondition::level)
    ).apply(inst, HasXpLevelCondition::new));

    @Override
    public boolean test(LootContext context) {
        Entity entity = context.getParamOrNull(LootContextParams.KILLER_ENTITY);
        return entity instanceof Player player && player.experienceLevel >= level;
    }

    @Override
    public Set<LootContextParam<?>> getReferencedContextParams() {
        return ImmutableSet.of(LootContextParams.KILLER_ENTITY);
    }
}
```

注册战利品条件类型：

```java
public static final DeferredRegister<LootItemConditionType> LOOT_CONDITION_TYPES =
    DeferredRegister.create(Registries.LOOT_CONDITION_TYPE, ExampleMod.MOD_ID);

public static final Supplier<LootItemConditionType> MIN_XP_LEVEL =
    LOOT_CONDITION_TYPES.register("min_xp_level", () -> new LootItemConditionType(HasXpLevelCondition.CODEC));
```

在战利品条件类中重写 `getType()`​：

```java
@Override
public LootItemConditionType getType() {
    return MIN_XP_LEVEL.get();
}
```

---

### 自定义战利品函数

要创建自定义战利品函数，可以扩展 `LootItemFunction`​ 类。以下是一个示例，创建一个为物品随机附魔的函数：

```java
public class RandomEnchantmentWithLevelFunction extends LootItemConditionalFunction {
    private final Optional<HolderSet<Enchantment>> enchantments;
    private final int level;

    public static final MapCodec<RandomEnchantmentWithLevelFunction> CODEC =
        RecordCodecBuilder.mapCodec(inst -> commonFields(inst).and(inst.group(
            RegistryCodecs.homogeneousList(Registries.ENCHANTMENT).optionalFieldOf("enchantments").forGetter(e -> e.enchantments),
            Codec.INT.fieldOf("level").forGetter(e -> e.level)
        ).apply(inst, RandomEnchantmentWithLevelFunction::new));

    public RandomEnchantmentWithLevelFunction(List<LootItemCondition> conditions, Optional<HolderSet<Enchantment>> enchantments, int level) {
        super(conditions);
        this.enchantments = enchantments;
        this.level = level;
    }

    @Override
    public ItemStack run(ItemStack stack, LootContext context) {
        RandomSource random = context.getRandom();
        List<Holder<Enchantment>> stream = this.enchantments
            .map(HolderSet::stream)
            .orElseGet(() -> context.getLevel().registryAccess().registryOrThrow(Registries.ENCHANTMENT).holders().map(Function.identity()))
            .filter(e -> e.value().canEnchant(stack))
            .toList();
        Optional<Holder<Enchantment>> optional = Util.getRandomSafe(list, random);
        if (optional.isEmpty()) {
            LOGGER.warn("Couldn't find a compatible enchantment for {}", stack);
        } else {
            if (stack.is(Items.BOOK)) {
                stack = new ItemStack(Items.ENCHANTED_BOOK);
            }
            stack.enchant(enchantment, Mth.nextInt(random, enchantment.value().getMinLevel(), enchantment.value().getMaxLevel()));
        }
        return stack;
    }
}
```

注册战利品函数类型：

```java
public static final DeferredRegister<LootItemFunctionType<?>> LOOT_FUNCTION_TYPES =
    DeferredRegister.create(Registries.LOOT_FUNCTION_TYPE, ExampleMod.MOD_ID);

public static final Supplier<LootItemFunctionType<RandomEnchantmentWithLevelFunction>> RANDOM_ENCHANTMENT_WITH_LEVEL =
    LOOT_FUNCTION_TYPES.register("random_enchantment_with_level", () -> new LootItemFunctionType(RandomEnchantmentWithLevelFunction.CODEC));
```

在战利品函数类中重写 `getType()`​：

```java
@Override
public LootItemFunctionType getType() {
    return RANDOM_ENCHANTMENT_WITH_LEVEL.get();
}
```
