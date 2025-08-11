# Day 2：响应式进阶

## 📚 学习目标
- [ ] 掌握 reactive 创建对象响应式
- [ ] 理解 ref vs reactive 的区别和选择
- [ ] 掌握 computed 计算属性的使用
- [ ] 掌握 watch 和 watchEffect 侦听器
- [ ] 理解响应式原理（Proxy）
- [ ] 掌握响应式工具函数（toRef、toRefs等）

## 📝 学习内容

### 1. reactive - 对象响应式

#### 1.1 基本概念
`reactive` 是 Vue3 中用于创建响应式对象的函数。与 `ref` 不同，`reactive` 专门用于对象类型。

#### 1.2 基本用法

```javascript
import { reactive } from 'vue'

// 创建响应式对象
const state = reactive({
  count: 0,
  message: 'Hello',
  user: {
    name: '张三',
    age: 25
  }
})

// 直接访问和修改，不需要 .value
state.count++
state.message = 'World'
state.user.age = 26
```

#### 1.3 reactive vs ref 详细对比

| 特性 | ref | reactive |
|------|-----|----------|
| **适用类型** | 基本类型 + 对象 | 仅对象类型 |
| **访问方式** | 需要 `.value` | 直接访问 |
| **解构** | 保持响应式 | 失去响应式 |
| **类型推导** | 更好 | 一般 |
| **使用场景** | 单个值 | 多个相关值 |

#### 1.4 实际示例

```vue
<script setup>
import { reactive, ref } from 'vue'

// ❌ 错误用法：reactive 不能用于基本类型
// const count = reactive(0)  // 不会生效

// ✅ 正确用法：reactive 用于对象
const userState = reactive({
  profile: {
    name: '张三',
    age: 25,
    email: 'zhangsan@example.com'
  },
  settings: {
    theme: 'dark',
    notifications: true
  }
})

// ref 适合单个值
const isLoading = ref(false)
const errorMessage = ref('')

// 修改 reactive 对象
function updateProfile() {
  userState.profile.name = '李四'  // 直接修改，无需 .value
  userState.profile.age++
}

// 修改 ref
function setLoading(value) {
  isLoading.value = value  // 需要 .value
}
</script>

<template>
  <div>
    <!-- reactive 对象直接访问属性 -->
    <p>用户：{{ userState.profile.name }}</p>
    <p>年龄：{{ userState.profile.age }}</p>
    
    <!-- ref 在模板中自动解包 -->
    <p v-if="isLoading">加载中...</p>
    <p v-if="errorMessage">{{ errorMessage }}</p>
  </div>
</template>
```

#### 1.5 注意事项

##### 解构会失去响应式
```javascript
const state = reactive({ count: 0 })

// ❌ 错误：解构后失去响应式
const { count } = state
count++ // 不会触发更新

// ✅ 正确：使用 toRefs 保持响应式
import { toRefs } from 'vue'
const { count } = toRefs(state)
count.value++ // 会触发更新
```

##### 不能整体替换
```javascript
let state = reactive({ count: 0 })

// ❌ 错误：整体替换会失去响应式
state = reactive({ count: 1 })  // 组件不会更新

// ✅ 正确：修改属性
state.count = 1  // 会触发更新
```

### 2. computed - 计算属性

#### 2.1 基本概念
`computed` 创建一个响应式的计算属性，它会根据依赖自动更新，并且具有缓存特性。

#### 2.2 基础用法

```vue
<script setup>
import { ref, computed } from 'vue'

const firstName = ref('张')
const lastName = ref('三')

// 只读计算属性
const fullName = computed(() => {
  return firstName.value + lastName.value
})

// 可写计算属性
const fullNameWritable = computed({
  get() {
    return firstName.value + lastName.value
  },
  set(newValue) {
    const names = newValue.split(' ')
    firstName.value = names[0] || ''
    lastName.value = names[1] || ''
  }
})
</script>
```

#### 2.3 computed vs methods 对比

