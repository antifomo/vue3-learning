<script setup>
// 导入Vue3的响应式函数
import { ref } from 'vue'

// 创建响应式数据 - ref用于基本类型
const message = ref('Hello Vue3!')  // 响应式字符串
const count = ref(0)                // 响应式数字

// 练习1：新增数据
const userInput = ref('')           // 用户输入（双向绑定）
const clickHistory = ref([])        // 操作历史记录

// 定义方法 - 点击按钮时执行
const increment = () => {
  count.value++  // 修改ref值需要用.value
  
  // 练习1：记录到历史
  clickHistory.value.push({
    id: Date.now(),
    action: '点击 +1',
    value: count.value,
    time: new Date().toLocaleTimeString('zh-CN')
  })
}

// 定义方法 - 修改消息
const changeMessage = () => {
  message.value = `你已经点击了 ${count.value} 次！`
  
  // 练习1：记录到历史
  clickHistory.value.push({
    id: Date.now(),
    action: '更新消息',
    value: message.value,
    time: new Date().toLocaleTimeString('zh-CN')
  })
}

// 练习1：新增方法 - 处理输入框回车
const handleInputEnter = () => {
  if (userInput.value.trim()) {
    clickHistory.value.push({
      id: Date.now(),
      action: '输入内容',
      value: userInput.value,
      time: new Date().toLocaleTimeString('zh-CN')
    })
    message.value = `你输入了：${userInput.value}`
    userInput.value = ''  // 清空输入框
  }
}

// 练习1：清空历史记录
const clearHistory = () => {
  clickHistory.value = []
}

// 练习1：重置时也记录历史
const resetCount = () => {
  count.value = 0
  clickHistory.value.push({
    id: Date.now(),
    action: '重置计数',
    value: 0,
    time: new Date().toLocaleTimeString('zh-CN')
  })
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
      <button @click="resetCount" class="btn btn-danger">
        重置
      </button>
    </div>
    
    <!-- 条件渲染：v-if -->
    <p v-if="count > 10" class="achievement">
      🎉 太棒了！你已经点击超过10次了！
    </p>
    
    <!-- 练习1：双向绑定输入框 -->
    <div class="input-section">
      <h3>📝 双向绑定演示</h3>
      <input 
        v-model="userInput"
        @keyup.enter="handleInputEnter"
        placeholder="输入内容并按回车添加到历史"
        class="user-input"
      >
      <p class="input-preview">你正在输入：<strong>{{ userInput || '(空)' }}</strong></p>
    </div>
    
    <!-- 练习1：操作历史列表 -->
    <div class="history-section">
      <div class="history-header">
        <h3>📋 操作历史记录</h3>
        <button 
          @click="clearHistory" 
          v-if="clickHistory.length > 0"
          class="btn btn-warning"
        >
          清空历史
        </button>
      </div>
      
      <!-- 有历史记录时显示列表 -->
      <ul class="history-list" v-if="clickHistory.length > 0">
        <li 
          v-for="item in clickHistory" 
          :key="item.id"
          :class="{
            'even-item': typeof item.value === 'number' && item.value % 2 === 0,
            'odd-item': typeof item.value === 'number' && item.value % 2 !== 0,
            'text-item': typeof item.value === 'string'
          }"
        >
          <span class="time">{{ item.time }}</span>
          <span class="action">{{ item.action }}</span>
          <span class="value">
            <span v-if="typeof item.value === 'number'">
              {{ item.value }}
              <span class="number-type">
                ({{ item.value % 2 === 0 ? '偶数' : '奇数' }})
              </span>
            </span>
            <span v-else>{{ item.value }}</span>
          </span>
        </li>
      </ul>
      
      <!-- 无历史记录时显示提示 -->
      <p v-else class="no-history">
        暂无历史记录，试试点击按钮或输入内容吧！
      </p>
    </div>
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

/* 练习1：新增样式 */

/* 输入框部分样式 */
.input-section {
  margin: 30px auto;
  padding: 25px;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  max-width: 500px;
}

.input-section h3 {
  color: #42b883;
  margin-bottom: 15px;
}

.user-input {
  width: 100%;
  padding: 12px;
  font-size: 16px;
  border: 2px solid #42b883;
  border-radius: 5px;
  outline: none;
  transition: border-color 0.3s;
  box-sizing: border-box;
}

.user-input:focus {
  border-color: #35a372;
  box-shadow: 0 0 0 3px rgba(66, 184, 131, 0.1);
}

.input-preview {
  margin-top: 10px;
  color: #666;
}

.input-preview strong {
  color: #42b883;
}

/* 历史记录部分样式 */
.history-section {
  margin: 30px auto;
  padding: 25px;
  background: rgba(255, 255, 255, 0.95);
  border-radius: 10px;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  max-width: 600px;
}

.history-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
}

.history-header h3 {
  color: #42b883;
  margin: 0;
}

.history-list {
  list-style: none;
  padding: 0;
  margin: 0;
  max-height: 400px;
  overflow-y: auto;
}

.history-list li {
  padding: 12px 15px;
  margin: 8px 0;
  background: white;
  border-radius: 8px;
  display: flex;
  justify-content: space-between;
  align-items: center;
  transition: all 0.3s;
  border-left: 4px solid transparent;
}

.history-list li:hover {
  transform: translateX(5px);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

/* 奇偶数不同颜色 */
.even-item {
  background: linear-gradient(to right, #e3f2fd, #bbdefb);
  border-left-color: #2196f3;
}

.even-item .value {
  color: #1565c0;
}

.odd-item {
  background: linear-gradient(to right, #fff3e0, #ffe0b2);
  border-left-color: #ff9800;
}

.odd-item .value {
  color: #e65100;
}

.text-item {
  background: linear-gradient(to right, #f3e5f5, #e1bee7);
  border-left-color: #9c27b0;
}

.text-item .value {
  color: #6a1b9a;
}

/* 历史记录内容样式 */
.time {
  font-size: 12px;
  color: #999;
  min-width: 80px;
}

.action {
  font-weight: 600;
  color: #333;
  min-width: 80px;
  text-align: center;
}

.value {
  font-size: 16px;
  font-weight: bold;
  flex: 1;
  text-align: right;
}

.number-type {
  font-size: 12px;
  font-weight: normal;
  opacity: 0.7;
  margin-left: 5px;
}

.no-history {
  text-align: center;
  color: #999;
  font-style: italic;
  padding: 30px;
}

.btn-warning {
  background-color: #ff9800;
  color: white;
}

.btn-warning:hover {
  background-color: #f57c00;
}

/* 滚动条美化 */
.history-list::-webkit-scrollbar {
  width: 6px;
}

.history-list::-webkit-scrollbar-track {
  background: #f1f1f1;
  border-radius: 3px;
}

.history-list::-webkit-scrollbar-thumb {
  background: #42b883;
  border-radius: 3px;
}

.history-list::-webkit-scrollbar-thumb:hover {
  background: #35a372;
}
</style>