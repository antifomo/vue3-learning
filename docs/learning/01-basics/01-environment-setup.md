# Day 1：Vue3 环境搭建

## 📚 学习目标
- [ ] 安装Node.js开发环境
- [ ] 使用Vite创建Vue3项目
- [ ] 理解Vue3项目结构
- [ ] 运行第一个Hello World应用
- [ ] 配置基础开发环境

## 📝 学习内容

### 1. 前置准备

#### 1.1 安装Node.js
```bash
# 检查是否已安装Node.js
node -v
# 应该显示 v16.0.0 或更高版本

# 检查npm版本
npm -v
# 应该显示 8.0.0 或更高版本
```

> 💡 **提示**：如果未安装，请访问 [Node.js官网](https://nodejs.org/) 下载LTS版本

#### 1.2 了解Vite
Vite是新一代前端构建工具，特点：
- ⚡ 极速的服务启动
- ⚡ 轻量快速的热重载
- 🛠️ 丰富的功能
- 📦 优化的构建

### 2. 创建Vue3项目

#### 2.1 使用Vite创建项目（交互式）

> 📌 **创建方式说明**：
> - `npm create vite@latest .` - 在当前目录创建（适合已有文件夹）
> - `npm create vite@latest my-vue3-app` - 创建新文件夹（推荐初学者使用）
> - 本教程使用第二种方式，项目名为 `my-vue3-app`

```bash
# 创建新的Vue3项目
npm create vite@latest my-vue3-app

# 会出现以下交互式选择：
```

**步骤1：选择框架**
```
◇  Select a framework:
│  › Vue         # 👈 选择这个（使用方向键↑↓，回车确认）
│    React
│    Preact
│    Lit
│    Svelte
│    Solid
│    Qwik
│    Others
```

**步骤2：选择变体**
```
◆  Select a variant:
│  › JavaScript      # 👈 初学者选择这个（纯JavaScript）
│    TypeScript      # 进阶选择（需要TypeScript基础）
│    Customize with create-vue ↗   # 官方脚手架（更多配置选项）
│    Nuxt ↗         # Nuxt框架（全栈框架）
```

#### 2.2 选择建议

##### 初学者推荐配置
- **框架**: Vue
- **变体**: JavaScript
- **原因**: 
  - 语法简单，学习曲线平缓
  - 不需要额外学习TypeScript
  - 专注于Vue3核心概念

##### 进阶配置说明
- **TypeScript版本**: 提供类型检查，适合大型项目，需要TypeScript基础
- **create-vue**: Vue官方脚手架，可选Router、Pinia、ESLint等工具
- **Nuxt**: 基于Vue的全栈框架，包含SSR、文件路由等高级功能

#### 2.3 创建项目常见问题

**Q: 不小心选错了怎么办？**
A: 按 `Ctrl+C` 退出，重新运行命令选择

**Q: JavaScript和TypeScript有什么区别？**
A: 
- JavaScript：动态类型，灵活简单，适合快速开发
- TypeScript：静态类型，代码更安全，IDE提示更好

**Q: 什么时候用create-vue？**
A: 当你需要完整的项目配置时：
- 需要Vue Router（路由）
- 需要Pinia（状态管理）
- 需要ESLint（代码检查）
- 需要单元测试配置

**Q: 如果想跳过交互选择怎么办？**
A: 可以使用模板参数直接创建：
```bash
# 直接创建JavaScript版本
npm create vite@latest . -- --template vue

# 直接创建TypeScript版本  
npm create vite@latest . -- --template vue-ts
```

#### 2.4 安装依赖
```bash
# 进入项目目录
cd my-vue3-app

# 安装项目依赖
npm install

# 如果使用pnpm（推荐，更快）
pnpm install
```

### 3. 项目结构解析

> ⚠️ **重要提示**：以下所有操作都需要在项目根目录（my-vue3-app）中执行

```
my-vue3-app/            # 您的项目根目录
├── node_modules/        # 依赖包（自动生成）
├── public/             # 静态资源
│   └── vite.svg       # 网站图标
├── src/                # 源代码目录
│   ├── assets/        # 资源文件（图片、样式等）
│   ├── components/    # Vue组件
│   │   └── HelloWorld.vue
│   ├── App.vue        # 根组件
│   ├── main.js        # 应用入口文件
│   └── style.css      # 全局样式
├── index.html          # HTML入口文件
├── package.json        # 项目配置文件
├── vite.config.js      # Vite配置文件
└── README.md          # 项目说明文档
```

#### 文件说明：
- **index.html**: 应用的入口HTML，Vite直接使用
- **main.js**: JavaScript入口，创建Vue应用实例
- **App.vue**: 根组件，所有组件的父组件
- **components/**: 存放可复用的Vue组件
- **vite.config.js**: Vite的配置文件

### 4. 理解核心文件

#### 4.1 main.js - 应用入口
```javascript
import { createApp } from 'vue'  // 从vue导入createApp函数
import './style.css'            // 导入全局样式
import App from './App.vue'      // 导入根组件

createApp(App).mount('#app')     // 创建应用实例并挂载到#app元素
```

#### 4.2 App.vue - 根组件结构
```vue
<script setup>
// JavaScript逻辑部分 - 使用Composition API
import HelloWorld from './components/HelloWorld.vue'
</script>

<template>
  <!-- HTML模板部分 -->
  <div>
    <HelloWorld msg="Welcome to Vue 3!" />
  </div>
</template>

<style scoped>
/* CSS样式部分 - scoped表示样式只在当前组件生效 */
</style>
```

#### 4.3 Vue单文件组件（SFC）三要素
1. **`<script>`**: 组件的JavaScript逻辑
2. **`<template>`**: 组件的HTML模板
3. **`<style>`**: 组件的CSS样式

#### 4.4 应用加载流程详解

理解Vue应用是如何从静态文件变成动态应用的，对掌握Vue3至关重要。

##### 📊 加载顺序图
```
浏览器访问 → index.html → main.js → App.vue → 子组件
    ↓            ↓           ↓         ↓         ↓
 请求页面    HTML结构    创建Vue应用  根组件   组件树渲染
```

##### 🔄 详细加载流程

**步骤1：浏览器请求 index.html**
```html
<!-- index.html -->
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <link rel="icon" type="image/svg+xml" href="/vite.svg">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Vite + Vue</title>
  </head>
  <body>
    <div id="app"></div>  <!-- 👈 Vue挂载点，初始为空 -->
    <script type="module" src="/src/main.js"></script>  <!-- 👈 入口脚本 -->
  </body>
</html>
```

执行细节：
- 浏览器加载HTML文档
- 解析到 `<div id="app"></div>` - 这是Vue的挂载点（目前是空的）
- 遇到 `<script type="module">` 标签，开始加载 main.js
- `type="module"` 表示使用ES6模块系统

**步骤2：加载并执行 main.js**
```javascript
// main.js 执行顺序
import { createApp } from 'vue'      // 1. 从vue包导入createApp函数
import './style.css'                 // 2. 加载全局样式
import App from './App.vue'          // 3. 加载根组件

const app = createApp(App)           // 4. 创建Vue应用实例
app.mount('#app')                    // 5. 挂载到DOM
```

执行细节：
1. **导入Vue核心函数**：
   - 从 `node_modules/vue` 加载 createApp 函数
   - Vite会处理这个导入，返回编译好的Vue代码

2. **加载全局样式**：
   - style.css 被Vite处理
   - 样式被注入到 `<head>` 中的 `<style>` 标签

3. **加载根组件**：
   - App.vue 被Vite即时编译成JavaScript对象
   - 包含了template编译后的渲染函数

4. **创建应用实例**：
   - `createApp(App)` 创建一个Vue应用实例
   - 此时应用还没有渲染，只是准备好了

5. **挂载应用**：
   - `.mount('#app')` 找到id="app"的元素
   - 渲染组件并替换其内容

**步骤3：App.vue 组件加载过程**
```vue
<!-- App.vue 原始代码 -->
<script setup>
import HelloWorld from './components/HelloWorld.vue'  // 导入子组件
const msg = 'Welcome'
</script>

<template>
  <HelloWorld :msg="msg" />
</template>
```

Vite编译后（简化版）：
```javascript
// App.vue 编译后的JavaScript对象
export default {
  setup() {
    const msg = 'Welcome'
    return { msg }
  },
  render() {
    // template编译成的渲染函数
    return h(HelloWorld, { msg: this.msg })
  }
}
```

**步骤4：组件渲染过程**
1. **创建组件实例**：Vue创建App组件的实例
2. **执行setup函数**：建立响应式数据
3. **生成虚拟DOM**：执行render函数，创建虚拟节点树
4. **渲染真实DOM**：将虚拟DOM转换为真实DOM元素
5. **替换挂载点**：新生成的DOM替换 `<div id="app"></div>` 的内容

##### ⚡ Vite在开发环境的作用

**1. 依赖预构建**
```
node_modules/vue → .vite/deps/vue.js (预构建的ESM模块)
```
- 将CommonJS模块转换为ESM
- 合并小模块，减少请求次数

**2. 即时编译**
```
.vue文件 → Vite处理 → JavaScript模块
```
- 浏览器请求.vue文件时，Vite实时编译
- 不需要提前打包整个应用

**3. 模块热替换（HMR）**
```javascript
// 修改代码后
if (import.meta.hot) {
  import.meta.hot.accept()  // 接受更新，不刷新页面
}
```

##### 🎯 关键概念理解

| 概念 | 说明 | 重要性 |
|------|------|--------|
| **挂载点** | `<div id="app">` Vue接管的DOM容器 | ⭐⭐⭐⭐⭐ |
| **单页应用(SPA)** | 整个应用只有一个HTML页面，内容动态切换 | ⭐⭐⭐⭐ |
| **虚拟DOM** | JavaScript对象表示的DOM结构 | ⭐⭐⭐⭐ |
| **组件树** | 从App.vue开始的组件父子关系结构 | ⭐⭐⭐⭐ |
| **响应式系统** | 数据变化自动触发视图更新 | ⭐⭐⭐⭐⭐ |

##### 🔍 开发者工具查看加载过程

**1. Network标签（网络）**
```
查看顺序：
1. localhost (HTML)
2. main.js
3. vue.js (从.vite/deps/)
4. App.vue
5. style.css
6. HelloWorld.vue
```

**2. Elements标签（元素）**
```html
<!-- 初始状态 -->
<div id="app"></div>

<!-- Vue渲染后 -->
<div id="app">
  <div class="hello-container">
    <h1>Hello Vue3!</h1>
    <!-- ... -->
  </div>
</div>
```

**3. Console控制台**
```javascript
// 查看Vue应用实例
console.log(app)  // 在main.js中打印

// 查看组件实例
const vm = app._instance.proxy
console.log(vm)
```

**4. Vue DevTools**
- 安装Vue官方浏览器插件
- 查看组件树结构
- 检查组件的props和data
- 追踪事件和性能

##### 💡 调试技巧

1. **添加调试点**
```javascript
// main.js
console.log('1. 开始创建应用')
const app = createApp(App)
console.log('2. 应用创建完成', app)
app.mount('#app')
console.log('3. 应用已挂载')
```

2. **查看编译结果**
在浏览器Sources标签中，可以看到Vite编译后的代码

3. **性能分析**
使用Performance标签记录应用加载过程

### 5. 创建第一个Hello World

#### 5.1 创建学习目录
```bash
# 确保在项目根目录中（如果还没进入）
cd my-vue3-app

# 创建学习专用目录
mkdir -p src/learning/01-basics
```

#### 5.2 创建HelloWorld.vue
创建文件：`src/learning/01-basics/HelloWorld.vue`

```vue
<script setup>
// 导入Vue3的响应式函数
import { ref } from 'vue'

// 创建响应式数据 - ref用于基本类型
const message = ref('Hello Vue3!')  // 响应式字符串
const count = ref(0)                // 响应式数字

// 定义方法 - 点击按钮时执行
const increment = () => {
  count.value++  // 修改ref值需要用.value
}

// 定义方法 - 修改消息
const changeMessage = () => {
  message.value = `你已经点击了 ${count.value} 次！`
}
</script>

<template>
  <div class="hello-container">
    <!-- 插值表达式：显示响应式数据 -->
    <h1>{{ message }}</h1>
    
    <!-- 显示计数 -->
    <p class="count">当前计数：{{ count }}</p>
    
    <!-- 事件绑定：@click 是 v-on:click 的简写 -->
    <div class="buttons">
      <button @click="increment" class="btn">
        点击 +1
      </button>
      
      <button @click="changeMessage" class="btn btn-primary">
        更新消息
      </button>
      
      <!-- 行内事件处理 -->
      <button @click="count = 0" class="btn btn-danger">
        重置
      </button>
    </div>
    
    <!-- 条件渲染：v-if -->
    <p v-if="count > 10" class="achievement">
      🎉 太棒了！你已经点击超过10次了！
    </p>
  </div>
</template>

<style scoped>
/* scoped: 样式只在当前组件生效 */
.hello-container {
  text-align: center;
  padding: 40px;
  font-family: Arial, sans-serif;
}

h1 {
  color: #42b883;  /* Vue绿色 */
  font-size: 2.5em;
  margin-bottom: 20px;
}

.count {
  font-size: 1.5em;
  color: #333;
  margin: 20px 0;
}

.buttons {
  display: flex;
  gap: 10px;
  justify-content: center;
  margin: 30px 0;
}

.btn {
  padding: 10px 20px;
  font-size: 16px;
  border: none;
  border-radius: 5px;
  cursor: pointer;
  background-color: #f0f0f0;
  transition: all 0.3s;
}

.btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 2px 8px rgba(0,0,0,0.2);
}

