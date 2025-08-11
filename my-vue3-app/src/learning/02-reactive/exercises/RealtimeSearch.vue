<script setup>
// Day 2 练习2：实时搜索
import { ref, watchEffect, computed, onBeforeUnmount } from 'vue'

// ==================== 数据状态 ====================

// 搜索相关
const searchQuery = ref('')
const searchResults = ref([])
const isSearching = ref(false)
const searchError = ref('')

// 搜索历史
const searchHistory = ref([])
const showHistory = ref(false)
const maxHistoryItems = 10

// 搜索配置
const searchConfig = ref({
  debounceTime: 300,  // 防抖延迟（毫秒）
  minQueryLength: 2,   // 最小搜索长度
  maxResults: 20,      // 最大结果数
  highlightMatches: true  // 是否高亮匹配
})

// 模拟数据库
const database = [
  // 编程语言
  { id: 1, title: 'JavaScript 基础教程', category: '编程', tags: ['前端', 'web'], content: 'JavaScript 是一种动态类型的编程语言...' },
  { id: 2, title: 'Python 入门指南', category: '编程', tags: ['后端', 'AI'], content: 'Python 是一种简单易学的编程语言...' },
  { id: 3, title: 'Java 高级编程', category: '编程', tags: ['后端', '企业级'], content: 'Java 是一种面向对象的编程语言...' },
  { id: 4, title: 'TypeScript 实战', category: '编程', tags: ['前端', '类型'], content: 'TypeScript 是 JavaScript 的超集...' },
  { id: 5, title: 'Go 语言精通', category: '编程', tags: ['后端', '并发'], content: 'Go 是一种高效的编程语言...' },
  
  // 框架
  { id: 6, title: 'Vue 3 完全指南', category: '框架', tags: ['前端', 'MVVM'], content: 'Vue 3 是一个渐进式 JavaScript 框架...' },
  { id: 7, title: 'React 深入浅出', category: '框架', tags: ['前端', '组件'], content: 'React 是一个用于构建用户界面的库...' },
  { id: 8, title: 'Angular 企业应用', category: '框架', tags: ['前端', '全栈'], content: 'Angular 是一个完整的前端框架...' },
  { id: 9, title: 'Spring Boot 实战', category: '框架', tags: ['后端', 'Java'], content: 'Spring Boot 简化了 Spring 应用开发...' },
  { id: 10, title: 'Django 快速开发', category: '框架', tags: ['后端', 'Python'], content: 'Django 是一个高级 Python Web 框架...' },
  
  // 数据库
  { id: 11, title: 'MySQL 性能优化', category: '数据库', tags: ['SQL', '关系型'], content: 'MySQL 是最流行的开源关系数据库...' },
  { id: 12, title: 'MongoDB 实践', category: '数据库', tags: ['NoSQL', '文档'], content: 'MongoDB 是一个文档型数据库...' },
  { id: 13, title: 'Redis 高级应用', category: '数据库', tags: ['NoSQL', '缓存'], content: 'Redis 是一个内存数据结构存储...' },
  { id: 14, title: 'PostgreSQL 进阶', category: '数据库', tags: ['SQL', '开源'], content: 'PostgreSQL 是一个强大的开源数据库...' },
  
  // 工具
  { id: 15, title: 'Git 版本控制', category: '工具', tags: ['版本控制', '协作'], content: 'Git 是分布式版本控制系统...' },
  { id: 16, title: 'Docker 容器化', category: '工具', tags: ['容器', '部署'], content: 'Docker 是容器化平台...' },
  { id: 17, title: 'Webpack 打包优化', category: '工具', tags: ['构建', '前端'], content: 'Webpack 是模块打包工具...' },
  { id: 18, title: 'VS Code 高效开发', category: '工具', tags: ['编辑器', 'IDE'], content: 'VS Code 是流行的代码编辑器...' },
  
  // 其他
  { id: 19, title: '算法与数据结构', category: '计算机基础', tags: ['算法', '面试'], content: '掌握常用算法和数据结构...' },
  { id: 20, title: '设计模式精讲', category: '架构', tags: ['设计', '模式'], content: '23种经典设计模式详解...' }
]

// ==================== 搜索功能 ====================

