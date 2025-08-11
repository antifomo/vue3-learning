<script setup>
// Day 2 - watch 和 watchEffect 示例
import { ref, reactive, watch, watchEffect, onBeforeUnmount } from 'vue'

// ==================== watch 基础示例 ====================

// 监听 ref
const count = ref(0)
const watchLog = ref([])

watch(count, (newVal, oldVal) => {
  const log = `count: ${oldVal} → ${newVal}`
  console.log(log)
  watchLog.value.push({
    time: new Date().toLocaleTimeString(),
    message: log
  })
})

// 监听 reactive 对象的属性
const user = reactive({
  name: '张三',
  age: 25,
  address: {
    city: '北京',
    district: '朝阳区'
  }
})

// 监听整个对象（需要 deep）
watch(user, (newVal, oldVal) => {
  console.log('user 对象变化:', newVal)
}, { deep: true })

// 监听特定属性（使用 getter 函数）
watch(
  () => user.age,
  (newAge, oldAge) => {
    console.log(`年龄从 ${oldAge} 变为 ${newAge}`)
    watchLog.value.push({
      time: new Date().toLocaleTimeString(),
      message: `年龄: ${oldAge} → ${newAge}`
    })
  }
)

// 监听嵌套属性
watch(
  () => user.address.city,
  (newCity, oldCity) => {
    console.log(`城市从 ${oldCity} 变为 ${newCity}`)
    watchLog.value.push({
      time: new Date().toLocaleTimeString(),
      message: `城市: ${oldCity} → ${newCity}`
    })
  }
)

// ==================== 监听多个源 ====================

const firstName = ref('张')
const lastName = ref('三')

watch(
  [firstName, lastName],
  ([newFirst, newLast], [oldFirst, oldLast]) => {
    console.log(`姓名从 ${oldFirst}${oldLast} 变为 ${newFirst}${newLast}`)
    watchLog.value.push({
      time: new Date().toLocaleTimeString(),
      message: `姓名: ${oldFirst}${oldLast} → ${newFirst}${newLast}`
    })
  }
)

// ==================== watch 配置选项 ====================

const searchQuery = ref('')
const searchResults = ref([])
const isSearching = ref(false)

// 模拟搜索 API
async function searchAPI(query) {
  await new Promise(resolve => setTimeout(resolve, 500))
  return [
    `结果1: ${query}`,
    `结果2: ${query}`,
    `结果3: ${query}`
  ]
}

// immediate: 立即执行
// deep: 深度监听
watch(searchQuery, async (newQuery) => {
  if (newQuery.length < 2) {
    searchResults.value = []
    return
  }
  
  isSearching.value = true
  console.log('搜索:', newQuery)
  
  try {
    const results = await searchAPI(newQuery)
    searchResults.value = results
  } finally {
    isSearching.value = false
  }
}, {
  immediate: false,  // 是否立即执行
  // flush: 'post'    // 调整回调时机
})

// ==================== watchEffect 示例 ====================

const width = ref(100)
const height = ref(100)
const area = ref(0)
const effectLog = ref([])

// watchEffect: 自动收集依赖
const stopEffect = watchEffect(() => {
  // 这个函数会自动追踪 width 和 height
  area.value = width.value * height.value
  
  const log = `面积计算: ${width.value} × ${height.value} = ${area.value}`
  console.log(log)
  effectLog.value.push({
    time: new Date().toLocaleTimeString(),
    message: log
  })
})

// ==================== watchEffect 清理副作用 ====================

const intervalEnabled = ref(false)
const intervalCount = ref(0)

watchEffect((onCleanup) => {
  if (intervalEnabled.value) {
    console.log('启动定时器')
    const timer = setInterval(() => {
      intervalCount.value++
    }, 1000)
    
    // 清理函数：当依赖变化或组件卸载时调用
    onCleanup(() => {
      console.log('清理定时器')
      clearInterval(timer)
    })
  }
})

// ==================== 表单自动保存示例 ====================