.btn-primary {
  background-color: #42b883;
  color: white;
}

.btn-danger {
  background-color: #ff6b6b;
  color: white;
}

.achievement {
  font-size: 1.2em;
  color: #ff6b6b;
  margin-top: 20px;
  animation: bounce 0.5s;
}

@keyframes bounce {
  0%, 100% { transform: translateY(0); }
  50% { transform: translateY(-10px); }
}
</style>
```

#### 5.3 修改App.vue使用新组件
```vue
<script setup>
// 导入我们的学习组件
import HelloWorld from './learning/01-basics/HelloWorld.vue'
</script>

<template>
  <div id="app">
    <HelloWorld />
  </div>
</template>

<style>
#app {
  margin: 0;
  padding: 0;
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  display: flex;
  justify-content: center;
  align-items: center;
}
</style>
```

### 6. 运行项目

```bash
# 确保在项目根目录
cd my-vue3-app

# 启动开发服务器
npm run dev

# 项目会运行在 http://localhost:5173
# 打开浏览器访问该地址
```

#### 常用命令：
```bash
npm run dev      # 启动开发服务器
npm run build    # 构建生产版本
npm run preview  # 预览生产构建
```

### 7. 开发工具推荐

#### 7.1 VS Code插件
- **Vue - Official**: Vue3官方插件（必装）
- **Volar**: Vue3语言支持
- **ESLint**: 代码规范检查
- **Prettier**: 代码格式化

#### 7.2 浏览器插件
- **Vue.js devtools**: Vue开发者工具（Chrome/Firefox）

### 8. 环境配置优化

#### 8.1 添加别名配置（vite.config.js）
```javascript
import { defineConfig } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig({
  plugins: [vue()],
  resolve: {
    alias: {
      '@': resolve(__dirname, 'src'),  // @ 指向 src 目录
    }
  }
})
```

使用别名后可以这样导入：
```javascript
import HelloWorld from '@/learning/01-basics/HelloWorld.vue'
```

## 💡 关键概念理解

### 1. ref() - 响应式基础详解

#### 核心概念
`ref()` 是 Vue3 中创建响应式数据的核心方法之一，专门用于包装基本类型值（string、number、boolean），使其具有响应式能力。

#### 完整示例代码

```vue
<template>
  <!-- 在模板中使用 ref 时，自动解包，不需要 .value -->
  <div class="ref-example">
    <h2>ref() 使用示例</h2>
    
    <!-- 1. 数字类型：计数器 -->
    <div>
      <p>计数器：{{ count }}</p>  <!-- 模板中直接使用 count，不需要 count.value -->
      <button @click="increment">点击 +1</button>
    </div>
    
    <!-- 2. 字符串类型：输入框 -->
    <div>
      <input v-model="message" placeholder="输入消息">  <!-- 直接绑定 message -->
      <p>你输入的是：{{ message }}</p>  <!-- 直接显示 message -->
    </div>
    
    <!-- 3. 布尔类型：显示/隐藏 -->
    <div>
      <button @click="toggleShow">切换显示</button>
      <p v-if="isVisible">这段文字可以显示/隐藏</p>  <!-- 直接使用 isVisible -->
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// 1. 创建 ref - 数字类型
const count = ref(0)  // 初始值为 0

