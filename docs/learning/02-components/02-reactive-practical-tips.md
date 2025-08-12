# Day 2 响应式实践技巧

## 📌 实际工作中的常见问题

这份文档记录了在实际项目开发中使用 Vue3 响应式系统时遇到的常见问题和解决方案。

---

## 💼 问题1：为什么会解构 reactive？

> "平常工作中会要解构reactive，我没事儿解构他干嘛呢？"

确实，不会故意去解构。但在实际工作中，有些场景会"不小心"或"不得不"解构。

### 1. **从组合式函数返回状态时**

这是最容易踩坑的地方，尤其是从 Vue2 或其他框架迁移过来的开发者。

```javascript
// ❌ 容易犯的错误
function useUser() {
  const state = reactive({
    name: '张三',
    age: 25,
    isVip: false
  })
  
  // 想返回多个值，习惯性解构
  return {
    ...state,  // ❌ 解构了，失去响应式
    updateName: (name) => state.name = name
  }
}

// 使用时
const { name, age, updateName } = useUser()
// name 和 age 不是响应式的！
```

**✅ 正确做法：**

```javascript
// 方案1：返回整个对象
function useUser() {
  const state = reactive({
    name: '张三',
    age: 25,
    isVip: false
  })
  
  return { 
    state,
    updateName: (name) => state.name = name
  }
}

// 使用时
const { state, updateName } = useUser()
console.log(state.name)  // 响应式

// 方案2：使用 toRefs
function useUser() {
  const state = reactive({
    name: '张三',
    age: 25,
    isVip: false
  })
  
  return {
    ...toRefs(state),  // 转换为 ref
    updateName: (name) => state.name = name
  }
}

// 使用时
const { name, age, updateName } = useUser()
console.log(name.value)  // 响应式
```

### 2. **传递给子组件 props 时**

在父子组件通信时，很容易习惯性地解构。

```vue
<!-- 父组件 -->
<script setup>
const formData = reactive({
  username: '',
  email: '',
  password: ''
})
</script>

<template>
  <!-- ❌ 解构传递（很多人会这么写） -->
  <UserForm 
    :username="formData.username"
    :email="formData.email"
    :password="formData.password"
    @update="handleUpdate"
  />
  
  <!-- ✅ 更好的做法：整体传递 -->
  <UserForm 
    :form-data="formData"
    @update="handleUpdate"
  />
  
  <!-- ✅ 或者使用 v-model -->
  <UserForm v-model="formData" />
</template>
```

### 3. **在模板中简化访问**

为了避免在模板中写很长的路径，开发者常常想要解构。

```vue
<script setup>
const state = reactive({
  user: { 
    name: '张三', 
    age: 25,
    address: {
      city: '北京',
      district: '朝阳区'
    }
  },
  loading: false,
  error: null
})

// ❌ 为了在模板中方便使用而解构
const { user, loading, error } = state  // 失去响应式！

// ✅ 方案1：使用 toRefs
const { user, loading, error } = toRefs(state)

// ✅ 方案2：使用 computed
const user = computed(() => state.user)
const loading = computed(() => state.loading)
</script>

<template>
  <!-- 现在可以直接用 user 而不是 state.user -->
  <div v-if="loading">加载中...</div>
  <div v-else>
    <p>{{ user.name }}</p>
    <p>{{ user.address.city }}</p>
  </div>
</template>
```

### 4. **团队协作时的代码风格冲突**

不同背景的开发者有不同的编码习惯。

```javascript
// 团队成员A（来自 React）写的代码
const userStore = reactive({
  profile: {},
  settings: {},
  permissions: []
})

// 团队成员B（习惯解构）想用其中一部分
// ❌ 习惯性地解构
const { profile, settings } = userStore  // 失去响应式

// ✅ 正确做法
// 方案1：保持引用
const profile = toRef(userStore, 'profile')
const settings = toRef(userStore, 'settings')

// 方案2：制定团队规范，优先使用 ref
const profile = ref({})
const settings = ref({})
const permissions = ref([])
```

