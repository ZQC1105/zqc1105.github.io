## 🎨 Tailwind CSS 设置宽度的方法

### **1. 固定宽度类**

```html
<!-- 固定像素值 -->
<div class="w-1">w-1 (0.25rem = 4px)</div>
<div class="w-4">w-4 (1rem = 16px)</div>
<div class="w-8">w-8 (2rem = 32px)</div>
<div class="w-12">w-12 (3rem = 48px)</div>
<div class="w-16">w-16 (4rem = 64px)</div>
<div class="w-20">w-20 (5rem = 80px)</div>
<div class="w-24">w-24 (6rem = 96px)</div>
<div class="w-32">w-32 (8rem = 128px)</div>
<div class="w-40">w-40 (10rem = 160px)</div>
<div class="w-48">w-48 (12rem = 192px)</div>
<div class="w-56">w-56 (14rem = 224px)</div>
<div class="w-64">w-64 (16rem = 256px)</div>
<div class="w-72">w-72 (18rem = 288px)</div>
<div class="w-80">w-80 (20rem = 320px)</div>
<div class="w-96">w-96 (24rem = 384px)</div>

<!-- 特殊固定宽度 -->
<div class="w-auto">宽度自适应内容</div>
<div class="w-screen">全屏宽度 (100vw)</div>
<div class="w-min">最小内容宽度</div>
<div class="w-max">最大内容宽度</div>
<div class="w-fit">适应内容宽度</div>
```

### **2. 百分比宽度**

```html
<!-- 百分比宽度 -->
<div class="w-1/2">50% 宽度</div>
<div class="w-1/3">33.333% 宽度</div>
<div class="w-2/3">66.666% 宽度</div>
<div class="w-1/4">25% 宽度</div>
<div class="w-2/4">50% 宽度</div>
<div class="w-3/4">75% 宽度</div>
<div class="w-1/5">20% 宽度</div>
<div class="w-2/5">40% 宽度</div>
<div class="w-3/5">60% 宽度</div>
<div class="w-4/5">80% 宽度</div>
<div class="w-1/6">16.666% 宽度</div>
<div class="w-2/6">33.333% 宽度</div>
<div class="w-3/6">50% 宽度</div>
<div class="w-4/6">66.666% 宽度</div>
<div class="w-5/6">83.333% 宽度</div>
<div class="w-full">100% 宽度</div>
<div class="w-11/12">91.666% 宽度</div>
```

### **3. 响应式宽度**

```html
<!-- 响应式设计：不同屏幕尺寸下不同宽度 -->
<div class="w-full md:w-1/2 lg:w-1/3 xl:w-1/4">
  <!--
    移动端: 100% 宽度
    md屏幕(≥768px): 50% 宽度  
    lg屏幕(≥1024px): 33.333% 宽度
    xl屏幕(≥1280px): 25% 宽度
  -->
</div>

<!-- 断点前缀 -->
<div class="w-16 sm:w-32 md:w-48 lg:w-64 xl:w-96">
  <!-- 随屏幕增大宽度增加 -->
</div>
```

### **4. 最大/最小宽度**

```html
<!-- 最大宽度 -->
<div class="max-w-none">无最大宽度限制</div>
<div class="max-w-xs">最大宽度 20rem (320px)</div>
<div class="max-w-sm">最大宽度 24rem (384px)</div>
<div class="max-w-md">最大宽度 28rem (448px)</div>
<div class="max-w-lg">最大宽度 32rem (512px)</div>
<div class="max-w-xl">最大宽度 36rem (576px)</div>
<div class="max-w-2xl">最大宽度 42rem (672px)</div>
<div class="max-w-3xl">最大宽度 48rem (768px)</div>
<div class="max-w-4xl">最大宽度 56rem (896px)</div>
<div class="max-w-5xl">最大宽度 64rem (1024px)</div>
<div class="max-w-6xl">最大宽度 72rem (1152px)</div>
<div class="max-w-7xl">最大宽度 80rem (1280px)</div>
<div class="max-w-full">最大宽度 100%</div>
<div class="max-w-screen-sm">最大宽度 640px</div>
<div class="max-w-screen-md">最大宽度 768px</div>
<div class="max-w-screen-lg">最大宽度 1024px</div>
<div class="max-w-screen-xl">最大宽度 1280px</div>
<div class="max-w-screen-2xl">最大宽度 1536px</div>

<!-- 最小宽度 -->
<div class="min-w-0">最小宽度 0</div>
<div class="min-w-full">最小宽度 100%</div>
<div class="min-w-min">最小内容宽度</div>
<div class="min-w-max">最大内容宽度</div>
<div class="min-w-fit">适应内容宽度</div>
```