// 2. 创建 ref - 字符串类型
const message = ref('Hello Vue3')  // 初始值为字符串

// 3. 创建 ref - 布尔类型
const isVisible = ref(true)  // 初始值为 true

// 在 JavaScript 中访问和修改 ref 的值，必须使用 .value
function increment() {
  count.value++  // 必须使用 .value 来修改
  console.log('当前计数：', count.value)  // 必须使用 .value 来访问
}

function toggleShow() {
  isVisible.value = !isVisible.value  // 必须使用 .value 来修改布尔值
  
  // 也可以基于条件修改其他 ref
  if (isVisible.value) {
    message.value = '现在显示中'  // 修改字符串 ref 也需要 .value
  } else {
    message.value = '已经隐藏了'
  }
}

// 更多 JavaScript 操作示例
setTimeout(() => {
  count.value = 10  // 3秒后设置为 10
  message.value = '3秒后自动更新'  // 同时更新消息
}, 3000)

// 监听 ref 的变化（示例）
console.log('初始值：', {
  count: count.value,      // 访问需要 .value
  message: message.value,   // 访问需要 .value
  isVisible: isVisible.value // 访问需要 .value
})
</script>
```

#### 关键点详解

##### 创建 ref
```javascript
const count = ref(0)        // 数字
const message = ref('text')  // 字符串
const isVisible = ref(true)  // 布尔
```

##### 在 JavaScript 中必须用 .value
```javascript
// ✅ 正确 - JavaScript 中访问
console.log(count.value)  