### 5. **从 Pinia store 获取状态**

Pinia 通常会帮我们处理好响应式，但也要注意使用方式。

```javascript
// Pinia store 定义
export const useUserStore = defineStore('user', () => {
  const state = reactive({
    userInfo: {},
    token: '',
    permissions: []
  })
  
  // ✅ store 通常会用 toRefs 导出
  return {
    ...toRefs(state),
    // actions...
    login,
    logout
  }
})

// 组件中使用
const store = useUserStore()

// ✅ 这样解构是安全的（因为 store 已经处理了）
const { userInfo, token } = storeToRefs(store)

// ❌ 不要这样（会失去响应式）
const { userInfo, token } = store
```

### 6. **处理 API 响应数据**

从后端获取数据后的处理也容易出问题。

```javascript
// ❌ 错误的做法
const state = reactive({
  user: null
})

async function fetchUser() {
  const res = await api.getUser()
  // 解构赋值，破坏了响应式
  const { name, age, email } = res.data
  state.user = { name, age, email }  // 这样还是响应式的
  
  // 但如果这样就有问题了
  Object.assign(state, res.data)  // 可能会有问题
}

// ✅ 正确的做法
const state = reactive({
  user: null
})

async function fetchUser() {
  const res = await api.getUser()
  state.user = res.data  // 直接赋值整个对象
}

// 或者使用 ref
const user = ref(null)
async function fetchUser() {
  user.value = await api.getUser()  // 更简单清晰
}
```

---

## 🔍 问题2：toRefs 对嵌套对象的限制

> "如果对象特别复杂，有嵌套，用 toRefs 好使吗？"

**答案：不好使！toRefs 只处理第一层。**

### 问题演示

```javascript
const state = reactive({
  count: 0,                    // 第一层
  user: {                      // 第一层
    profile: {                 // 第二层（嵌套）
      name: '张三',            // 第三层
      age: 25,
      address: {               // 第三层
        city: '北京',          // 第四层
        district: '朝阳区'
      }
    },
    settings: {                // 第二层
      theme: 'dark',
      notifications: true
    }
  }
})

// 使用 toRefs
const refs = toRefs(state)

// 分析结果
console.log(isRef(refs.count))                    // ✅ true - 是 ref
console.log(isRef(refs.user))                     // ✅ true - 是 ref
console.log(isRef(refs.user.value))               // ❌ false - 不是 ref，是 reactive 对象
console.log(isRef(refs.user.value.profile))       // ❌ false - 不是 ref
console.log(isRef(refs.user.value.profile.name))  // ❌ false - 不是 ref
```

### 图解 toRefs 的效果

```
原始 reactive 对象：
{
  count: 0,              ← 第一层
  user: {                ← 第一层
    profile: {           ← 第二层（嵌套）
      name: '张三',      ← 第三层
      age: 25
    }
  }
}

toRefs 转换后：
{
  count: Ref<0>,         ← ✅ 转换为 ref
  user: Ref<{            ← ✅ 转换为 ref，但值还是 reactive
    profile: {           ← ⚠️ 还是 reactive，没有转换为 ref
      name: '张三',      ← ⚠️ 普通属性
      age: 25
    }
  }>
}
```

### 实际问题场景

```javascript
const state = reactive({
  user: {
    profile: {
      name: '张三',
      age: 25
    }
  }
})

const refs = toRefs(state)

// ✅ 第一层解构是安全的
const { user } = refs
console.log(user.value.profile.name)  // 响应式

// ❌ 但继续解构嵌套对象会失去响应式
const { profile } = user.value  // profile 不是 ref
profile.name = '李四'  // 可能不会触发更新

// ❌ 更深层的解构更危险
const { name, age } = user.value.profile
name = '李四'  // 完全无效，name 只是个字符串
```

### 解决方案

#### 方案1：手动递归处理（不推荐，太复杂）

