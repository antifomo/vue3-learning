<script setup>
// Day 2 - 增强版待办事项列表（综合实践）
import { reactive, computed, watch, watchEffect, toRefs, ref } from 'vue'

// ==================== 状态管理 ====================

// 使用 reactive 管理整个应用状态
const state = reactive({
  todos: [],
  filter: 'all',  // all | active | completed
  searchQuery: '',
  newTodoText: '',
  editingId: null,
  editingText: '',
  sortBy: 'createdAt',  // createdAt | text | status
  sortOrder: 'asc'  // asc | desc
})

// 统计信息
const stats = reactive({
  totalAdded: 0,
  totalCompleted: 0,
  totalDeleted: 0
})

// ==================== 本地存储 ====================

// 从本地存储加载数据
const loadTodos = () => {
  try {
    const saved = localStorage.getItem('todos-advanced')
    if (saved) {
      const data = JSON.parse(saved)
      state.todos = data.todos || []
      stats.totalAdded = data.stats?.totalAdded || 0
      stats.totalCompleted = data.stats?.totalCompleted || 0
      stats.totalDeleted = data.stats?.totalDeleted || 0
    }
  } catch (error) {
    console.error('加载数据失败:', error)
  }
}

// 初始化加载
loadTodos()

// ==================== 核心功能 ====================

// 添加待办项
const addTodo = () => {
  if (!state.newTodoText.trim()) return
  
  const newTodo = {
    id: Date.now(),
    text: state.newTodoText.trim(),
    completed: false,
    createdAt: new Date().toISOString(),
    completedAt: null,
    priority: 'normal',  // low | normal | high
    tags: []
  }
  
  state.todos.unshift(newTodo)
  state.newTodoText = ''
  stats.totalAdded++
  
  console.log('添加待办项:', newTodo)
}

// 删除待办项
const removeTodo = (id) => {
  const index = state.todos.findIndex(todo => todo.id === id)
  if (index > -1) {
    const todo = state.todos[index]
    state.todos.splice(index, 1)
    stats.totalDeleted++
    console.log('删除待办项:', todo)
  }
}

// 切换完成状态
const toggleTodo = (id) => {
  const todo = state.todos.find(t => t.id === id)
  if (todo) {
    todo.completed = !todo.completed
    if (todo.completed) {
      todo.completedAt = new Date().toISOString()
      stats.totalCompleted++
    } else {
      todo.completedAt = null
      stats.totalCompleted = Math.max(0, stats.totalCompleted - 1)
    }
  }
}

// 开始编辑
const startEdit = (todo) => {
  state.editingId = todo.id
  state.editingText = todo.text
}

// 保存编辑
const saveEdit = () => {
  const todo = state.todos.find(t => t.id === state.editingId)
  if (todo && state.editingText.trim()) {
    todo.text = state.editingText.trim()
  }
  cancelEdit()
}

// 取消编辑
const cancelEdit = () => {
  state.editingId = null
  state.editingText = ''
}

// 设置优先级
const setPriority = (id, priority) => {
  const todo = state.todos.find(t => t.id === id)
  if (todo) {
    todo.priority = priority
  }
}

// 批量操作
const clearCompleted = () => {
  const toRemove = state.todos.filter(todo => todo.completed)
  stats.totalDeleted += toRemove.length
  state.todos = state.todos.filter(todo => !todo.completed)
}

const toggleAll = () => {
  const allCompleted = state.todos.every(todo => todo.completed)
  state.todos.forEach(todo => {
    if (!todo.completed && !allCompleted) {
      stats.totalCompleted++
      todo.completedAt = new Date().toISOString()
    } else if (todo.completed && allCompleted) {
      stats.totalCompleted = Math.max(0, stats.totalCompleted - 1)
      todo.completedAt = null
    }
    todo.completed = !allCompleted
  })
}

// ==================== 计算属性 ====================

// 过滤和排序后的待办项
const filteredTodos = computed(() => {
  let result = [...state.todos]
  
  // 根据完成状态筛选
  if (state.filter === 'active') {
    result = result.filter(todo => !todo.completed)
  } else if (state.filter === 'completed') {
    result = result.filter(todo => todo.completed)
  }
  
  // 根据搜索词筛选
  if (state.searchQuery) {
    const query = state.searchQuery.toLowerCase()
    result = result.filter(todo => 
      todo.text.toLowerCase().includes(query) ||
      todo.tags.some(tag => tag.toLowerCase().includes(query))
    )
  }
  
  // 排序
  result.sort((a, b) => {
    let compareValue = 0
    
    if (state.sortBy === 'text') {
      compareValue = a.text.localeCompare(b.text)
    } else if (state.sortBy === 'status') {
      compareValue = (a.completed ? 1 : 0) - (b.completed ? 1 : 0)
    } else if (state.sortBy === 'priority') {
      const priorityOrder = { high: 0, normal: 1, low: 2 }
      compareValue = priorityOrder[a.priority] - priorityOrder[b.priority]
    } else {  // createdAt
      compareValue = new Date(a.createdAt) - new Date(b.createdAt)
    }
    
    return state.sortOrder === 'asc' ? compareValue : -compareValue
  })
  
  return result
})