// ❌ 错误 - 这会打印 ref 对象本身
console.log(count)  

// ✅ 正确 - JavaScript 中修改
count.value = 5  

// ❌ 错误 - 这会破坏响应式
count = 5  
```

##### 在模板中自动解包（不需要 .value）
```vue
<!-- ✅ 正确 - 模板中直接使用 -->
<p>{{ count }}</p>
<input v-model="message">
<div v-if="isVisible">...</div>

<!-- ❌ 错误 - 模板中不需要 .value -->
<p>{{ count.value }}</p>  <!-- 这是错误的！ -->
```

#### 为什么需要 .value？

ref 返回的是一个对象，结构如下：
```javascript
{
  value: 实际的值,
  // ... 响应式相关的内部属性
}
```

这样设计的原因：
1. **保持引用稳定** - ref 对象本身的引用不变
2. **实现响应式追踪** - Vue 可以拦截 .value 的读写
3. **类型一致性** - 无论包装什么类型，都通过 .value 访问

#### 实际应用场景

```javascript
// 表单处理示例
const username = ref('')
const password = ref('')
const isLoading = ref(false)

async function handleLogin() {
  isLoading.value = true  // 开始加载
  
  try {
    const result = await login({
      username: username.value,  // 获取用户名
      password: password.value   // 获取密码
    })
    
    // 登录成功后清空
    username.value = ''
    password.value = ''
  } finally {
    isLoading.value = false  // 结束加载
  }
}
```

### 2. 事件处理详解

#### 基本语法
```vue
<!-- 完整语法 -->
<button v-on:click="handleClick">点击我</button>