```javascript
// 深度转换为 refs（仅作示例，实际不建议使用）
function deepToRefs(obj, rootObj = obj, path = []) {
  const result = {}
  
  for (const key in obj) {
    const fullPath = [...path, key]
    const val = obj[key]
    
    if (typeof val === 'object' && val !== null && !isRef(val)) {
      // 递归处理嵌套对象
      result[key] = deepToRefs(val, rootObj, fullPath)
    } else {
      // 创建 ref 指向原始路径
      result[key] = customRef((track, trigger) => ({
        get() {
          track()
          let current = rootObj
          for (const p of fullPath) {
            current = current[p]
          }
          return current
        },
        set(newVal) {
          let current = rootObj
          for (let i = 0; i < fullPath.length - 1; i++) {
            current = current[fullPath[i]]
          }
          current[fullPath[fullPath.length - 1]] = newVal
          trigger()
        }
      }))
    }
  }
  
  return result
}
```

#### 方案2：只转换需要的路径（推荐）

```javascript
const state = reactive({
  user: {
    profile: {
      name: '张三',
      age: 25,
      address: {
        city: '北京'
      }
    }
  }
})

// 只转换实际需要的深层属性
const userName = toRef(state.user.profile, 'name')
const userAge = toRef(state.user.profile, 'age')
const userCity = toRef(state.user.profile.address, 'city')

// 现在这些都是响应式的 ref
userName.value = '李四'  // ✅ 响应式
userCity.value = '上海'  // ✅ 响应式
```

#### 方案3：保持对象引用（最推荐）

```javascript
const state = reactive({
  user: {
    profile: {
      name: '张三',
      age: 25
    }
  }
})

// 不要解构，保持引用
const user = toRef(state, 'user')

// 使用时通过路径访问
function updateName(newName) {
  user.value.profile.name = newName  // ✅ 响应式
}

// 在模板中
<template>
  <div>{{ user.profile.name }}</div>
</template>
```

#### 方案4：使用 computed 包装（灵活方案）

```javascript
const state = reactive({
  user: {
    profile: {
      name: '张三',
      age: 25
    }
  }
})

// 为常用的深层属性创建 computed
const userProfile = computed(() => state.user.profile)
const userName = computed({
  get: () => state.user.profile.name,
  set: (val) => state.user.profile.name = val
})

// 可读写的响应式引用
userName.value = '李四'  // ✅ 响应式
```

#### 方案5：扁平化数据结构（架构层面）

```javascript
// ❌ 不好：深层嵌套
const state = reactive({
  user: {
    profile: {
      personal: {
        name: '张三',
        age: 25
      },
      contact: {
        email: 'zhang@example.com',
        phone: '13800138000'
      }
    },
    settings: {
      preferences: {
        theme: 'dark',
        language: 'zh-CN'
      }
    }
  }
})

// ✅ 更好：扁平化结构
const userProfile = reactive({
  name: '张三',
  age: 25
})

const userContact = reactive({
  email: 'zhang@example.com',
  phone: '13800138000'
})

const userSettings = reactive({
  theme: 'dark',
  language: 'zh-CN'
})

// 如果需要组合，使用 computed
const fullUser = computed(() => ({
  profile: userProfile,
  contact: userContact,
  settings: userSettings
}))
```

#### 方案6：使用 ref 包装整个对象（简单直接）

```javascript
// 对于复杂嵌套对象，直接用 ref 包装整体
const user = ref({
  profile: {
    name: '张三',
    age: 25,
    address: {
      city: '北京',
      district: '朝阳区'
    }
  }
})

// 修改嵌套属性
user.value.profile.name = '李四'  // ✅ 响应式
user.value.profile.address.city = '上海'  // ✅ 响应式

// 整体替换也很方便
user.value = await fetchUserData()  // ✅ 整体替换
```

### 性能考虑

#### 使用 shallowReactive 优化性能

