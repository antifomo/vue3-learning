<script setup>
// Day 2 - reactive 基础示例
import { reactive, ref, toRefs } from 'vue'

// ==================== reactive vs ref 对比 ====================

// 1. ref 用于基本类型和对象
const refCount = ref(0)
const refUser = ref({ name: '张三', age: 25 })

// 2. reactive 只用于对象
const reactiveState = reactive({
  count: 0,
  user: {
    name: '李四',
    age: 30,
    address: {
      city: '北京',
      district: '朝阳区'
    }
  },
  hobbies: ['读书', '游泳', '编程']
})

// ==================== 操作对比 ====================

// ref 操作：需要 .value
function incrementRef() {
  refCount.value++
  refUser.value.age++
}

// reactive 操作：直接访问
function incrementReactive() {
  reactiveState.count++
  reactiveState.user.age++
}

// ==================== 解构问题演示 ====================

// ❌ 错误：直接解构会失去响应式
let { count: wrongCount } = reactiveState  // 这是普通变量，不是响应式

// ✅ 正确：使用 toRefs 保持响应式
const { count: rightCount, user } = toRefs(reactiveState)

function testDestructuring() {
  wrongCount++  // 不会更新视图
  console.log('错误解构后的值:', wrongCount)
  console.log('原始响应式值:', reactiveState.count)
  
  rightCount.value++  // 会更新视图
  console.log('正确解构后的值:', rightCount.value)
  console.log('原始响应式值:', reactiveState.count)
}

// ==================== 深层响应式 ====================

function updateNested() {
  // reactive 是深层响应式的
  reactiveState.user.address.city = '上海'  // 会触发更新
  reactiveState.user.address.district = '浦东新区'
}

// ==================== 数组操作 ====================

function addHobby() {
  const newHobby = prompt('请输入新爱好：')
  if (newHobby) {
    reactiveState.hobbies.push(newHobby)  // 响应式
  }
}

function removeHobby(index) {
  reactiveState.hobbies.splice(index, 1)  // 响应式
}

// ==================== 整体替换问题 ====================

let tempState = reactive({ temp: 0 })

function wrongReplace() {
  // ❌ 错误：整体替换会失去响应式连接
  tempState = reactive({ temp: 100 })  // 组件不会更新
  console.log('替换后的值:', tempState.temp)
}

function correctReplace() {
  // ✅ 正确：修改属性而不是替换整个对象
  tempState.temp = 200
}

// ==================== 响应式检查 ====================
import { isReactive, isRef } from 'vue'

const checkStatus = () => {
  console.log('=== 响应式检查 ===')
  console.log('refCount 是 ref?', isRef(refCount))
  console.log('reactiveState 是 reactive?', isReactive(reactiveState))
  console.log('wrongCount 是响应式?', isRef(wrongCount) || isReactive(wrongCount))
  console.log('rightCount 是 ref?', isRef(rightCount))
}
</script>