<!-- 简写语法（推荐） -->
<button @click="handleClick">点击我</button>
```

#### 五种使用方式

```vue
<template>
  <div>
    <!-- 方式1：直接调用方法（最常用） -->
    <button @click="sayHello">直接调用方法</button>
    
    <!-- 方式2：内联语句 -->
    <button @click="count++">内联语句：计数+1</button>
    <button @click="message = '已点击'">修改消息</button>
    
    <!-- 方式3：传递参数 -->
    <button @click="greet('张三')">传递参数</button>
    
    <!-- 方式4：同时获取事件对象和参数 -->
    <button @click="handleEvent($event, 'data')">事件对象 + 参数</button>
    
    <!-- 方式5：多个语句（用分号分隔） -->
    <button @click="count++; message = '计数: ' + count">多个语句</button>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const count = ref(0)
const message = ref('初始消息')

// 方式1：简单方法
function sayHello() {
  console.log('Hello!')
}

// 方式3：接收参数
function greet(name) {
  alert(`你好，${name}！`)
}

// 方式4：同时接收事件对象和参数
function handleEvent(event, data) {
  console.log('事件对象:', event)
  console.log('自定义数据:', data)
  console.log('点击位置:', event.clientX, event.clientY)
}
</script>
```

#### 常用事件和修饰符

```vue
<template>
  <div>
    <!-- 常用事件类型 -->
    <input @input="onInput">           <!-- 输入事件 -->
    <input @change="onChange">         <!-- 值改变事件 -->
    <input @focus="onFocus">           <!-- 获得焦点 -->
    <input @blur="onBlur">             <!-- 失去焦点 -->
    <form @submit="onSubmit">          <!-- 表单提交 -->
    <div @mouseover="onMouseOver">     <!-- 鼠标悬停 -->
    <input @keyup="onKeyUp">           <!-- 键盘抬起 -->
    <input @keyup.enter="onEnter">     <!-- 回车键 -->
    
    <!-- 事件修饰符 -->
    <form @submit.prevent="handleSubmit">   <!-- 阻止默认行为 -->
    <div @click.stop="handleClick">         <!-- 阻止事件冒泡 -->
    <button @click.once="handleOnce">       <!-- 只触发一次 -->
    <div @click.self="handleSelf">          <!-- 只在元素自身触发 -->
    
    <!-- 组合修饰符 -->
    <a @click.prevent.stop="handleLink">链接</a>
  </div>
