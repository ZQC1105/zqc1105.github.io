在 Bootstrap 中，Flex 功能是其内置的 CSS 弹性布局系统，基于 CSS 的 `display: flex` 属性。Bootstrap 提供了一系列预定义的类，用于快速实现灵活的响应式布局，而无需编写自定义 CSS。以下是 Bootstrap 中 Flex 功能的核心内容：

---

### **1. 启用 Flex 容器**
使用 `.d-flex` 或响应式变体（如 `.d-md-flex`）将元素设为 Flex 容器：
```html
<div class="d-flex">Flex 容器</div>
```

---

### **2. 方向控制**
通过以下类控制主轴方向：
- `.flex-row`（默认）：水平从左到右。
- `.flex-row-reverse`：水平从右到左。
- `.flex-column`：垂直从上到下。
- `.flex-column-reverse`：垂直从下到上。

示例：
```html
<div class="d-flex flex-column">
  <div>项目 1</div>
  <div>项目 2</div>
</div>
```

---

### **3. 对齐方式**
#### **主轴对齐（水平）**
- `.justify-content-start`（默认）
- `.justify-content-center`
- `.justify-content-end`
- `.justify-content-between`（两端对齐，项目间间隔相等）
- `.justify-content-around`（项目两侧间隔相等）

#### **交叉轴对齐（垂直）**
- `.align-items-start`
- `.align-items-center`
- `.align-items-end`
- `.align-items-stretch`（默认，拉伸填满容器）

#### **多行对齐（需换行）**
- `.align-content-start`
- `.align-content-center`
- `.align-content-between`
- `.align-content-around`

---

### **4. 子元素对齐**
对单个 Flex 项目使用：
- `.align-self-start`
- `.align-self-center`
- `.align-self-end`
- `.align-self-stretch`

---

### **5. 填充与等宽**
- `.flex-fill`：让子元素自动填充剩余空间。
- `.flex-grow-1`：扩展占满额外空间。
- `.flex-shrink-1`：允许收缩。

示例：
```html
<div class="d-flex">
  <div class="flex-fill">等宽列</div>
  <div class="flex-fill">等宽列</div>
</div>
```

---

### **6. 自动外边距**
通过 `.mr-auto` 或 `.ml-auto` 将子元素推到另一侧：
```html
<div class="d-flex">
  <div class="mr-auto">左侧</div>
  <div>右侧</div>
</div>
```

---

### **7. 响应式 Flex 类**
所有类均可结合断点使用（如 `.justify-content-md-center`），适配不同屏幕尺寸。

---

### **8. 与 Grid 系统的区别**
- **Grid** 用于二维布局（行列同时控制）。
- **Flex** 更适合一维布局（行或列），适合导航栏、卡片组等场景。

---

### **示例：导航栏**
```html
<nav class="d-flex justify-content-between align-items-center">
  <div>Logo</div>
  <div class="d-flex">
    <a href="#">链接1</a>
    <a href="#">链接2</a>
  </div>
</nav>
```

---

### **总结**
Bootstrap 的 Flex 工具类覆盖了大多数常见布局需求，通过组合这些类可以快速构建响应式、灵活的页面结构。无需编写额外 CSS，即可实现复杂的对齐和分布效果。