<template>
  <div class="reactive-basics">
    <h2>📚 Reactive 基础示例</h2>
    
    <!-- ref vs reactive 对比 -->
    <div class="section">
      <h3>1. ref vs reactive 对比</h3>
      
      <div class="comparison">
        <div class="ref-demo card">
          <h4>Ref 示例</h4>
          <p>计数: {{ refCount }}</p>
          <p>用户: {{ refUser.name }} ({{ refUser.age }}岁)</p>
          <button @click="incrementRef">Ref +1</button>
        </div>
        
        <div class="reactive-demo card">
          <h4>Reactive 示例</h4>
          <p>计数: {{ reactiveState.count }}</p>
          <p>用户: {{ reactiveState.user.name }} ({{ reactiveState.user.age }}岁)</p>
          <button @click="incrementReactive">Reactive +1</button>
        </div>
      </div>
    </div>
    
    <!-- 解构问题演示 -->
    <div class="section">
      <h3>2. 解构响应式对象</h3>
      
      <div class="destructure-demo card">
        <p>原始值: {{ reactiveState.count }}</p>
        <p>错误解构（不响应）: {{ wrongCount }}</p>
        <p>正确解构（toRefs）: {{ rightCount }}</p>
        <button @click="testDestructuring">测试解构</button>
        <p class="tip">💡 点击按钮查看控制台输出</p>
      </div>
    </div>
    
    <!-- 深层响应式 -->
    <div class="section">
      <h3>3. 深层响应式</h3>
      
      <div class="nested-demo card">
        <p>城市: {{ reactiveState.user.address.city }}</p>
        <p>区域: {{ reactiveState.user.address.district }}</p>
        <button @click="updateNested">更新嵌套数据</button>
        <p class="tip">💡 reactive 的嵌套属性也是响应式的</p>
      </div>
    </div>
    
    <!-- 数组操作 -->
    <div class="section">
      <h3>4. 响应式数组</h3>
      
      <div class="array-demo card">
        <p>爱好列表:</p>
        <ul>
          <li v-for="(hobby, index) in reactiveState.hobbies" :key="index">
            {{ hobby }}
            <button @click="removeHobby(index)" class="small-btn">删除</button>
          </li>
        </ul>
        <button @click="addHobby">添加爱好</button>
      </div>
    </div>
    
    <!-- 替换问题 -->
    <div class="section">
      <h3>5. 整体替换问题</h3>
      
      <div class="replace-demo card">
        <p>临时状态: {{ tempState.temp }}</p>
        <button @click="wrongReplace" class="danger">❌ 错误替换</button>
        <button @click="correctReplace" class="success">✅ 正确修改</button>
        <p class="tip">⚠️ 不要整体替换 reactive 对象</p>
      </div>
    </div>
    
    <!-- 响应式检查 -->
    <div class="section">
      <h3>6. 响应式检查</h3>
      
      <div class="check-demo card">
        <button @click="checkStatus">检查响应式状态</button>
        <p class="tip">💡 打开控制台查看检查结果</p>
      </div>
    </div>
    
    <!-- 总结 -->
    <div class="section summary">
      <h3>📝 使用建议</h3>
      <ul>
        <li>✅ 单个值用 <code>ref</code></li>
        <li>✅ 多个相关值用 <code>reactive</code></li>
        <li>✅ 需要解构时用 <code>toRefs</code></li>
        <li>❌ 不要整体替换 reactive 对象</li>
        <li>❌ 不要直接解构 reactive 对象</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.reactive-basics {
  padding: 20px;
  max-width: 1000px;
  margin: 0 auto;
}

.section {
  margin-bottom: 30px;
}

.section h3 {
  color: #42b883;
  margin-bottom: 15px;
}

.card {
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 15px;
}

.comparison {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

.ref-demo h4,
.reactive-demo h4 {
  color: #333;
  margin-bottom: 10px;
}

button {
  padding: 8px 16px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 10px;
  transition: background 0.3s;
}

button:hover {
  background: #35a372;
}

button.danger {
  background: #ff6b6b;
}

button.danger:hover {
  background: #ff5252;
}

button.success {
  background: #51cf66;
}

button.success:hover {
  background: #40c057;
}

.small-btn {
  padding: 4px 8px;
  font-size: 12px;
  margin-left: 10px;
}

.tip {
  color: #666;
  font-size: 14px;
  margin-top: 10px;
  font-style: italic;
}

ul {
  list-style: none;
  padding: 0;
}

li {
  padding: 8px;
  background: #f5f5f5;
  margin-bottom: 5px;
  border-radius: 4px;
  display: flex;
  justify-content: space-between;
  align-items: center;
}

code {
  background: #f0f0f0;
  padding: 2px 6px;
  border-radius: 3px;
  font-family: 'Courier New', monospace;
}

.summary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 20px;
  border-radius: 8px;
}

.summary h3 {
  color: white;
}

.summary ul {
  margin: 0;
}

.summary li {
  background: rgba(255, 255, 255, 0.1);
  color: white;
  border: 1px solid rgba(255, 255, 255, 0.2);
}

.summary code {
  background: rgba(255, 255, 255, 0.2);
  color: white;
}

@media (max-width: 768px) {
  .comparison {
    grid-template-columns: 1fr;
  }
}
</style>