const formData = reactive({
  title: '',
  content: '',
  tags: []
})
const saveStatus = ref('未保存')
const saveCount = ref(0)

// 监听表单变化，自动保存到本地存储
let saveTimer = null
watch(
  formData,
  (newData) => {
    saveStatus.value = '保存中...'
    
    // 防抖：清除之前的定时器
    if (saveTimer) clearTimeout(saveTimer)
    
    // 延迟保存
    saveTimer = setTimeout(() => {
      localStorage.setItem('formDraft', JSON.stringify(newData))
      saveCount.value++
      saveStatus.value = `已自动保存 (${saveCount.value}次)`
      console.log('表单已保存:', newData)
    }, 1000)
  },
  { 
    deep: true  // 深度监听对象
  }
)

// 加载草稿
function loadDraft() {
  const draft = localStorage.getItem('formDraft')
  if (draft) {
    const data = JSON.parse(draft)
    formData.title = data.title
    formData.content = data.content
    formData.tags = data.tags || []
    saveStatus.value = '已加载草稿'
  }
}

// ==================== 停止侦听 ====================

const manualWatch = ref(0)
let stopManualWatch = null

function startWatch() {
  if (stopManualWatch) {
    console.log('监听已在运行')
    return
  }
  
  stopManualWatch = watch(manualWatch, (newVal) => {
    console.log('手动监听值变化:', newVal)
    watchLog.value.push({
      time: new Date().toLocaleTimeString(),
      message: `手动监听: ${newVal}`
    })
  })
  
  console.log('开始监听')
}

function stopWatch() {
  if (stopManualWatch) {
    stopManualWatch()
    stopManualWatch = null
    console.log('停止监听')
  }
}

// ==================== watch vs watchEffect 对比 ====================

const compareA = ref(1)
const compareB = ref(2)
const compareLog = ref([])

// watch: 明确指定依赖，可以获取新旧值
watch([compareA, compareB], ([newA, newB], [oldA, oldB]) => {
  compareLog.value.push({
    type: 'watch',
    message: `A: ${oldA}→${newA}, B: ${oldB}→${newB}`
  })
})

// watchEffect: 自动收集依赖，不能获取旧值
watchEffect(() => {
  compareLog.value.push({
    type: 'watchEffect',
    message: `当前值 A: ${compareA.value}, B: ${compareB.value}`
  })
})

// 清理函数
onBeforeUnmount(() => {
  if (stopManualWatch) stopManualWatch()
  if (stopEffect) stopEffect()
})
</script>