### **5. 组合使用**

```html
<!-- 固定宽度 + 居中 -->
<div class="w-64 mx-auto">固定宽度 256px 并水平居中</div>

<!-- 百分比宽度 + 最大宽度限制 -->
<div class="w-full max-w-4xl mx-auto">100%宽度但不超过896px，并居中</div>

<!-- 响应式组合 -->
<div class="w-full sm:max-w-md md:max-w-lg lg:max-w-2xl mx-auto">
  <!-- 移动端全宽，大屏幕有最大限制 -->
</div>

<!-- 配合其他布局 -->
<div class="flex">
  <div class="w-1/4">侧边栏 25% 宽度</div>
  <div class="w-3/4">主内容区 75% 宽度</div>
</div>
```

### **6. 在 Vue + Naive UI 中使用**

```vue
<template>
  <!-- 在 Naive UI 组件上使用 -->
  <n-card class="w-full md:w-2/3 lg:w-1/2 mx-auto">
    <template #header>
      <div class="w-full text-center">标题</div>
    </template>

    <n-layout class="min-w-80">
      <!-- 固定最小宽度 -->
      <n-space class="w-full">
        <n-button class="w-32">固定宽度按钮</n-button>
        <n-button class="w-full">全宽按钮</n-button>
      </n-space>
    </n-layout>
  </n-card>

  <!-- 响应式容器 -->
  <div class="container mx-auto px-4">
    <div class="grid grid-cols-1 md:grid-cols-2 lg:grid-cols-3 gap-4">
      <div class="w-full">内容1</div>
      <div class="w-full">内容2</div>
      <div class="w-full">内容3</div>
    </div>
  </div>
</template>
```

### **7. 自定义宽度（如果需要）**

在 `tailwind.config.js` 中扩展：

```javascript
module.exports = {
  theme: {
    extend: {
      width: {
        128: "32rem", // 512px
        144: "36rem", // 576px
        160: "40rem", // 640px
        200: "50rem", // 800px
        "1/10": "10%",
        "3/10": "30%",
        "7/10": "70%",
        "9/10": "90%",
        // 自定义像素值
        "300px": "300px",
        "400px": "400px",
        "500px": "500px",
      },
      maxWidth: {
        "8xl": "88rem", // 1408px
        "9xl": "96rem", // 1536px
        prose: "65ch", // 适合阅读的宽度
      },
      minWidth: {
        20: "5rem", // 80px
        40: "10rem", // 160px
        60: "15rem", // 240px
      },
    },
  },
};
```

然后在 Vue 中使用自定义宽度：

```vue
<template>
  <div class="w-300px">自定义 300px 宽度</div>
  <div class="max-w-prose">适合阅读的最大宽度</div>
  <div class="min-w-40">最小宽度 160px</div>
</template>
```

### **8. 实用示例**

```vue
<template>
  <!-- 侧边栏布局 -->
  <div class="flex min-h-screen">
    <!-- 固定宽度侧边栏 -->
    <aside class="w-64 bg-gray-800 text-white">侧边栏内容</aside>

    <!-- 自适应主区域 -->
    <main class="flex-1 p-6">
      <!-- 响应式卡片 -->
      <div class="max-w-4xl mx-auto">
        <n-card class="w-full">
          <div class="space-y-4">
            <!-- 输入框组 -->
            <div class="flex gap-4">
              <n-input class="w-1/3" placeholder="短输入" />
              <n-input class="w-2/3" placeholder="长输入" />
            </div>

            <!-- 按钮组 -->
            <div class="flex gap-2">
              <n-button class="w-32">取消</n-button>
              <n-button class="flex-1" type="primary">确认</n-button>
            </div>
          </div>
        </n-card>
      </div>
    </main>
  </div>
</template>
```

### **📝 快速参考表**

| 类名               | 描述         | 等效值            |
| ------------------ | ------------ | ----------------- |
| `w-0` to `w-96`    | 固定宽度     | 0-24rem (0-384px) |
| `w-1/2` to `w-5/6` | 百分比宽度   | 50%-83.33%        |
| `w-full`           | 100% 宽度    | 100%              |
| `w-screen`         | 视口宽度     | 100vw             |
| `w-auto`           | 自动宽度     | auto              |
| `w-min`            | 最小内容宽度 | min-content       |
| `w-max`            | 最大内容宽度 | max-content       |
| `w-fit`            | 适应内容     | fit-content       |
| `max-w-*`          | 最大宽度限制 | 各种尺寸          |
| `min-w-*`          | 最小宽度限制 | 各种尺寸          |

选择最适合你布局需求的宽度类即可！