// 模拟异步搜索 API
async function searchAPI(query) {
  // 模拟网络延迟
  await new Promise(resolve => setTimeout(resolve, 200))
  
  // 模拟随机错误（5%概率）
  if (Math.random() < 0.05) {
    throw new Error('搜索服务暂时不可用')
  }
  
  // 执行搜索
  const lowerQuery = query.toLowerCase()
  const results = database.filter(item => {
    const searchableText = `${item.title} ${item.category} ${item.tags.join(' ')} ${item.content}`.toLowerCase()
    return searchableText.includes(lowerQuery)
  })
  
  // 按相关度排序（标题匹配优先）
  results.sort((a, b) => {
    const aInTitle = a.title.toLowerCase().includes(lowerQuery)
    const bInTitle = b.title.toLowerCase().includes(lowerQuery)
    if (aInTitle && !bInTitle) return -1
    if (!aInTitle && bInTitle) return 1
    return 0
  })
  
  return results.slice(0, searchConfig.value.maxResults)
}

// 防抖定时器
let debounceTimer = null

// 使用 watchEffect 实现自动搜索（带防抖）
watchEffect(() => {
  // 清除之前的定时器
  if (debounceTimer) {
    clearTimeout(debounceTimer)
  }
  
  const query = searchQuery.value.trim()
  
  // 检查最小长度
  if (query.length < searchConfig.value.minQueryLength) {
    searchResults.value = []
    isSearching.value = false
    searchError.value = ''
    return
  }
  
  // 显示加载状态
  isSearching.value = true
  searchError.value = ''
  
  // 防抖处理
  debounceTimer = setTimeout(async () => {
    try {
      const results = await searchAPI(query)
      searchResults.value = results
      
      // 添加到搜索历史（去重）
      if (!searchHistory.value.includes(query)) {
        searchHistory.value.unshift(query)
        // 限制历史记录数量
        if (searchHistory.value.length > maxHistoryItems) {
          searchHistory.value.pop()
        }
        // 保存到本地存储
        saveHistory()
      }
    } catch (error) {
      searchError.value = error.message
      searchResults.value = []
    } finally {
      isSearching.value = false
    }
  }, searchConfig.value.debounceTime)
})

// ==================== 搜索历史管理 ====================

// 加载搜索历史
function loadHistory() {
  const saved = localStorage.getItem('search-history')
  if (saved) {
    searchHistory.value = JSON.parse(saved)
  }
}

// 保存搜索历史
function saveHistory() {
  localStorage.setItem('search-history', JSON.stringify(searchHistory.value))
}

// 清除搜索历史
function clearHistory() {
  searchHistory.value = []
  localStorage.removeItem('search-history')
}

// 使用历史记录
function useHistoryItem(item) {
  searchQuery.value = item
  showHistory.value = false
}

// 删除单个历史记录
function removeHistoryItem(index) {
  searchHistory.value.splice(index, 1)
  saveHistory()
}

// 初始化加载历史
loadHistory()

// ==================== 高亮功能 ====================

// 高亮匹配文本
function highlightText(text, query) {
  if (!query || !searchConfig.value.highlightMatches) return text
  
  const regex = new RegExp(`(${query})`, 'gi')
  return text.replace(regex, '<mark>$1</mark>')
}

// ==================== 搜索统计 ====================

// 搜索统计
const searchStats = computed(() => {
  const categoryCount = {}
  searchResults.value.forEach(item => {
    categoryCount[item.category] = (categoryCount[item.category] || 0) + 1
  })
  
  return {
    total: searchResults.value.length,
    byCategory: categoryCount,
    hasMore: searchResults.value.length >= searchConfig.value.maxResults
  }
})

// 热门搜索（模拟）
const hotSearches = [
  'Vue 3', 'React', 'JavaScript', 'TypeScript', 
  'Python', 'Docker', 'Git', 'MySQL'
]

// ==================== 清理 ====================

onBeforeUnmount(() => {
  if (debounceTimer) {
    clearTimeout(debounceTimer)
  }
})

// ==================== 快捷操作 ====================

// 清空搜索
function clearSearch() {
  searchQuery.value = ''
  searchResults.value = []
}

// 快速搜索
function quickSearch(keyword) {
  searchQuery.value = keyword
}
</script>