</template>
```

#### 事件修饰符详解

| 修饰符 | 作用 | 等价于 |
|--------|------|---------|
| `.prevent` | 阻止默认行为 | `event.preventDefault()` |
| `.stop` | 阻止事件冒泡 | `event.stopPropagation()` |
| `.once` | 事件只触发一次 | - |
| `.self` | 只在元素自身触发 | `event.target === event.currentTarget` |
| `.capture` | 使用捕获模式 | addEventListener 的 capture 选项 |
| `.passive` | 提升滚动性能 | addEventListener 的 passive 选项 |

### 3. 条件渲染详解

#### v-if 详解

v-if 指令用于条件性地渲染元素。当条件为假时，元素及其子元素不会被渲染到 DOM 中。

```vue
<template>
  <div>
    <!-- 单个条件 -->
    <p v-if="isLoggedIn">欢迎回来，用户！</p>
    
    <!-- v-if / v-else-if / v-else 链 -->
    <div v-if="score >= 90">
      优秀（90分以上）
    </div>
    <div v-else-if="score >= 60">
      及格（60-89分）
    </div>
    <div v-else>
      不及格（60分以下）
    </div>
    
    <!-- 在 template 上使用（不会渲染额外元素） -->
    <template v-if="showDetails">
      <h4>详细信息</h4>
      <p>姓名：张三</p>
      <p>年龄：25</p>
    </template>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const isLoggedIn = ref(true)
const score = ref(75)
const showDetails = ref(true)
</script>
```

#### v-show 详解

v-show 指令通过切换元素的 CSS `display` 属性来控制显示/隐藏。元素始终被渲染到 DOM 中。

```vue
<template>
  <div>
    <!-- 基本用法 -->
    <p v-show="isVisible">这个段落通过 v-show 控制显示</p>
    
    <!-- v-show 不能用在 template 上 -->
    <!-- ❌ 错误：<template v-show="show">...</template> -->
    
    <!-- v-show 没有 v-else -->
    <div v-show="condition">显示内容</div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const isVisible = ref(true)
const condition = ref(false)

// 切换显示/隐藏
function toggle() {
  isVisible.value = !isVisible.value
}
</script>
```

#### v-if vs v-show 对比

| 特性 | v-if | v-show |
|------|------|--------|
| **渲染方式** | 条件为假时，不渲染 DOM | 始终渲染，通过 CSS 控制显示 |
| **初始渲染开销** | 较小（false 时不渲染） | 较大（始终渲染） |
| **切换开销** | 较大（创建/销毁 DOM） | 较小（只改变 CSS） |
| **支持 template** | ✅ 支持 | ❌ 不支持 |
| **支持 v-else** | ✅ 支持 | ❌ 不支持 |

#### 使用场景建议

**何时使用 v-if：**
```javascript
// 1. 条件很少改变
// 2. 有复杂的子组件
// 3. 需要配合 v-else-if/v-else
// 4. 初始条件为 false 的情况

// 示例：用户权限控制
<div v-if="user.role === 'admin'">
  <AdminPanel />  <!-- 复杂的管理员面板组件 -->