// 统计信息（计算属性）
const computedStats = computed(() => {
  const total = state.todos.length
  const completed = state.todos.filter(t => t.completed).length
  const active = total - completed
  const completionRate = total > 0 ? Math.round((completed / total) * 100) : 0
  
  // 按优先级统计
  const byPriority = {
    high: state.todos.filter(t => t.priority === 'high' && !t.completed).length,
    normal: state.todos.filter(t => t.priority === 'normal' && !t.completed).length,
    low: state.todos.filter(t => t.priority === 'low' && !t.completed).length
  }
  
  return {
    total,
    completed,
    active,
    completionRate,
    byPriority,
    // 历史统计
    totalAdded: stats.totalAdded,
    totalCompleted: stats.totalCompleted,
    totalDeleted: stats.totalDeleted
  }
})

// 是否显示清除按钮
const hasCompleted = computed(() => {
  return state.todos.some(todo => todo.completed)
})

// ==================== 侦听器 ====================

// 监听待办项变化，自动保存
watch(
  () => ({ todos: state.todos, stats }),
  (data) => {
    localStorage.setItem('todos-advanced', JSON.stringify(data))
    console.log('数据已保存到本地存储')
  },
  { deep: true }
)

// 使用 watchEffect 实时显示搜索结果数量
const searchResultCount = ref(0)
watchEffect(() => {
  searchResultCount.value = filteredTodos.value.length
  if (state.searchQuery) {
    console.log(`搜索 "${state.searchQuery}" 找到 ${searchResultCount.value} 个结果`)
  }
})

// ==================== 工具函数 ====================