```vue
<script setup>
import { ref, computed } from 'vue'

const count = ref(0)

// computed：有缓存，依赖不变不会重新计算
const doubleComputed = computed(() => {
  console.log('computed 执行了')
  return count.value * 2
})

// method：每次调用都会执行
function doubleMethod() {
  console.log('method 执行了')
  return count.value * 2
}
</script>

<template>
  <div>
    <!-- computed 多次访问只执行一次 -->
    <p>{{ doubleComputed }}</p>
    <p>{{ doubleComputed }}</p>
    <p>{{ doubleComputed }}</p>
    
    <!-- method 每次调用都执行 -->
    <p>{{ doubleMethod() }}</p>
    <p>{{ doubleMethod() }}</p>
    <p>{{ doubleMethod() }}</p>
  </div>
</template>
```

#### 2.4 实际应用场景

```vue
<script setup>
import { reactive, computed } from 'vue'

// 购物车示例
const cart = reactive({
  items: [
    { id: 1, name: '苹果', price: 10, quantity: 2 },
    { id: 2, name: '香蕉', price: 5, quantity: 3 },
    { id: 3, name: '橙子', price: 8, quantity: 1 }
  ],
  discount: 0.9  // 9折
})

// 计算总价
const totalPrice = computed(() => {
  return cart.items.reduce((sum, item) => {
    return sum + item.price * item.quantity
  }, 0)
})

// 计算折后价
const finalPrice = computed(() => {
  return (totalPrice.value * cart.discount).toFixed(2)
})

// 计算总数量
const totalQuantity = computed(() => {
  return cart.items.reduce((sum, item) => sum + item.quantity, 0)
})

// 筛选计算
const searchTerm = ref('')
const filteredItems = computed(() => {
  if (!searchTerm.value) return cart.items
  return cart.items.filter(item => 
    item.name.includes(searchTerm.value)
  )
})
</script>
```

### 3. watch - 侦听器

#### 3.1 基本概念
`watch` 用于监听响应式数据的变化，并执行副作用操作。

#### 3.2 监听单个源

```javascript
import { ref, watch } from 'vue'

const count = ref(0)

// 监听 ref
watch(count, (newValue, oldValue) => {
  console.log(`count 从 ${oldValue} 变为 ${newValue}`)
})

// 监听 reactive 的某个属性
const state = reactive({ count: 0 })
watch(
  () => state.count,  // getter 函数
  (newValue, oldValue) => {
    console.log(`state.count 从 ${oldValue} 变为 ${newValue}`)
  }
)
```

#### 3.3 监听多个源

```javascript
const firstName = ref('张')
const lastName = ref('三')

// 监听多个源
watch(
  [firstName, lastName],
  ([newFirst, newLast], [oldFirst, oldLast]) => {
    console.log(`姓名从 ${oldFirst}${oldLast} 变为 ${newFirst}${newLast}`)
  }
)
```

#### 3.4 配置选项

```javascript
const data = ref({ count: 0 })

watch(data, (newValue, oldValue) => {
  console.log('data 变化了')
}, {
  deep: true,      // 深度监听
  immediate: true, // 立即执行
  flush: 'post'    // 调整回调时机
})

// 停止侦听
const stop = watch(source, callback)
// 在需要时停止
stop()
```

#### 3.5 实际应用示例

```vue
<script setup>
import { ref, reactive, watch } from 'vue'

// 表单自动保存
const formData = reactive({
  title: '',
  content: '',
  tags: []
})

// 监听表单变化，自动保存到本地存储
watch(
  formData,
  (newData) => {
    localStorage.setItem('draft', JSON.stringify(newData))
    console.log('草稿已自动保存')
  },
  { 
    deep: true,  // 深度监听对象内部变化
    debounce: 1000  // 防抖，1秒后才保存
  }
)

// 搜索功能
const searchQuery = ref('')
const searchResults = ref([])

watch(searchQuery, async (newQuery) => {
  if (newQuery.length < 2) {
    searchResults.value = []
    return
  }
  
  // 模拟 API 调用
  const results = await searchAPI(newQuery)
  searchResults.value = results
})
</script>
```

### 4. watchEffect - 自动依赖收集

#### 4.1 基本概念
`watchEffect` 会自动收集依赖，不需要明确指定要监听的数据。

#### 4.2 基本用法