</div>
<div v-else-if="user.role === 'user'">
  <UserDashboard />  <!-- 普通用户仪表板 -->
</div>
<div v-else>
  <LoginForm />  <!-- 登录表单 -->
</div>
```

**何时使用 v-show：**
```javascript
// 1. 需要频繁切换显示/隐藏
// 2. 元素内容简单
// 3. 初始就需要渲染的元素

// 示例：下拉菜单、提示框、标签页
<button @click="showMenu = !showMenu">菜单</button>
<div v-show="showMenu" class="dropdown-menu">
  <a href="#">选项1</a>
  <a href="#">选项2</a>
  <a href="#">选项3</a>
</div>
```

#### 性能对比示例

```vue
<template>
  <div class="performance-test">
    <h3>性能场景示例</h3>
    
    <!-- 场景1：频繁切换 - 推荐 v-show -->
    <div>
      <button @click="frequentToggle = !frequentToggle">
        频繁切换（适合 v-show）
      </button>
      <span v-show="frequentToggle">⭐ 状态指示器</span>
    </div>
    
    <!-- 场景2：条件很少改变 - 推荐 v-if -->
    <div>
      <button @click="rareToggle = !rareToggle">
        很少切换（适合 v-if）
      </button>
      <div v-if="rareToggle">
        <h4>复杂组件</h4>
        <p>包含大量内容...</p>
        <p>初始化开销大...</p>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

const frequentToggle = ref(false)
const rareToggle = ref(false)
</script>
```

#### 常见错误和注意事项

```vue
<!-- ❌ 错误：v-show 不能用在 template -->
<template v-show="condition">...</template>

<!-- ✅ 正确：v-if 可以用在 template -->
<template v-if="condition">...</template>

<!-- ❌ 错误：v-for 和 v-if 不要在同一元素上使用 -->
<li v-for="user in users" v-if="user.isActive">

<!-- ✅ 正确：使用 template 包装或计算属性 -->
<template v-for="user in users" :key="user.id">
  <li v-if="user.isActive">{{ user.name }}</li>
</template>

<!-- ❌ 错误：忘记阻止表单默认提交 -->
<form @submit="handleSubmit">  <!-- 会刷新页面 -->

<!-- ✅ 正确：使用 .prevent 修饰符 -->
<form @submit.prevent="handleSubmit">
```

### 核心要点总结

- **ref()**: 在 JavaScript 中用 `.value`，在模板中自动解包
- **@click**: 是 `v-on:click` 的简写，支持多种使用方式
- **v-if**: 控制元素的创建/销毁（适合条件很少改变的场景）
- **v-show**: 只切换 CSS display（适合频繁切换的场景）

## ✅ 今日练习

### 练习1：修改HelloWorld组件
1. 添加一个输入框，实现双向绑定
2. 添加一个列表，显示点击历史
3. 使用不同的颜色显示奇偶数

### 练习2：创建新组件
创建一个 `UserCard.vue` 组件：
- 显示用户名和年龄
- 有增加年龄的按钮
- 年龄大于18显示"成年"，否则显示"未成年"

## 🤔 思考题
1. ref和普通变量有什么区别？
2. 为什么在script中访问ref需要.value，而在template中不需要？
3. scoped样式是如何实现样式隔离的？

## 📊 学习总结

### 今日收获
- ✅ 成功搭建Vue3开发环境
- ✅ 理解项目基本结构
- ✅ 创建第一个Vue组件
- ✅ 理解响应式数据基础

### 遇到的问题
> 记录你遇到的问题...

### 解决方案
> 记录问题的解决方案...

### 下一步计划
- 深入学习响应式系统（ref vs reactive）
- 学习更多模板语法
- 实践待办事项列表项目

## 📚 参考资源
- [Vue3官方文档](https://cn.vuejs.org/)
- [Vite官方文档](https://cn.vitejs.dev/)
- [Vue3设计与实现](https://www.ituring.com.cn/book/2953)

## 💬 互动提示
完成环境搭建后，可以对Claude说：
- "我完成了环境搭建，帮我解释一下代码"
- "我想深入了解ref的原理"
- "能给我更多练习题吗？"
- "我遇到了[具体错误]，帮我分析一下"

---
**下一节**: [02-响应式基础](./02-reactive-basics.md)