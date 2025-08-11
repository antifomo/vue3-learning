<script setup>
// 练习2：UserCard 组件
// 功能：显示用户信息，控制年龄，判断成年状态

import { ref, computed } from 'vue'

// 响应式数据
const userName = ref('张三')  // 用户姓名
const userAge = ref(16)       // 用户年龄（初始设为16，便于测试未成年状态）
const isEditing = ref(false)  // 是否在编辑用户名
const tempName = ref('')       // 临时存储编辑中的名字

// 计算属性：判断是否成年
const isAdult = computed(() => userAge.value >= 18)

// 计算属性：获取年龄状态文本
const ageStatus = computed(() => {
  if (userAge.value < 18) {
    return `未成年 (还差 ${18 - userAge.value} 年)`
  } else if (userAge.value >= 60) {
    return '老年'
  } else {
    return '成年'
  }
})

// 计算属性：状态样式类
const statusClass = computed(() => {
  return isAdult.value ? 'adult' : 'minor'
})

// 计算属性：成年进度
const progressWidth = computed(() => {
  const progress = Math.min((userAge.value / 18) * 100, 100)
  return `${progress}%`
})

// 计算属性：进度文本
const progressText = computed(() => {
  if (userAge.value >= 18) {
    return '已成年'
  }
  return `${Math.round((userAge.value / 18) * 100)}%`
})

// 方法：增加年龄
const increaseAge = () => {
  userAge.value++
}

// 方法：减少年龄
const decreaseAge = () => {
  if (userAge.value > 0) {
    userAge.value--
  }
}

// 方法：开始编辑用户名
const startEdit = () => {
  tempName.value = userName.value
  isEditing.value = true
}

// 方法：保存编辑
const saveEdit = () => {
  if (tempName.value.trim()) {
    userName.value = tempName.value.trim()
  }
  isEditing.value = false
}

// 方法：取消编辑
const cancelEdit = () => {
  isEditing.value = false
  tempName.value = ''
}

// 方法：重置为默认值
const reset = () => {
  userName.value = '张三'
  userAge.value = 16
  isEditing.value = false
}
</script>

<template>
  <div class="user-card">
    <div class="card-header">
      <h2>👤 用户卡片</h2>
    </div>
    
    <div class="card-body">
      <!-- 用户名部分 -->
      <div class="info-row">
        <label>姓名：</label>
        <div v-if="!isEditing" class="name-display">
          <span class="name-text">{{ userName }}</span>
          <button @click="startEdit" class="edit-btn" title="编辑姓名">✏️</button>
        </div>
        <div v-else class="name-edit">
          <input 
            v-model="tempName" 
            @keyup.enter="saveEdit"
            @keyup.esc="cancelEdit"
            placeholder="输入新名字"
            class="name-input"
          >
          <button @click="saveEdit" class="icon-btn" title="保存">✅</button>
          <button @click="cancelEdit" class="icon-btn" title="取消">❌</button>
        </div>
      </div>
      
      <!-- 年龄部分 -->
      <div class="info-row">
        <label>年龄：</label>
        <span class="age-value">{{ userAge }} 岁</span>
      </div>
      
      <!-- 状态显示 -->
      <div class="status-display" :class="statusClass">
        <div class="status-left">
          <span class="status-label">状态：</span>
          <span class="status-text">{{ ageStatus }}</span>
        </div>
        <span v-if="isAdult" class="badge badge-success">✓ 成年</span>
        <span v-else class="badge badge-warning">⚠ 未成年</span>
      </div>
      
      <!-- 年龄控制按钮 -->
      <div class="age-controls">
        <button 
          @click="decreaseAge" 
          :disabled="userAge <= 0"
          class="age-btn"
        >
          年龄 -1
        </button>
        <span class="age-display">{{ userAge }}</span>
        <button 
          @click="increaseAge" 
          class="age-btn primary"
        >
          年龄 +1
        </button>
      </div>
      
      <!-- 额外功能：年龄进度条 -->
      <div class="age-progress">
        <label>成年进度：</label>
        <div class="progress-container">
          <div class="progress-bar">
            <div 
              class="progress-fill" 
              :style="{ width: progressWidth }"
            ></div>
          </div>
          <span class="progress-text">{{ progressText }}</span>
        </div>
      </div>
      
      <!-- 特殊年龄提示 -->
      <div v-if="userAge === 17" class="special-tip tip-warning">
        🎂 再过一年就成年了，加油！
      </div>
      <div v-else-if="userAge === 18" class="special-tip tip-success">
        🎉 恭喜你刚好成年了！这是人生的重要时刻！
      </div>
      <div v-else-if="userAge === 0" class="special-tip tip-info">
        👶 刚出生的小宝宝！
      </div>
      <div v-else-if="userAge >= 100" class="special-tip tip-success">
        🎊 百岁老人，祝您健康长寿！
      </div>
      
      <!-- 重置按钮 -->
      <div class="card-actions">
        <button @click="reset" class="reset-btn">
          🔄 重置
        </button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.user-card {
  max-width: 450px;
  margin: 30px auto;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 20px;
  padding: 4px;
  box-shadow: 0 10px 40px rgba(0, 0, 0, 0.2);
}

.card-header {
  background: white;
  border-radius: 16px 16px 0 0;
  padding: 20px;
  text-align: center;
}