<template>
  <div class="realtime-search">
    <h2>🔍 练习2：实时搜索</h2>
    
    <!-- 搜索配置 -->
    <div class="search-config">
      <h4>⚙️ 搜索配置</h4>
      <div class="config-grid">
        <label>
          防抖延迟:
          <input 
            v-model.number="searchConfig.debounceTime" 
            type="number" 
            min="0" 
            max="1000"
            step="100"
          > ms
        </label>
        <label>
          最小长度:
          <input 
            v-model.number="searchConfig.minQueryLength" 
            type="number" 
            min="1" 
            max="5"
          > 字符
        </label>
        <label>
          最大结果:
          <input 
            v-model.number="searchConfig.maxResults" 
            type="number" 
            min="5" 
            max="50"
          > 条
        </label>
        <label>
          <input 
            v-model="searchConfig.highlightMatches" 
            type="checkbox"
          >
          高亮匹配
        </label>
      </div>
    </div>
    
    <!-- 搜索框 -->
    <div class="search-box">
      <div class="search-input-wrapper">
        <input 
          v-model="searchQuery"
          @focus="showHistory = true"
          @blur="setTimeout(() => showHistory = false, 200)"
          placeholder="输入关键词搜索（最少2个字符）..."
          class="search-input"
        >
        <button v-if="searchQuery" @click="clearSearch" class="clear-btn">
          ✕
        </button>
      </div>
      
      <!-- 搜索状态 -->
      <div class="search-status">
        <span v-if="isSearching" class="loading">🔄 搜索中...</span>
        <span v-else-if="searchQuery.length > 0 && searchQuery.length < searchConfig.minQueryLength" class="hint">
          请输入至少 {{ searchConfig.minQueryLength }} 个字符
        </span>
        <span v-else-if="searchError" class="error">❌ {{ searchError }}</span>
        <span v-else-if="searchQuery && searchResults.length === 0" class="no-results">
          没有找到相关结果
        </span>
      </div>
      
      <!-- 搜索历史 -->
      <div v-if="showHistory && searchHistory.length > 0 && !searchQuery" class="search-history">
        <div class="history-header">
          <span>搜索历史</span>
          <button @click="clearHistory" class="clear-history">清除</button>
        </div>
        <div class="history-list">
          <div 
            v-for="(item, index) in searchHistory" 
            :key="index"
            class="history-item"
            @click="useHistoryItem(item)"
          >
            <span>🕒 {{ item }}</span>
            <button @click.stop="removeHistoryItem(index)" class="remove-history">×</button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 热门搜索 -->
    <div v-if="!searchQuery" class="hot-searches">
      <h4>🔥 热门搜索</h4>
      <div class="hot-tags">
        <span 
          v-for="keyword in hotSearches" 
          :key="keyword"
          @click="quickSearch(keyword)"
          class="hot-tag"
        >
          {{ keyword }}
        </span>
      </div>
    </div>
    
    <!-- 搜索结果 -->
    <div v-if="searchResults.length > 0" class="search-results">
      <!-- 结果统计 -->
      <div class="result-stats">
        <span>找到 {{ searchStats.total }} 个结果</span>
        <span v-if="searchStats.hasMore" class="has-more">（仅显示前 {{ searchConfig.maxResults }} 条）</span>
        <div class="category-stats">
          <span 
            v-for="(count, category) in searchStats.byCategory" 
            :key="category"
            class="category-tag"
          >
            {{ category }} ({{ count }})
          </span>
        </div>
      </div>
      
      <!-- 结果列表 -->
      <div class="result-list">
        <div 
          v-for="result in searchResults" 
          :key="result.id"
          class="result-item"
        >
          <div class="result-header">
            <h3 v-html="highlightText(result.title, searchQuery)"></h3>
            <span class="result-category">{{ result.category }}</span>
          </div>
          
          <p 
            class="result-content" 
            v-html="highlightText(result.content, searchQuery)"
          ></p>
          
          <div class="result-tags">
            <span 
              v-for="tag in result.tags" 
              :key="tag"
              class="tag"
              v-html="highlightText(tag, searchQuery)"
            ></span>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 搜索提示 -->
    <div v-if="!searchQuery && searchHistory.length === 0" class="search-tips">
      <h4>💡 搜索提示</h4>
      <ul>
        <li>输入至少 {{ searchConfig.minQueryLength }} 个字符开始搜索</li>
        <li>搜索会在你停止输入 {{ searchConfig.debounceTime }}ms 后自动执行</li>
        <li>支持搜索标题、分类、标签和内容</li>
        <li>搜索历史会自动保存最近 {{ maxHistoryItems }} 条记录</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.realtime-search {
  max-width: 900px;
  margin: 0 auto;
  padding: 20px;
}

h2 {
  text-align: center;
  color: #42b883;
  margin-bottom: 30px;
}

/* 搜索配置 */
.search-config {
  background: white;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.search-config h4 {
  margin-top: 0;
  color: #333;
  margin-bottom: 15px;
}

.config-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 15px;
}

.config-grid label {
  display: flex;
  align-items: center;
  gap: 5px;
  font-size: 14px;
}