```javascript
import { ref, watchEffect } from 'vue'

const count = ref(0)
const message = ref('Hello')

// 自动追踪使用的响应式数据
watchEffect(() => {
  // 这个函数会在 count 或 message 变化时重新执行
  console.log(`count: ${count.value}, message: ${message.value}`)
})
```

#### 4.3 watch vs watchEffect

| 特性 | watch | watchEffect |
|------|-------|-------------|
| **依赖追踪** | 显式指定 | 自动收集 |
| **立即执行** | 配置 immediate | 默认立即执行 |
| **旧值访问** | 可以访问 | 不能访问 |
| **使用场景** | 需要旧值或精确控制 | 简单的副作用 |

#### 4.4 清理副作用

```javascript
watchEffect((onCleanup) => {
  const timer = setInterval(() => {
    console.log('定时器执行')
  }, 1000)
  
  // 清理函数
  onCleanup(() => {
    clearInterval(timer)
  })
})
```

### 5. 响应式工具函数

#### 5.1 toRef 和 toRefs

```javascript
import { reactive, toRef, toRefs } from 'vue'

const state = reactive({
  foo: 1,
  bar: 2
})

// toRef: 为响应式对象的属性创建 ref
const fooRef = toRef(state, 'foo')
fooRef.value++  // state.foo 也会更新

// toRefs: 将响应式对象转换为普通对象，每个属性都是 ref
const stateRefs = toRefs(state)
// 现在可以解构而不失去响应式
const { foo, bar } = stateRefs
```

#### 5.2 其他工具函数

```javascript
import { 
  isRef, 
  unref, 
  isReactive, 
  isProxy,
  shallowRef,
  readonly
} from 'vue'

// isRef: 检查是否为 ref
const count = ref(0)
console.log(isRef(count))  // true

// unref: 解包 ref
const value = unref(count)  // 相当于 isRef(count) ? count.value : count

// shallowRef: 浅层响应式
const shallow = shallowRef({ count: 0 })
shallow.value.count++  // 不会触发更新
shallow.value = { count: 1 }  // 会触发更新

// readonly: 创建只读代理
const original = reactive({ count: 0 })
const copy = readonly(original)
// copy.count++  // 警告：不能修改只读属性
```

### 6. 响应式原理深入

#### 6.1 Proxy 实现原理

Vue3 使用 Proxy 实现响应式：

```javascript
// 简化的响应式实现
function reactive(target) {
  return new Proxy(target, {
    get(target, key, receiver) {
      // 依赖收集
      track(target, key)
      return Reflect.get(target, key, receiver)
    },
    set(target, key, value, receiver) {
      const result = Reflect.set(target, key, value, receiver)
      // 触发更新
      trigger(target, key)
      return result
    }
  })
}
```

#### 6.2 响应式系统流程

```
1. 创建响应式数据
   ↓
2. 组件渲染时访问数据
   ↓
3. getter 触发，收集依赖
   ↓
4. 数据修改时
   ↓
5. setter 触发，通知更新
   ↓
6. 组件重新渲染
```

## 💻 实践项目：增强版待办事项列表

### 功能需求
1. 添加/删除待办项
2. 标记完成/未完成
3. 筛选功能（全部/进行中/已完成）
4. 统计信息（总数/完成数/完成率）
5. 搜索功能
6. 本地存储
7. 批量操作

### 技术要点
- 使用 `reactive` 管理待办列表
- 使用 `computed` 计算统计信息和筛选结果
- 使用 `watch` 实现自动保存
- 使用 `watchEffect` 实现搜索
- 使用 `toRefs` 解构状态

### 完整代码实现