.card-header h2 {
  margin: 0;
  color: #333;
  font-size: 24px;
}

.card-body {
  background: white;
  padding: 25px;
  border-radius: 0 0 16px 16px;
}

/* 信息行样式 */
.info-row {
  display: flex;
  align-items: center;
  margin: 20px 0;
  font-size: 18px;
}

.info-row label {
  font-weight: bold;
  margin-right: 15px;
  min-width: 70px;
  color: #555;
}

.name-display {
  display: flex;
  align-items: center;
  gap: 10px;
}

.name-text {
  font-size: 20px;
  color: #333;
  font-weight: 500;
}

.edit-btn {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 18px;
  padding: 5px;
  transition: transform 0.2s;
}

.edit-btn:hover {
  transform: scale(1.2);
}

.name-edit {
  display: flex;
  align-items: center;
  gap: 8px;
}

.name-input {
  padding: 8px 12px;
  border: 2px solid #667eea;
  border-radius: 5px;
  font-size: 16px;
  outline: none;
  transition: border-color 0.3s;
}

.name-input:focus {
  border-color: #764ba2;
}

.icon-btn {
  background: none;
  border: none;
  cursor: pointer;
  font-size: 18px;
  padding: 5px;
  transition: transform 0.2s;
}

.icon-btn:hover {
  transform: scale(1.2);
}

.age-value {
  font-size: 20px;
  color: #667eea;
  font-weight: bold;
}

/* 状态显示样式 */
.status-display {
  padding: 15px 20px;
  margin: 25px 0;
  border-radius: 12px;
  display: flex;
  align-items: center;
  justify-content: space-between;
  transition: all 0.3s ease;
}

.status-left {
  display: flex;
  align-items: center;
  gap: 10px;
}

.status-label {
  font-weight: bold;
  color: #555;
}

.status-text {
  font-size: 16px;
  font-weight: 500;
}

.status-display.adult {
  background: linear-gradient(135deg, #d4fc79 0%, #96e6a1 100%);
}

.status-display.minor {
  background: linear-gradient(135deg, #ffecd2 0%, #fcb69f 100%);
}

.badge {
  padding: 6px 12px;
  border-radius: 20px;
  font-size: 14px;
  font-weight: bold;
  text-transform: uppercase;
}

.badge-success {
  background: #4caf50;
  color: white;
}

.badge-warning {
  background: #ff9800;
  color: white;
}

/* 年龄控制按钮 */
.age-controls {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 20px;
  margin: 25px 0;
}

.age-btn {
  padding: 10px 20px;
  font-size: 16px;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  cursor: pointer;
  background: white;
  transition: all 0.3s;
  font-weight: 500;
}

.age-btn:hover:not(:disabled) {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.age-btn.primary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border: none;
}

.age-btn:disabled {
  opacity: 0.4;
  cursor: not-allowed;
  transform: none;
}

.age-display {
  font-size: 24px;
  font-weight: bold;
  color: #667eea;
  min-width: 50px;
  text-align: center;
}

/* 年龄进度条 */
.age-progress {
  margin: 25px 0;
  padding: 20px;
  background: #f8f9fa;
  border-radius: 12px;
}

.age-progress label {
  display: block;
  margin-bottom: 10px;
  font-weight: bold;
  color: #555;
}

.progress-container {
  display: flex;
  align-items: center;
  gap: 15px;
}

.progress-bar {
  flex: 1;
  height: 24px;
  background: #e9ecef;
  border-radius: 12px;
  overflow: hidden;
  position: relative;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #667eea 0%, #764ba2 100%);
  transition: width 0.5s ease;
  border-radius: 12px;
  position: relative;
}

.progress-fill::after {
  content: '';
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: linear-gradient(
    90deg,
    transparent,
    rgba(255, 255, 255, 0.3),
    transparent
  );
  animation: shimmer 2s infinite;
}

@keyframes shimmer {
  0% { transform: translateX(-100%); }
  100% { transform: translateX(100%); }
}

.progress-text {
  font-weight: bold;
  color: #667eea;
  min-width: 50px;
}

/* 特殊提示 */
.special-tip {
  padding: 12px 16px;
  border-radius: 8px;
  margin: 20px 0;
  font-size: 15px;
  animation: fadeIn 0.5s;
}

@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.tip-warning {
  background: linear-gradient(135deg, #fff3cd 0%, #ffe8a1 100%);
  color: #856404;
  border-left: 4px solid #ffc107;
}

.tip-success {
  background: linear-gradient(135deg, #d4edda 0%, #c3e6cb 100%);
  color: #155724;
  border-left: 4px solid #28a745;
}

.tip-info {
  background: linear-gradient(135deg, #d1ecf1 0%, #bee5eb 100%);
  color: #0c5460;
  border-left: 4px solid #17a2b8;
}

/* 卡片操作区 */
.card-actions {
  margin-top: 25px;
  text-align: center;
}

.reset-btn {
  padding: 10px 30px;
  font-size: 16px;
  border: 2px solid #e0e0e0;
  border-radius: 8px;
  cursor: pointer;
  background: white;
  transition: all 0.3s;
  font-weight: 500;
}

.reset-btn:hover {
  background: #f5f5f5;
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.1);
}
</style>