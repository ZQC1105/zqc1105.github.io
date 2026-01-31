`watch` 和 `watchEffect` 都是 Vue 3 的响应式监听 API，但它们在用法和用途上有重要区别。

## 核心区别

### **watch** - 显式监听特定数据源

```typescript
// 你需要明确指定要监听什么
watch(source, callback);
```

### **watchEffect** - 自动收集依赖

```typescript
// 自动追踪回调函数中用到的响应式数据
watchEffect(callback);
```

## 详细对比

### 1. **参数不同**

#### **watch**

```typescript
// 三个参数
watch(
  source,          // 必填：明确的数据源
  callback,        // 必填：数据变化时的回调
  options?         // 可选：配置
)
```

#### **watchEffect**

```typescript
// 两个参数
watchEffect(
  callback,        // 必填：自动追踪依赖
  options?         // 可选：配置（更少选项）
)
```

### 2. **依赖收集方式不同**

#### **watch** - 手动指定依赖

```typescript
const count = ref(0);
const name = ref("Alice");

// 只监听 count，不监听 name
watch(count, (newCount) => {
  console.log("count 变化了:", newCount);
});

// 即使 name 变化，上面的 watch 也不会触发
```

#### **watchEffect** - 自动追踪依赖

```typescript
const count = ref(0);
const name = ref("Alice");

// 自动追踪回调中用到的所有响应式数据
watchEffect(() => {
  console.log("count:", count.value, "name:", name.value);
  // 这里用到了 count.value 和 name.value
  // 所以两者任何一个变化，都会触发这个 effect
});

count.value++; // 触发
name.value = "Bob"; // 也触发
```

### 3. **访问新旧值的能力**

#### **watch** - 可以访问新旧值

```typescript
const count = ref(0);

watch(count, (newValue, oldValue) => {
  console.log(`从 ${oldValue} 变为 ${newValue}`);
});
// 输出：从 0 变为 1
```

#### **watchEffect** - 无法访问旧值

```typescript
const count = ref(0);

watchEffect(() => {
  console.log("当前值:", count.value);
  // ❌ 无法知道变化前的值是什么
});
// 只输出：当前值: 1
```

### 4. **初始执行时机**

#### **watch** - 默认不立即执行

```typescript
const count = ref(0);

watch(count, () => {
  console.log("watch 触发");
});

count.value = 1;
// 输出：watch 触发（只在变化时触发）
```

#### **watchEffect** - 立即执行一次

```typescript
const count = ref(0);

watchEffect(() => {
  console.log("当前 count:", count.value);
});
// 立即输出：当前 count: 0（立即执行）

count.value = 1;
// 输出：当前 count: 1
```

### 5. **实际使用场景**

#### **适合使用 watch 的场景**：

**场景1：需要新旧值对比**

```typescript
const userId = ref(1);

watch(userId, async (newId, oldId) => {
  // 新旧 ID 不同时才请求
  if (newId !== oldId) {
    const user = await fetchUser(newId);
    userData.value = user;
  }
});
```

**场景2：精确监听特定数据**

```typescript
// 只监听搜索关键词，不监听其他状态
const searchKeyword = ref("");

watch(searchKeyword, async (keyword) => {
  if (keyword.trim()) {
    const results = await search(keyword);
    searchResults.value = results;
  }
});
```

**场景3：监听路由参数（你的案例）**

```typescript
const route = useRoute();

// 只监听 page 参数变化
watch(
  () => route.query.page,
  (newPage) => {
    currentPage.value = parseInt(newPage || "1");
  },
);
```

#### **适合使用 watchEffect 的场景**：

**场景1：依赖多个响应式数据**

```typescript
const form = reactive({
  name: "",
  email: "",
  phone: "",
});

// 自动追踪 form 中的所有字段
watchEffect(() => {
  // 任何一个字段变化都会触发
  const isFormValid = form.name && form.email && form.phone;
  submitButtonDisabled.value = !isFormValid;
});
```

**场景2：副作用操作**

```typescript
const scrollPosition = ref(0);

// 自动响应滚动位置变化
watchEffect(() => {
  // 每次 scrollPosition 变化都更新 DOM
  document.querySelector(".container").scrollTop = scrollPosition.value;
});
```