// 导出数据
const exportData = () => {
  const dataStr = JSON.stringify({
    todos: state.todos,
    stats,
    exportedAt: new Date().toISOString()
  }, null, 2)
  
  const blob = new Blob([dataStr], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `todos-${new Date().toISOString().split('T')[0]}.json`
  a.click()
  URL.revokeObjectURL(url)
}

// 导入数据
const importData = (event) => {
  const file = event.target.files[0]
  if (!file) return
  
  const reader = new FileReader()
  reader.onload = (e) => {
    try {
      const data = JSON.parse(e.target.result)
      state.todos = data.todos || []
      if (data.stats) {
        Object.assign(stats, data.stats)
      }
      console.log('数据导入成功')
    } catch (error) {
      console.error('导入失败:', error)
      alert('导入失败，请检查文件格式')
    }
  }
  reader.readAsText(file)
}

// 获取优先级样式
const getPriorityClass = (priority) => {
  return `priority-${priority}`
}

// 格式化日期
const formatDate = (dateString) => {
  if (!dateString) return ''
  const date = new Date(dateString)
  return date.toLocaleString('zh-CN', {
    month: 'short',
    day: 'numeric',
    hour: '2-digit',
    minute: '2-digit'
  })
}

// 使用 toRefs 解构状态
const { todos, filter, searchQuery, newTodoText, sortBy, sortOrder } = toRefs(state)
</script>

<template>
  <div class="todo-app-advanced">
    <h2>📝 增强版待办事项列表</h2>
    
    <!-- 统计面板 -->
    <div class="stats-panel">
      <div class="stat-card">
        <div class="stat-value">{{ computedStats.total }}</div>
        <div class="stat-label">总计</div>
      </div>
      <div class="stat-card">
        <div class="stat-value">{{ computedStats.active }}</div>
        <div class="stat-label">进行中</div>
      </div>
      <div class="stat-card">
        <div class="stat-value">{{ computedStats.completed }}</div>
        <div class="stat-label">已完成</div>
      </div>
      <div class="stat-card">
        <div class="stat-value">{{ computedStats.completionRate }}%</div>
        <div class="stat-label">完成率</div>
      </div>
    </div>
    
    <!-- 优先级统计 -->
    <div class="priority-stats">
      <span class="priority-stat priority-high">
        高: {{ computedStats.byPriority.high }}
      </span>
      <span class="priority-stat priority-normal">
        中: {{ computedStats.byPriority.normal }}
      </span>
      <span class="priority-stat priority-low">
        低: {{ computedStats.byPriority.low }}
      </span>
    </div>
    
    <!-- 输入区域 -->
    <div class="input-section">
      <input 
        v-model="newTodoText"
        @keyup.enter="addTodo"
        placeholder="添加新的待办事项..."
        class="todo-input"
      >
      <button @click="addTodo" class="add-btn">添加</button>
    </div>
    
    <!-- 控制区域 -->
    <div class="controls">
      <!-- 搜索 -->
      <input 
        v-model="searchQuery"
        placeholder="搜索待办事项..."
        class="search-input"
      >
      
      <!-- 筛选 -->
      <div class="filter-buttons">
        <button 
          @click="filter = 'all'"
          :class="{ active: filter === 'all' }"
        >
          全部
        </button>
        <button 
          @click="filter = 'active'"
          :class="{ active: filter === 'active' }"
        >
          进行中
        </button>
        <button 
          @click="filter = 'completed'"
          :class="{ active: filter === 'completed' }"
        >
          已完成
        </button>
      </div>
      
      <!-- 排序 -->
      <div class="sort-controls">
        <select v-model="sortBy">
          <option value="createdAt">创建时间</option>
          <option value="text">名称</option>
          <option value="status">状态</option>
          <option value="priority">优先级</option>
        </select>
        <button @click="sortOrder = sortOrder === 'asc' ? 'desc' : 'asc'">
          {{ sortOrder === 'asc' ? '↑' : '↓' }}
        </button>
      </div>
    </div>
    
    <!-- 批量操作 -->
    <div class="batch-actions">
      <button @click="toggleAll">全选/取消</button>
      <button @click="clearCompleted" v-if="hasCompleted">清除已完成</button>
      <button @click="exportData">导出数据</button>
      <label class="import-btn">
        导入数据
        <input type="file" @change="importData" accept=".json" hidden>
      </label>
    </div>
    
    <!-- 搜索结果提示 -->
    <div v-if="searchQuery" class="search-info">
      搜索 "{{ searchQuery }}" 找到 {{ searchResultCount }} 个结果
    </div>
    
    <!-- 待办列表 -->
    <transition-group name="list" tag="ul" class="todo-list">
      <li 
        v-for="todo in filteredTodos" 
        :key="todo.id"
        :class="{ 
          completed: todo.completed,
          editing: state.editingId === todo.id
        }"
        class="todo-item"
      >
        <!-- 复选框 -->
        <input 
          type="checkbox"
          :checked="todo.completed"
          @change="toggleTodo(todo.id)"
          class="todo-checkbox"
        >
        
        <!-- 内容 -->
        <div class="todo-content">
          <div v-if="state.editingId === todo.id" class="edit-form">
            <input 
              v-model="state.editingText"
              @keyup.enter="saveEdit"
              @keyup.esc="cancelEdit"
              class="edit-input"
            >
            <button @click="saveEdit" class="small">保存</button>
            <button @click="cancelEdit" class="small">取消</button>
          </div>
          <div v-else class="todo-text" @dblclick="startEdit(todo)">
            <span :class="getPriorityClass(todo.priority)">
              {{ todo.text }}
            </span>
            <div class="todo-meta">
              <span class="date">创建: {{ formatDate(todo.createdAt) }}</span>
              <span v-if="todo.completedAt" class="date">
                完成: {{ formatDate(todo.completedAt) }}
              </span>
            </div>
          </div>
        </div>
        
        <!-- 优先级选择 -->
        <select 
          :value="todo.priority"
          @change="setPriority(todo.id, $event.target.value)"
          class="priority-select"
          :class="getPriorityClass(todo.priority)"
        >
          <option value="high">高</option>
          <option value="normal">中</option>
          <option value="low">低</option>
        </select>
        
        <!-- 删除按钮 -->
        <button @click="removeTodo(todo.id)" class="delete-btn">删除</button>
      </li>
    </transition-group>
    
    <!-- 空状态 -->
    <div v-if="filteredTodos.length === 0" class="empty-state">
      <p v-if="searchQuery">没有找到匹配的待办事项</p>
      <p v-else-if="filter === 'completed'">还没有完成的待办事项</p>
      <p v-else-if="filter === 'active'">没有进行中的待办事项</p>
      <p v-else>暂无待办事项，添加一个吧！</p>
    </div>
    
    <!-- 历史统计 -->
    <div class="history-stats">
      <h4>历史统计</h4>
      <p>总添加: {{ computedStats.totalAdded }} 项</p>
      <p>总完成: {{ computedStats.totalCompleted }} 项</p>
      <p>总删除: {{ computedStats.totalDeleted }} 项</p>
    </div>
  </div>
</template>

<style scoped>
.todo-app-advanced {
  max-width: 800px;
  margin: 0 auto;
  padding: 20px;
}