```javascript
import { shallowReactive, reactive } from 'vue'

// 只有第一层是响应式的
const state = shallowReactive({
  count: 0,           // ✅ 响应式
  user: {            // ⚠️ user 本身响应式，但内部不是
    name: '张三',    // ❌ 修改不会触发更新
    age: 25
  }
})

// 修改第一层 - 触发更新
state.count++  // ✅ 
state.user = { name: '李四', age: 26 }  // ✅ 

// 修改嵌套属性 - 不触发更新
state.user.name = '王五'  // ❌ 不会触发视图更新

// 如果某个嵌套对象需要响应式，单独处理
state.settings = reactive({
  theme: 'dark',
  notifications: true
})
```

#### 使用 markRaw 标记非响应式

```javascript
import { reactive, markRaw } from 'vue'

const state = reactive({
  user: {
    name: '张三',
    // 大型对象不需要响应式
    bigData: markRaw({
      // 几万条数据...
    })
  }
})

// bigData 的修改不会触发更新，提升性能
state.user.bigData.someProperty = 'value'  // 不触发更新
```

---

## 🎯 最佳实践总结

### 1. **数据结构设计原则**

```javascript
// ✅ 好的设计
// 1. 扁平化
const userName = ref('张三')
const userAge = ref(25)

// 2. 相关数据才组合
const formData = reactive({
  username: '',
  password: ''
})

// 3. 复杂对象用 ref
const complexData = ref({
  deep: { nested: { value: 1 } }
})

// ❌ 避免的设计
const state = reactive({
  deep: {
    nested: {
      more: {
        nested: {
          value: 1
        }
      }
    }
  }
})
```

### 2. **选择指南**

| 场景 | 推荐方案 | 原因 |
|------|---------|------|
| 基本类型 | `ref` | 天然支持，无需额外处理 |
| 单个对象 | `ref` | 可以整体替换，灵活性高 |
| 表单数据 | `reactive` + `toRefs` | 字段相关，适合组合 |
| 深层嵌套 | `ref` 或 分割成多个 | 避免 toRefs 的限制 |
| 大型数据 | `shallowRef` 或 `markRaw` | 性能优化 |
| 组合式函数返回值 | `ref` 或 `toRefs` | 保持响应式 |

### 3. **代码规范建议**

```javascript
// composables/useUser.js
export function useUser() {
  // ✅ 1. 优先使用 ref
  const userName = ref('')
  const userAge = ref(0)
  
  // ✅ 2. 相关数据用 reactive，但要 toRefs
  const userForm = reactive({
    email: '',
    phone: ''
  })
  
  // ✅ 3. 复杂对象整体用 ref
  const userProfile = ref(null)
  
  // ✅ 4. 返回时保持响应式
  return {
    userName,
    userAge,
    ...toRefs(userForm),
    userProfile,
    // methods
    updateProfile
  }
}
```

### 4. **常见错误检查清单**

- [ ] 是否解构了 `reactive` 对象？
- [ ] 是否对 `toRefs` 后的嵌套对象再次解构？
- [ ] 组合式函数的返回值是否保持响应式？
- [ ] props 传递时是否不必要地解构？
- [ ] 从 API 获取数据后的赋值方式是否正确？
- [ ] 是否在不需要响应式的大数据上使用了 `reactive`？

---

## 📚 延伸阅读

- [Vue3 官方文档 - 响应式基础](https://cn.vuejs.org/guide/essentials/reactivity-fundamentals.html)
- [Vue3 深度响应性](https://cn.vuejs.org/guide/extras/reactivity-in-depth.html)
- [Vue3 性能优化](https://cn.vuejs.org/guide/best-practices/performance.html)

## 💭 思考

1. 为什么 Vue3 不默认提供 `deepToRefs` 功能？
2. 什么时候该牺牲响应式来换取性能？
3. 团队中如何统一响应式数据的使用规范？

记住：**保持简单** 永远是最好的方案。如果数据结构变得太复杂，先考虑是否能简化设计，而不是寻找复杂的技术方案。