**场景3：实时计算**

```typescript
const price = ref(100);
const quantity = ref(2);
const discount = ref(0.1);

// 自动计算总价
watchEffect(() => {
  total.value = price.value * quantity.value * (1 - discount.value);
  // 依赖：price、quantity、discount
  // 任何一个变化都会重新计算
});
```

### 6. **options 配置区别**

#### **两者都有的选项**：

```typescript
// 共同选项
{
  flush: 'pre' | 'post' | 'sync',  // 触发时机
  onTrack: (event) => {},          // 调试：依赖被追踪时
  onTrigger: (event) => {}         // 调试：触发时
}
```

#### **watch 独有的选项**：

```typescript
{
  immediate: true,    // 立即执行回调
  deep: true         // 深度监听对象/数组
}
```

#### **watchEffect 不能用的选项**：

```typescript
// ❌ watchEffect 不能用这些
{
  immediate: true,  // 不需要，因为 watchEffect 总是立即执行
  deep: true       // 不需要，因为自动追踪依赖
}
```

### 7. **性能考量**

#### **watch** - 更精确，性能更好

```typescript
// 只监听特定数据，减少不必要的执行
const largeArray = ref([
  /* 大数据 */
]);
const filter = ref("");

watch(filter, (newFilter) => {
  // 只在 filter 变化时执行，避免数组变化时的重复计算
  filteredData.value = largeArray.value.filter((item) =>
    item.includes(newFilter),
  );
});
```

#### **watchEffect** - 依赖自动收集，可能过度触发

```typescript
const largeArray = ref([
  /* 大数据 */
]);
const filter = ref("");

watchEffect(() => {
  // ⚠️ 如果 largeArray 内容变化（即使与过滤无关），也会触发
  filteredData.value = largeArray.value.filter((item) =>
    item.includes(filter.value),
  );
});
```

### 8. **代码示例对比**

#### **相同功能的不同实现**

**使用 watch**：

```typescript
const userId = ref(1);
const userData = ref(null);

// 明确监听 userId
watch(
  userId,
  async (newId) => {
    userData.value = await fetchUser(newId);
  },
  { immediate: true },
); // 需要手动添加 immediate
```

**使用 watchEffect**：

```typescript
const userId = ref(1);
const userData = ref(null);

// 自动追踪 userId
watchEffect(async () => {
  userData.value = await fetchUser(userId.value);
});
```

### 9. **选择指南**

| 情况               | 推荐使用        | 理由                 |
| ------------------ | --------------- | -------------------- |
| 需要知道变化前的值 | **watch**       | 可以获取 oldValue    |
| 只关心特定数据变化 | **watch**       | 精确控制监听目标     |
| 依赖多个响应式数据 | **watchEffect** | 自动收集依赖更方便   |
| 立即执行副作用     | **watchEffect** | 默认立即执行         |
| 深度监听对象       | **watch**       | 有 `deep: true` 选项 |
| 简单的响应式副作用 | **watchEffect** | 代码更简洁           |

### 10. **你的分页案例应该用哪个？**

你的当前代码：

```typescript
watch(currentPage, (newPage) => {
  fetchForumThreadPosts(newPage);
});
```

**这更适合用 watch**，因为：

1. 你只监听 `currentPage` 这一个特定数据
2. 不需要自动追踪其他依赖
3. 代码意图明确：当页码变化时获取数据

如果用 `watchEffect` 重写：

```typescript
// 用 watchEffect 实现
watchEffect(() => {
  fetchForumThreadPosts(currentPage.value);
  // 如果 futureThreadPostsDto 也在回调中被使用，
  // 那么它的变化也会触发这个 effect，可能不是你想要的
});
```

**结论**：

- 你的场景明确需要监听 `currentPage` 变化
- 不需要自动追踪其他依赖
- **使用 watch 更合适，意图更清晰**

**通用建议**：

- 当你明确知道要监听什么时 → 用 **watch**
- 当依赖关系复杂或需要自动追踪时 → 用 **watchEffect**
- 需要旧值 → 用 **watch**
- 立即执行副作用 → 用 **watchEffect**