<template>
  <div class="watch-example">
    <h2>👁️ Watch 和 WatchEffect 示例</h2>
    
    <!-- watch 基础示例 -->
    <div class="section">
      <h3>1. Watch 基础用法</h3>
      <div class="card">
        <div class="controls">
          <button @click="count++">Count: {{ count }}</button>
          <button @click="user.age++">Age: {{ user.age }}</button>
          <button @click="user.address.city = user.address.city === '北京' ? '上海' : '北京'">
            切换城市: {{ user.address.city }}
          </button>
        </div>
        
        <div class="log-panel">
          <h4>监听日志:</h4>
          <div class="log-list">
            <div v-for="(log, index) in watchLog" :key="index" class="log-item">
              <span class="time">{{ log.time }}</span>
              <span class="message">{{ log.message }}</span>
            </div>
            <p v-if="watchLog.length === 0" class="empty">暂无日志</p>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 监听多个源 -->
    <div class="section">
      <h3>2. 监听多个源</h3>
      <div class="card">
        <div class="input-group">
          <input v-model="firstName" placeholder="姓">
          <input v-model="lastName" placeholder="名">
        </div>
        <p>完整姓名: {{ firstName }}{{ lastName }}</p>
        <p class="tip">💡 同时监听多个数据源的变化</p>
      </div>
    </div>
    
    <!-- 搜索示例 -->
    <div class="section">
      <h3>3. 异步搜索（Watch）</h3>
      <div class="card">
        <input 
          v-model="searchQuery" 
          placeholder="输入至少2个字符搜索..."
          class="search-input"
        >
        
        <div v-if="isSearching" class="loading">搜索中...</div>
        
        <div v-else class="search-results">
          <div v-for="result in searchResults" :key="result" class="result-item">
            {{ result }}
          </div>
          <p v-if="searchResults.length === 0 && searchQuery.length >= 2" class="empty">
            没有找到结果
          </p>
        </div>
      </div>
    </div>
    
    <!-- watchEffect 示例 -->
    <div class="section">
      <h3>4. WatchEffect 自动依赖收集</h3>
      <div class="card">
        <div class="dimension-controls">
          <div>
            <label>宽度: {{ width }}</label>
            <input type="range" v-model.number="width" min="50" max="200">
          </div>
          <div>
            <label>高度: {{ height }}</label>
            <input type="range" v-model.number="height" min="50" max="200">
          </div>
        </div>
        
        <div class="result">
          <p>面积: {{ area }} 平方单位</p>
          <div class="rect" :style="{ width: width + 'px', height: height + 'px' }"></div>
        </div>
        
        <button @click="stopEffect">停止 Effect</button>
      </div>
    </div>
    
    <!-- 清理副作用 -->
    <div class="section">
      <h3>5. 清理副作用</h3>
      <div class="card">
        <button @click="intervalEnabled = !intervalEnabled">
          {{ intervalEnabled ? '停止' : '启动' }}定时器
        </button>
        <p>计数: {{ intervalCount }}</p>
        <p class="tip">💡 切换时会自动清理之前的定时器</p>
      </div>
    </div>
    
    <!-- 表单自动保存 -->
    <div class="section">
      <h3>6. 表单自动保存</h3>
      <div class="card">
        <div class="form">
          <input v-model="formData.title" placeholder="标题">
          <textarea v-model="formData.content" placeholder="内容"></textarea>
          <div class="tags">
            <input 
              placeholder="添加标签（回车）" 
              @keyup.enter="e => { formData.tags.push(e.target.value); e.target.value = '' }"
            >
            <div class="tag-list">
              <span v-for="(tag, index) in formData.tags" :key="index" class="tag">
                {{ tag }}
                <button @click="formData.tags.splice(index, 1)" class="remove">×</button>
              </span>
            </div>
          </div>
        </div>
        
        <div class="save-status">
          状态: {{ saveStatus }}
          <button @click="loadDraft">加载草稿</button>
        </div>
      </div>
    </div>
    
    <!-- 停止侦听 -->
    <div class="section">
      <h3>7. 手动控制侦听</h3>
      <div class="card">
        <div class="controls">
          <button @click="manualWatch++">值: {{ manualWatch }}</button>
          <button @click="startWatch">开始监听</button>
          <button @click="stopWatch">停止监听</button>
        </div>
        <p class="tip">💡 可以手动控制监听的开始和停止</p>
      </div>
    </div>
    
    <!-- watch vs watchEffect 对比 -->
    <div class="section">
      <h3>8. Watch vs WatchEffect 对比</h3>
      <div class="card">
        <div class="controls">
          <button @click="compareA++">A: {{ compareA }}</button>
          <button @click="compareB++">B: {{ compareB }}</button>
        </div>
        
        <div class="compare-log">
          <h4>执行日志:</h4>
          <div v-for="(log, index) in compareLog.slice(-10)" :key="index" class="compare-item">
            <span class="type" :class="log.type">{{ log.type }}</span>
            <span>{{ log.message }}</span>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 总结 -->
    <div class="section summary">
      <h3>📝 总结对比</h3>
      <table class="comparison-table">
        <thead>
          <tr>
            <th>特性</th>
            <th>watch</th>
            <th>watchEffect</th>
          </tr>
        </thead>
        <tbody>
          <tr>
            <td>依赖追踪</td>
            <td>显式指定</td>
            <td>自动收集</td>
          </tr>
          <tr>
            <td>获取旧值</td>
            <td>✅ 可以</td>
            <td>❌ 不能</td>
          </tr>
          <tr>
            <td>立即执行</td>
            <td>需配置 immediate</td>
            <td>默认立即执行</td>
          </tr>
          <tr>
            <td>使用场景</td>
            <td>需要旧值或精确控制</td>
            <td>简单的副作用</td>
          </tr>
        </tbody>
      </table>
    </div>
  </div>