```vue
<script setup>
import { reactive, computed, watch, watchEffect, toRefs } from 'vue'

// 待办事项状态
const state = reactive({
  todos: [],
  filter: 'all',  // all | active | completed
  searchQuery: '',
  newTodoText: ''
})

// 从本地存储加载数据
const loadTodos = () => {
  const saved = localStorage.getItem('todos')
  if (saved) {
    state.todos = JSON.parse(saved)
  }
}

// 初始化加载
loadTodos()

// 添加待办项
const addTodo = () => {
  if (!state.newTodoText.trim()) return
  
  state.todos.push({
    id: Date.now(),
    text: state.newTodoText,
    completed: false,
    createdAt: new Date().toISOString()
  })
  state.newTodoText = ''
}

// 删除待办项
const removeTodo = (id) => {
  const index = state.todos.findIndex(todo => todo.id === id)
  if (index > -1) {
    state.todos.splice(index, 1)
  }
}

// 切换完成状态
const toggleTodo = (id) => {
  const todo = state.todos.find(t => t.id === id)
  if (todo) {
    todo.completed = !todo.completed
  }
}

// 清除已完成
const clearCompleted = () => {
  state.todos = state.todos.filter(todo => !todo.completed)
}

// 全选/取消全选
const toggleAll = () => {
  const allCompleted = state.todos.every(todo => todo.completed)
  state.todos.forEach(todo => {
    todo.completed = !allCompleted
  })
}

// 计算属性：过滤后的待办项
const filteredTodos = computed(() => {
  let result = state.todos
  
  // 根据完成状态筛选
  if (state.filter === 'active') {
    result = result.filter(todo => !todo.completed)
  } else if (state.filter === 'completed') {
    result = result.filter(todo => todo.completed)
  }
  
  // 根据搜索词筛选
  if (state.searchQuery) {
    result = result.filter(todo => 
      todo.text.toLowerCase().includes(state.searchQuery.toLowerCase())
    )
  }
  
  return result
})

// 计算属性：统计信息
const stats = computed(() => {
  const total = state.todos.length
  const completed = state.todos.filter(t => t.completed).length
  const active = total - completed
  const completionRate = total > 0 ? Math.round((completed / total) * 100) : 0
  
  return {
    total,
    completed,
    active,
    completionRate
  }
})

// 监听待办项变化，自动保存
watch(
  () => state.todos,
  (todos) => {
    localStorage.setItem('todos', JSON.stringify(todos))
  },
  { deep: true }
)

// 使用 toRefs 解构
const { todos, filter, searchQuery, newTodoText } = toRefs(state)
</script>

<template>
  <div class="todo-app">
    <h1>📝 增强版待办事项</h1>
    
    <!-- 统计信息 -->
    <div class="stats">
      <span>总计: {{ stats.total }}</span>
      <span>完成: {{ stats.completed }}</span>
      <span>进行中: {{ stats.active }}</span>
      <span>完成率: {{ stats.completionRate }}%</span>
    </div>
    
    <!-- 输入框 -->
    <div class="input-section">
      <input 
        v-model="newTodoText"
        @keyup.enter="addTodo"
        placeholder="添加新的待办事项..."
      >
      <button @click="addTodo">添加</button>
    </div>
    
    <!-- 搜索框 -->
    <input 
      v-model="searchQuery"
      placeholder="搜索待办事项..."
      class="search-input"
    >
    
    <!-- 筛选按钮 -->
    <div class="filters">
      <button 
        @click="filter = 'all'"
        :class="{ active: filter === 'all' }"
      >全部</button>
      <button 
        @click="filter = 'active'"
        :class="{ active: filter === 'active' }"
      >进行中</button>
      <button 
        @click="filter = 'completed'"
        :class="{ active: filter === 'completed' }"
      >已完成</button>
    </div>
    
    <!-- 批量操作 -->
    <div class="batch-actions">
      <button @click="toggleAll">全选/取消</button>
      <button @click="clearCompleted">清除已完成</button>
    </div>
    
    <!-- 待办列表 -->
    <ul class="todo-list">
      <li 
        v-for="todo in filteredTodos" 
        :key="todo.id"
        :class="{ completed: todo.completed }"
      >
        <input 
          type="checkbox"
          :checked="todo.completed"
          @change="toggleTodo(todo.id)"
        >
        <span>{{ todo.text }}</span>
        <button @click="removeTodo(todo.id)">删除</button>
      </li>
    </ul>
    
    <p v-if="filteredTodos.length === 0" class="empty">
      暂无待办事项
    </p>
  </div>
</template>

<style scoped>
.todo-app {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

.stats {
  display: flex;
  justify-content: space-around;
  padding: 15px;
  background: #f5f5f5;
  border-radius: 8px;
  margin-bottom: 20px;
}

.stats span {
  font-weight: bold;
}

.input-section {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.input-section input {
  flex: 1;
  padding: 10px;
  border: 2px solid #ddd;
  border-radius: 4px;
}

.search-input {
  width: 100%;
  padding: 10px;
  border: 2px solid #ddd;
  border-radius: 4px;
  margin-bottom: 20px;
}

.filters {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.filters button {
  padding: 8px 16px;
  border: 1px solid #ddd;
  background: white;
  cursor: pointer;
  border-radius: 4px;
}

.filters button.active {
  background: #42b883;
  color: white;
  border-color: #42b883;
}

.batch-actions {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.todo-list {
  list-style: none;
  padding: 0;
}

.todo-list li {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-bottom: 10px;
}

.todo-list li.completed span {
  text-decoration: line-through;
  opacity: 0.6;
}

.todo-list li span {
  flex: 1;
}

.empty {
  text-align: center;
  color: #999;
  padding: 40px;
}
</style>
```

