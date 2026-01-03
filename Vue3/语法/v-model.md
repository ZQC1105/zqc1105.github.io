在 Vue 3 中，`v-model` 是一个**语法糖（syntactic sugar）**，用于在**表单输入元素或组件**上创建**双向数据绑定（two-way data binding）**。它的核心作用是：

> **自动同步父组件的数据与子组件（或 DOM 元素）的值，并在值变化时更新父组件的数据。**

---

## 🧩 1. 在原生表单元素上使用（如 `<input>`、`<textarea>`、`<select>`）

### 示例：

```vue
<template>
  <input v-model="message" placeholder="请输入内容" />
  <p>你输入了：{{ message }}</p>
</template>

<script setup>
import { ref } from "vue";
const message = ref(""); // 响应式数据
</script>
```

### 等价于（手动写法）：

```vue
<input :value="message" @input="message = $event.target.value" />
```

✅ `v-model` 自动做了两件事：

- 将 `value` 绑定到变量（`:value`）
- 监听 `input` 事件并更新变量（`@input`）

---

## 🧩 2. 在自定义组件上使用（Vue 3 的重大改进）

在 Vue 3 中，`v-model` 默认使用 **`modelValue` prop + `update:modelValue` 事件**。

### 子组件（CustomInput.vue）：

```vue
<template>
  <input
    :value="modelValue"
    @input="$emit('update:modelValue', $event.target.value)"
    placeholder="子组件输入框"
  />
</template>

<script setup>
// 接收父组件传来的值
defineProps({
  modelValue: String,
});

// 声明要 emit 的事件（可选但推荐）
defineEmits(["update:modelValue"]);
</script>
```

### 父组件使用：

```vue
<template>
  <CustomInput v-model="searchText" />
  <p>搜索词：{{ searchText }}</p>
</template>

<script setup>
import { ref } from "vue";
import CustomInput from "./CustomInput.vue";

const searchText = ref("");
</script>
```

> ✅ 这就是 Vue 3 的默认 `v-model` 协议：
>
> - prop 名：`modelValue`
> - 事件名：`update:modelValue`

---

## 🧩 3. 多个 v-model（Vue 3 新特性）

Vue 3 支持在同一个组件上使用**多个 `v-model`**，通过指定参数实现：

### 子组件：

```vue
<template>
  <input
    :value="firstName"
    @input="$emit('update:firstName', $event.target.value)"
  />
  <input
    :value="lastName"
    @input="$emit('update:lastName', $event.target.value)"
  />
</template>

<script setup>
defineProps({
  firstName: String,
  lastName: String,
});
defineEmits(["update:firstName", "update:lastName"]);
</script>
```

### 父组件：

```vue
<template>
  <FullNameInput
    v-model:first-name="user.first"
    v-model:last-name="user.last"
  />
</template>

<script setup>
import { reactive } from "vue";
const user = reactive({ first: "", last: "" });
</script>
```

> 🔑 语法：`v-model:propName="variable"`  
> 对应事件：`update:propName`

---

## 🧩 4. 修饰符（Modifiers）

`v-model` 支持一些内置修饰符：

| 修饰符    | 作用                                  |
| --------- | ------------------------------------- |
| `.lazy`   | 改为在 `change` 而不是 `input` 时同步 |
| `.number` | 自动将输入转为 `Number` 类型          |
| `.trim`   | 自动去除首尾空格                      |

### 示例：

```vue
<input v-model.trim.number="age" />
<!-- 输入 " 25 " → 变成数字 25 -->
```

---

## ✅ 总结：`v-model` 的作用

| 场景             | 作用                                                               |
| ---------------- | ------------------------------------------------------------------ |
| **原生表单元素** | 自动绑定 `value` + 监听 `input` 事件                               |
| **自定义组件**   | 约定使用 `modelValue` prop 和 `update:modelValue` 事件实现双向绑定 |
| **多值绑定**     | 通过 `v-model:xxx` 支持多个双向绑定                                |
| **提升开发体验** | 避免手动写 `:value` + `@input`，代码更简洁                         |

---

## 📚 官方文档参考

- 中文：https://cn.vuejs.org/guide/essentials/forms.html#basic-usage
- 自定义组件 v-model：https://cn.vuejs.org/guide/components/v-model.html

> 💡 Vue 3 的 `v-model` 比 Vue 2 更灵活、更一致，是构建表单和可复用组件的核心工具。