</template>

<style scoped>
.watch-example {
  padding: 20px;
  max-width: 1200px;
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
}

.controls {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

button {
  padding: 8px 16px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s;
}

button:hover {
  background: #35a372;
}

input, textarea, select {
  padding: 8px 12px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  width: 100%;
  box-sizing: border-box;
}

input:focus, textarea:focus {
  outline: none;
  border-color: #42b883;
}

.input-group {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
}

/* 日志面板 */
.log-panel {
  background: #f5f5f5;
  padding: 15px;
  border-radius: 4px;
  max-height: 200px;
  overflow-y: auto;
}

.log-panel h4 {
  margin-top: 0;
  color: #333;
}

.log-list {
  font-family: monospace;
  font-size: 13px;
}

.log-item {
  padding: 5px;
  border-bottom: 1px solid #e0e0e0;
}

.time {
  color: #666;
  margin-right: 10px;
}

.message {
  color: #333;
}

/* 搜索结果 */
.search-input {
  margin-bottom: 15px;
}

.loading {
  text-align: center;
  color: #666;
  padding: 20px;
}

.search-results {
  min-height: 100px;
}

.result-item {
  padding: 10px;
  background: #f5f5f5;
  margin-bottom: 5px;
  border-radius: 4px;
}

/* 尺寸控制 */
.dimension-controls {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin-bottom: 20px;
}

.dimension-controls label {
  display: block;
  margin-bottom: 5px;
}

.result {
  text-align: center;
}

.rect {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  margin: 20px auto;
  border-radius: 4px;
  transition: all 0.3s;
}

/* 表单 */
.form {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-bottom: 15px;
}

textarea {
  min-height: 100px;
  resize: vertical;
}

.tags {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.tag-list {
  display: flex;
  flex-wrap: wrap;
  gap: 5px;
}

.tag {
  background: #42b883;
  color: white;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 14px;
}

.tag .remove {
  background: none;
  padding: 0 4px;
  margin-left: 4px;
  cursor: pointer;
}

.save-status {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px;
  background: #e8f5e9;
  border-radius: 4px;
}

/* 对比日志 */
.compare-log {
  background: #f5f5f5;
  padding: 15px;
  border-radius: 4px;
  max-height: 250px;
  overflow-y: auto;
}

.compare-item {
  display: flex;
  gap: 10px;
  padding: 5px;
  border-bottom: 1px solid #e0e0e0;
  font-size: 13px;
}

.type {
  padding: 2px 6px;
  border-radius: 3px;
  font-weight: bold;
  min-width: 80px;
  text-align: center;
}

.type.watch {
  background: #e3f2fd;
  color: #1976d2;
}

.type.watchEffect {
  background: #f3e5f5;
  color: #7b1fa2;
}

/* 对比表格 */
.comparison-table {
  width: 100%;
  border-collapse: collapse;
}

.comparison-table th,
.comparison-table td {
  padding: 12px;
  text-align: left;
  border-bottom: 1px solid rgba(255, 255, 255, 0.2);
}

.comparison-table th {
  background: rgba(255, 255, 255, 0.1);
  font-weight: bold;
}

.empty {
  text-align: center;
  color: #999;
  padding: 20px;
}

.tip {
  color: #666;
  font-size: 14px;
  margin-top: 10px;
  font-style: italic;
}

/* 总结 */
.summary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 20px;
  border-radius: 8px;
}

.summary h3 {
  color: white;
}

.summary table {
  color: white;
}

@media (max-width: 768px) {
  .controls {
    flex-direction: column;
  }
  
  .dimension-controls {
    grid-template-columns: 1fr;
  }
}
</style>