h2 {
  text-align: center;
  color: #42b883;
  margin-bottom: 30px;
}

/* 统计面板 */
.stats-panel {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  gap: 15px;
  margin-bottom: 20px;
}

.stat-card {
  background: white;
  padding: 15px;
  border-radius: 8px;
  text-align: center;
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
}

.stat-value {
  font-size: 28px;
  font-weight: bold;
  color: #42b883;
}

.stat-label {
  font-size: 14px;
  color: #666;
  margin-top: 5px;
}

/* 优先级统计 */
.priority-stats {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
  justify-content: center;
}

.priority-stat {
  padding: 5px 15px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: bold;
}

.priority-high {
  background: #fee;
  color: #f44336;
}

.priority-normal {
  background: #fff3e0;
  color: #ff9800;
}

.priority-low {
  background: #e8f5e9;
  color: #4caf50;
}

/* 输入区域 */
.input-section {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.todo-input {
  flex: 1;
  padding: 12px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 16px;
}

.todo-input:focus {
  outline: none;
  border-color: #42b883;
}

.add-btn {
  padding: 12px 24px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background 0.3s;
}

.add-btn:hover {
  background: #35a372;
}

/* 控制区域 */
.controls {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
  flex-wrap: wrap;
  align-items: center;
}

.search-input {
  flex: 1;
  min-width: 200px;
  padding: 8px 12px;
  border: 2px solid #ddd;
  border-radius: 4px;
}

.filter-buttons {
  display: flex;
  gap: 5px;
}

.filter-buttons button {
  padding: 8px 16px;
  border: 1px solid #ddd;
  background: white;
  cursor: pointer;
  border-radius: 4px;
  transition: all 0.3s;
}

.filter-buttons button.active {
  background: #42b883;
  color: white;
  border-color: #42b883;
}

.sort-controls {
  display: flex;
  gap: 5px;
}

.sort-controls select {
  padding: 8px;
  border: 2px solid #ddd;
  border-radius: 4px;
  background: white;
}

.sort-controls button {
  padding: 8px 12px;
  background: #f5f5f5;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
}

/* 批量操作 */
.batch-actions {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.batch-actions button,
.import-btn {
  padding: 8px 16px;
  background: #f5f5f5;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s;
}

.batch-actions button:hover,
.import-btn:hover {
  background: #e0e0e0;
}

.import-btn {
  display: inline-block;
}

/* 搜索信息 */
.search-info {
  padding: 10px;
  background: #e3f2fd;
  border-radius: 4px;
  margin-bottom: 15px;
  color: #1976d2;
}

/* 待办列表 */
.todo-list {
  list-style: none;
  padding: 0;
  margin: 0;
}

.todo-item {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 15px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-bottom: 10px;
  transition: all 0.3s;
}

.todo-item:hover {
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.todo-item.completed {
  opacity: 0.6;
}

.todo-item.completed .todo-text {
  text-decoration: line-through;
  color: #999;
}

.todo-checkbox {
  width: 20px;
  height: 20px;
  cursor: pointer;
}

.todo-content {
  flex: 1;
}

.todo-text {
  cursor: text;
  user-select: none;
}

.todo-meta {
  font-size: 12px;
  color: #999;
  margin-top: 5px;
}

.date {
  margin-right: 15px;
}

.edit-form {
  display: flex;
  gap: 10px;
}

.edit-input {
  flex: 1;
  padding: 8px;
  border: 2px solid #42b883;
  border-radius: 4px;
}

.priority-select {
  padding: 4px 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  cursor: pointer;
}

.delete-btn {
  padding: 6px 12px;
  background: #ff6b6b;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s;
}

.delete-btn:hover {
  background: #ff5252;
}

button.small {
  padding: 6px 12px;
  font-size: 14px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

/* 空状态 */
.empty-state {
  text-align: center;
  padding: 60px 20px;
  color: #999;
}

/* 历史统计 */
.history-stats {
  margin-top: 30px;
  padding: 15px;
  background: #f5f5f5;
  border-radius: 8px;
  text-align: center;
}

.history-stats h4 {
  margin-top: 0;
  color: #333;
}

.history-stats p {
  margin: 5px 0;
  color: #666;
}

/* 过渡动画 */
.list-enter-active,
.list-leave-active {
  transition: all 0.3s;
}

.list-enter-from {
  opacity: 0;
  transform: translateX(-30px);
}

.list-leave-to {
  opacity: 0;
  transform: translateX(30px);
}

@media (max-width: 600px) {
  .controls {
    flex-direction: column;
  }
  
  .search-input {
    width: 100%;
  }
  
  .stats-panel {
    grid-template-columns: repeat(2, 1fr);
  }
}
</style>