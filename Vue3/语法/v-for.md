Vue 3 中的 `v-for` 是用于渲染列表数据的核心指令。以下是详细说明和示例：

## 基础用法

### 1. 遍历数组
```vue
<template>
  <ul>
    <li v-for="(item, index) in items" :key="item.id">
      {{ index }}: {{ item.name }}
    </li>
  </ul>
</template>

<script setup>
const items = [
  { id: 1, name: 'Apple' },
  { id: 2, name: 'Banana' },
  { id: 3, name: 'Orange' }
]
</script>
```

### 2. 遍历对象
```vue
<template>
  <ul>
    <li v-for="(value, key, index) in user" :key="key">
      {{ index }}. {{ key }}: {{ value }}
    </li>
  </ul>
</template>

<script setup>
const user = {
  name: 'John',
  age: 30,
  city: 'New York'
}
</script>
```

### 3. 遍历数字范围
```vue
<template>
  <span v-for="n in 5" :key="n">{{ n }} </span>
</template>
```

## 关键特性

### 1. `key` 的重要性
- **必须提供唯一 `key`**（推荐使用 ID）
- 帮助 Vue 识别节点，实现高效更新
- 避免使用索引作为 key（除非列表静态）

```vue
<!-- 推荐 -->
<div v-for="item in items" :key="item.id">

<!-- 避免（除非列表静态） -->
<div v-for="(item, index) in items" :key="index">
```

### 2. 使用 `v-for` 和 `v-if`
- Vue 3 中 `v-if` 比 `v-for` 优先级更高
- 避免在同一元素上同时使用

```vue
<!-- 正确做法：使用计算属性过滤 -->
<template v-for="item in activeItems" :key="item.id">
  <li v-if="item.isActive">{{ item.name }}</li>
</template>

<!-- 或使用计算属性 -->
<script setup>
import { computed } from 'vue'

const items = ref([...])
const activeItems = computed(() => 
  items.value.filter(item => item.isActive)
)
</script>
```

## Composition API 示例

### 1. 响应式列表
```vue
<template>
  <ul>
    <li v-for="todo in todos" :key="todo.id">
      {{ todo.text }}
      <button @click="removeTodo(todo.id)">×</button>
    </li>
  </ul>
</template>

<script setup>
import { ref } from 'vue'

const todos = ref([
  { id: 1, text: 'Learn Vue' },
  { id: 2, text: 'Build project' }
])

const removeTodo = (id) => {
  todos.value = todos.value.filter(todo => todo.id !== id)
}
</script>
```

### 2. 嵌套 v-for
```vue
<template>
  <div v-for="category in categories" :key="category.id">
    <h3>{{ category.name }}</h3>
    <ul>
      <li v-for="product in category.products" :key="product.id">
        {{ product.name }}
      </li>
    </ul>
  </div>
</template>
```

## 性能优化技巧

### 1. 使用 `v-once` 静态化
```vue
<li v-for="item in staticList" :key="item.id" v-once>
  {{ item.name }} <!-- 只渲染一次，不更新 -->
</li>
```

### 2. 虚拟滚动（大数据量）
```vue
<template>
  <RecycleScroller
    :items="largeList"
    :item-size="50"
    key-field="id"
  >
    <template #default="{ item }">
      <div>{{ item.name }}</div>
    </template>
  </RecycleScroller>
</template>
```

### 3. 使用 `:key` 强制更新
```vue
<!-- 当 item 内容改变但引用不变时 -->
<div v-for="item in items" :key="item.id + item.updatedAt">
  {{ item.content }}
</div>
```

## 实战示例：Todo List

```vue
<template>
  <div>
    <input v-model="newTodo" @keyup.enter="addTodo">
    <ul>
      <li v-for="todo in filteredTodos" :key="todo.id">
        <input type="checkbox" v-model="todo.done">
        <span :class="{ done: todo.done }">{{ todo.text }}</span>
      </li>
    </ul>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

let id = 0
const newTodo = ref('')
const todos = ref([])

const addTodo = () => {
  if (newTodo.value.trim()) {
    todos.value.push({
      id: id++,
      text: newTodo.value,
      done: false
    })
    newTodo.value = ''
  }
}

const filteredTodos = computed(() => 
  todos.value.filter(todo => !todo.done)
)
</script>

<style>
.done {
  text-decoration: line-through;
  color: #999;
}
</style>
```

## 常见注意事项

1. **避免修改循环中的数组**：使用数组方法（push, pop, splice等）
2. **保持 `key` 稳定**：不要用随机数作为 key
3. **复杂数据使用计算属性**：提高性能
4. **大列表考虑分页/虚拟滚动**：避免渲染过多 DOM 节点
5. **使用 `<template>` 包装多个元素**：

```vue
<template v-for="item in items" :key="item.id">
  <h3>{{ item.title }}</h3>
  <p>{{ item.content }}</p>
</template>
```

`v-for` 是 Vue 中最常用的指令之一，合理使用能显著提升应用性能和开发效率。