## ✅ 今日练习

### 练习1：购物车系统
创建一个购物车组件，要求：
1. 使用 `reactive` 管理商品列表
2. 使用 `computed` 计算总价、总数量、优惠金额
3. 使用 `watch` 监听购物车变化，同步到后端
4. 实现增加、减少商品数量
5. 实现删除商品
6. 实现优惠券功能

### 练习2：实时搜索
创建一个实时搜索组件，要求：
1. 使用 `ref` 管理搜索关键词
2. 使用 `watchEffect` 实现自动搜索
3. 实现防抖功能
4. 显示搜索结果
5. 高亮匹配的关键词
6. 显示搜索历史

### 练习3：表单验证
创建一个用户注册表单，要求：
1. 使用 `reactive` 管理表单数据
2. 使用 `computed` 实现实时验证
3. 使用 `watch` 实现字段联动
4. 验证规则：用户名长度、邮箱格式、密码强度
5. 显示错误提示
6. 提交前的整体验证

## 🤔 思考题

1. **reactive 和 ref 应该如何选择？**
   - 什么场景用 ref？
   - 什么场景用 reactive？
   - 能否混用？

2. **computed 的缓存机制是如何工作的？**
   - 什么时候会重新计算？
   - 与 methods 的性能差异？
   - 什么时候不适合用 computed？

3. **watch 和 watchEffect 的使用场景？**
   - 各自的优势是什么？
   - 什么时候用 watch？
   - 什么时候用 watchEffect？

4. **为什么解构 reactive 对象会失去响应式？**
   - JavaScript 的值传递机制
   - Proxy 的工作原理
   - toRefs 是如何解决这个问题的？

## 📊 学习总结

### 今日收获
- ✅ 理解了 reactive 创建对象响应式
- ✅ 掌握了 ref vs reactive 的选择
- ✅ 学会了 computed 计算属性的使用
- ✅ 掌握了 watch 和 watchEffect 的区别
- ✅ 理解了响应式系统的原理
- ✅ 学会了响应式工具函数的使用

### 重点回顾
1. **reactive** 用于对象，直接访问属性，解构会失去响应式
2. **computed** 有缓存，依赖不变不重新计算
3. **watch** 显式指定监听源，可以访问旧值
4. **watchEffect** 自动收集依赖，立即执行
5. **toRefs** 解决解构失去响应式的问题

### 下一步计划
- 学习组件系统（props、emit、slot）
- 学习组合式函数（composables）
- 学习生命周期钩子
- 实践更复杂的项目

## 📚 参考资源
- [Vue3 官方文档 - 响应式基础](https://cn.vuejs.org/guide/essentials/reactivity-fundamentals.html)
- [Vue3 官方文档 - 计算属性](https://cn.vuejs.org/guide/essentials/computed.html)
- [Vue3 官方文档 - 侦听器](https://cn.vuejs.org/guide/essentials/watchers.html)
- [Vue3 设计与实现](https://www.ituring.com.cn/book/2953)

## 💬 互动提示
完成学习后，可以对Claude说：
- "我完成了 reactive 的学习，有些地方不太理解"
- "能详细解释一下 computed 的缓存原理吗？"
- "watch 和 watchEffect 的区别还是不太清楚"
- "帮我检查一下我的代码实现"

---
**下一节**: [03-组件系统](./03-component-system.md)