.config-grid input[type="number"] {
  width: 60px;
  padding: 4px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

/* 搜索框 */
.search-box {
  position: relative;
  margin-bottom: 20px;
}

.search-input-wrapper {
  position: relative;
}

.search-input {
  width: 100%;
  padding: 15px 40px 15px 15px;
  font-size: 16px;
  border: 2px solid #42b883;
  border-radius: 8px;
  outline: none;
  transition: border-color 0.3s;
}

.search-input:focus {
  border-color: #35a372;
  box-shadow: 0 0 0 3px rgba(66, 184, 131, 0.1);
}

.clear-btn {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  background: #f0f0f0;
  border: none;
  border-radius: 50%;
  width: 30px;
  height: 30px;
  cursor: pointer;
  transition: background 0.3s;
}

.clear-btn:hover {
  background: #e0e0e0;
}

/* 搜索状态 */
.search-status {
  margin-top: 10px;
  font-size: 14px;
}

.loading {
  color: #ff9800;
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

.hint {
  color: #666;
}

.error {
  color: #f44336;
}

.no-results {
  color: #999;
}

/* 搜索历史 */
.search-history {
  position: absolute;
  top: 100%;
  left: 0;
  right: 0;
  background: white;
  border: 1px solid #ddd;
  border-radius: 8px;
  margin-top: 5px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
  z-index: 100;
  max-height: 300px;
  overflow-y: auto;
}

.history-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 15px;
  border-bottom: 1px solid #f0f0f0;
  font-weight: bold;
}

.clear-history {
  padding: 4px 10px;
  background: #ff6b6b;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
}

.history-list {
  padding: 5px;
}

.history-item {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px 10px;
  cursor: pointer;
  transition: background 0.2s;
  border-radius: 4px;
}

.history-item:hover {
  background: #f5f5f5;
}

.remove-history {
  background: none;
  border: none;
  color: #999;
  font-size: 20px;
  cursor: pointer;
  padding: 0 5px;
}

.remove-history:hover {
  color: #ff6b6b;
}

/* 热门搜索 */
.hot-searches {
  background: white;
  padding: 15px;
  border-radius: 8px;
  margin-bottom: 20px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.hot-searches h4 {
  margin-top: 0;
  color: #333;
  margin-bottom: 15px;
}

.hot-tags {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.hot-tag {
  padding: 6px 12px;
  background: #f0f0f0;
  border-radius: 20px;
  cursor: pointer;
  transition: all 0.3s;
  font-size: 14px;
}

.hot-tag:hover {
  background: #42b883;
  color: white;
  transform: translateY(-2px);
}

/* 搜索结果 */
.search-results {
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.result-stats {
  margin-bottom: 20px;
  padding-bottom: 15px;
  border-bottom: 2px solid #f0f0f0;
  font-size: 14px;
  color: #666;
}

.has-more {
  color: #ff9800;
  margin-left: 10px;
}

.category-stats {
  display: inline-flex;
  gap: 10px;
  margin-left: 20px;
}

.category-tag {
  padding: 2px 8px;
  background: #e3f2fd;
  border-radius: 4px;
  font-size: 12px;
  color: #1976d2;
}

.result-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.result-item {
  padding: 15px;
  background: #f9f9f9;
  border-radius: 8px;
  transition: transform 0.2s;
}

.result-item:hover {
  transform: translateX(5px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.result-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.result-header h3 {
  margin: 0;
  color: #333;
  font-size: 18px;
}

.result-category {
  padding: 4px 10px;
  background: #42b883;
  color: white;
  border-radius: 4px;
  font-size: 12px;
}

.result-content {
  color: #666;
  margin: 10px 0;
  line-height: 1.6;
}

.result-tags {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.tag {
  padding: 4px 8px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 12px;
  color: #666;
}

/* 高亮样式 */
:deep(mark) {
  background: yellow;
  padding: 0 2px;
  border-radius: 2px;
}

/* 搜索提示 */
.search-tips {
  background: #f5f5f5;
  padding: 20px;
  border-radius: 8px;
  margin-top: 20px;
}

.search-tips h4 {
  margin-top: 0;
  color: #333;
}

.search-tips ul {
  margin: 10px 0;
  padding-left: 20px;
  color: #666;
}

.search-tips li {
  margin: 8px 0;
}

@media (max-width: 600px) {
  .config-grid {
    grid-template-columns: 1fr;
  }
  
  .category-stats {
    display: block;
    margin-left: 0;
    margin-top: 10px;
  }
  
  .result-header {
    flex-direction: column;
    align-items: flex-start;
    gap: 5px;
  }
}
</style>