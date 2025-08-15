<script setup>
// Day 3 - 事件处理详解
import { ref, reactive, nextTick } from 'vue'

// ==================== 基础状态 ====================

// 基础计数器
const counter = ref(0)
const eventLog = ref([])

// 鼠标事件数据
const mouseData = reactive({
  position: { x: 0, y: 0 },
  isMouseDown: false,
  dragData: { startX: 0, startY: 0, isDragging: false },
  hoverCount: 0,
  clickCount: 0
})

// 键盘事件数据
const keyboardData = reactive({
  lastKey: '',
  inputValue: '',
  keySequence: [],
  shortcuts: {
    'ctrl+s': '保存',
    'ctrl+c': '复制',
    'ctrl+v': '粘贴',
    'esc': '取消',
    'enter': '确认'
  },
  pressedKeys: new Set()
})

// 表单事件数据
const formData = reactive({
  username: '',
  email: '',
  message: '',
  selectedOption: '',
  agreedToTerms: false,
  rating: 0
})

// 自定义事件数据
const customEvents = reactive({
  notifications: [],
  messages: [],
  gameScore: 0,
  achievements: []
})

// 事件修饰符演示
const modifiersDemo = reactive({
  bubbleLog: [],
  preventDefaultLog: [],
  stopPropagationLog: [],
  onceTriggered: false,
  passiveScrollCount: 0
})

// 动态事件数据
const dynamicEvents = reactive({
  eventType: 'click',
  isListening: false,
  dynamicEventLog: []
})

// ==================== 基础事件处理 ====================

// 记录事件日志
function logEvent(eventType, details = '') {
  const timestamp = new Date().toLocaleTimeString()
  eventLog.value.unshift({
    id: Date.now(),
    type: eventType,
    details,
    timestamp
  })
  
  // 只保留最近20条日志
  if (eventLog.value.length > 20) {
    eventLog.value = eventLog.value.slice(0, 20)
  }
}

// 基础点击处理
function handleBasicClick() {
  counter.value++
  logEvent('click', `计数器增加到 ${counter.value}`)
}

// 内联处理器
function handleInlineClick(message) {
  logEvent('inline-click', message)
}

// 带参数的事件处理
function handleParameterClick(param1, param2, event) {
  logEvent('parameter-click', `参数1: ${param1}, 参数2: ${param2}, 事件类型: ${event.type}`)
}

// ==================== 鼠标事件 ====================

// 鼠标移动
function handleMouseMove(event) {
  // 更新鼠标位置
  mouseData.position.x = event.clientX
  mouseData.position.y = event.clientY
  
  // 处理拖拽逻辑
  if (mouseData.isMouseDown) {
    const deltaX = event.clientX - mouseData.dragData.startX
    const deltaY = event.clientY - mouseData.dragData.startY
    
    if (Math.abs(deltaX) > 5 || Math.abs(deltaY) > 5) {
      mouseData.dragData.isDragging = true
      logEvent('drag', `拖拽距离: (${deltaX}, ${deltaY})`)
    }
  }
}

// 鼠标按下
function handleMouseDown(event) {
  mouseData.isMouseDown = true
  mouseData.dragData.startX = event.clientX
  mouseData.dragData.startY = event.clientY
  mouseData.dragData.isDragging = false
  logEvent('mousedown', `位置: (${event.clientX}, ${event.clientY})`)
}

// 鼠标松开
function handleMouseUp(event) {
  mouseData.isMouseDown = false
  mouseData.dragData.isDragging = false
  logEvent('mouseup', '鼠标松开')
}


// 鼠标悬停
function handleMouseEnter() {
  mouseData.hoverCount++
  logEvent('mouseenter', `悬停次数: ${mouseData.hoverCount}`)
}

// 鼠标离开
function handleMouseLeave() {
  logEvent('mouseleave', '鼠标离开区域')
}

// 双击处理
function handleDoubleClick() {
  mouseData.clickCount += 2
  logEvent('dblclick', `双击, 总点击数: ${mouseData.clickCount}`)
}

// ==================== 键盘事件 ====================

// 键盘按下
function handleKeyDown(event) {
  keyboardData.lastKey = event.key
  keyboardData.pressedKeys.add(event.key)
  
  // 记录按键序列（最多10个）
  keyboardData.keySequence.push(event.key)
  if (keyboardData.keySequence.length > 10) {
    keyboardData.keySequence.shift()
  }
  
  logEvent('keydown', `按键: ${event.key}`)
  
  // 检查快捷键
  checkShortcuts(event)
}

// 键盘松开
function handleKeyUp(event) {
  keyboardData.pressedKeys.delete(event.key)
  logEvent('keyup', `松开: ${event.key}`)
}

// 输入事件
function handleInput(event) {
  keyboardData.inputValue = event.target.value
  logEvent('input', `输入内容: ${event.target.value}`)
}

// 快捷键检查
function checkShortcuts(event) {
  let shortcutKey = ''
  
  if (event.ctrlKey) shortcutKey += 'ctrl+'
  if (event.altKey) shortcutKey += 'alt+'
  if (event.shiftKey) shortcutKey += 'shift+'
  
  shortcutKey += event.key.toLowerCase()
  
  if (keyboardData.shortcuts[shortcutKey]) {
    event.preventDefault()
    logEvent('shortcut', `快捷键: ${shortcutKey} - ${keyboardData.shortcuts[shortcutKey]}`)
  }
}

// 特殊按键处理
function handleSpecialKey(event) {
  switch (event.key) {
    case 'Enter':
      logEvent('special-key', '回车键 - 确认操作')
      break
    case 'Escape':
      logEvent('special-key', 'ESC键 - 取消操作')
      break
    case 'Tab':
      logEvent('special-key', 'Tab键 - 切换焦点')
      break
    case 'Delete':
      logEvent('special-key', 'Delete键 - 删除操作')
      break
    default:
      if (event.key.startsWith('Arrow')) {
        logEvent('special-key', `方向键: ${event.key}`)
      }
  }
}

// ==================== 表单事件 ====================

// 表单提交
function handleSubmit(event) {
  event.preventDefault()
  logEvent('submit', `提交表单: ${JSON.stringify(formData)}`)
}

// 输入框焦点
function handleFocus(event) {
  logEvent('focus', `焦点进入: ${event.target.name || event.target.type}`)
}

// 输入框失焦
function handleBlur(event) {
  logEvent('blur', `失去焦点: ${event.target.name || event.target.type}`)
}

// 下拉选择改变
function handleChange(event) {
  logEvent('change', `选择改变: ${event.target.value}`)
}

// 文件选择
function handleFileSelect(event) {
  const files = Array.from(event.target.files)
  const fileNames = files.map(file => file.name).join(', ')
  logEvent('file-select', `选择文件: ${fileNames}`)
}

// ==================== 自定义事件 ====================

// 发送通知
function sendNotification(type, message) {
  const notification = {
    id: Date.now(),
    type,
    message,
    timestamp: new Date().toLocaleTimeString()
  }
  
  customEvents.notifications.unshift(notification)
  logEvent('custom-notification', `${type}: ${message}`)
  
  // 3秒后自动移除
  setTimeout(() => {
    removeNotification(notification.id)
  }, 3000)
}

// 移除通知
function removeNotification(id) {
  const index = customEvents.notifications.findIndex(n => n.id === id)
  if (index > -1) {
    customEvents.notifications.splice(index, 1)
  }
}

// 游戏得分事件
function handleGameAction(action, points = 0) {
  customEvents.gameScore += points
  
  const message = {
    id: Date.now(),
    action,
    points,
    totalScore: customEvents.gameScore,
    timestamp: new Date().toLocaleTimeString()
  }
  
  customEvents.messages.unshift(message)
  logEvent('game-action', `${action}: +${points}分, 总分: ${customEvents.gameScore}`)
  
  // 检查成就
  checkAchievements()
}

// 检查成就
function checkAchievements() {
  const score = customEvents.gameScore
  const achievements = [
    { threshold: 100, title: '初学者', description: '获得100分' },
    { threshold: 500, title: '进阶者', description: '获得500分' },
    { threshold: 1000, title: '专家', description: '获得1000分' },
    { threshold: 2000, title: '大师', description: '获得2000分' }
  ]
  
  achievements.forEach(achievement => {
    if (score >= achievement.threshold && 
        !customEvents.achievements.some(a => a.title === achievement.title)) {
      customEvents.achievements.push({
        ...achievement,
        unlockedAt: new Date().toLocaleTimeString()
      })
      sendNotification('success', `🏆 解锁成就: ${achievement.title}`)
    }
  })
}

// ==================== 事件修饰符演示 ====================

// 冒泡事件演示
function handleBubbleParent() {
  modifiersDemo.bubbleLog.push('父元素被点击')
}

function handleBubbleChild() {
  modifiersDemo.bubbleLog.push('子元素被点击')
}

function handleBubbleChildStop() {
  modifiersDemo.bubbleLog.push('子元素被点击（阻止冒泡）')
}

// 阻止默认行为
function handlePreventDefault(event) {
  modifiersDemo.preventDefaultLog.push('链接点击被阻止')
}

// once 修饰符
function handleOnce() {
  modifiersDemo.onceTriggered = true
  logEvent('once-modifier', '这个事件只会触发一次')
}

// passive 修饰符
function handlePassiveScroll() {
  modifiersDemo.passiveScrollCount++
}

// ==================== 动态事件 ====================

// 动态添加事件监听
function toggleDynamicEvent() {
  dynamicEvents.isListening = !dynamicEvents.isListening
  
  if (dynamicEvents.isListening) {
    logEvent('dynamic-event', `开始监听 ${dynamicEvents.eventType} 事件`)
  } else {
    logEvent('dynamic-event', `停止监听 ${dynamicEvents.eventType} 事件`)
  }
}

// 动态事件处理
function handleDynamicEvent(event) {
  if (dynamicEvents.isListening) {
    dynamicEvents.dynamicEventLog.unshift({
      type: event.type,
      timestamp: new Date().toLocaleTimeString(),
      details: `事件被动态监听器捕获`
    })
  }
}

// 清空日志
function clearEventLog() {
  eventLog.value = []
  modifiersDemo.bubbleLog = []
  modifiersDemo.preventDefaultLog = []
  dynamicEvents.dynamicEventLog = []
  customEvents.messages = []
}

// 重置所有数据
function resetAllData() {
  counter.value = 0
  mouseData.hoverCount = 0
  mouseData.clickCount = 0
  keyboardData.keySequence = []
  keyboardData.inputValue = ''
  customEvents.gameScore = 0
  customEvents.achievements = []
  customEvents.notifications = []
  modifiersDemo.onceTriggered = false
  modifiersDemo.passiveScrollCount = 0
  clearEventLog()
}
</script>

<template>
  <div class="event-handling">
    <h2>🎯 事件处理详解</h2>
    
    <!-- 基础事件处理 -->
    <div class="section">
      <h3>1. 基础事件处理</h3>
      <div class="card">
        <div class="demo-group">
          <h4>点击事件处理</h4>
          
          <div class="basic-events">
            <!-- 方法处理器 -->
            <div class="event-example">
              <h5>方法处理器</h5>
              <button @click="handleBasicClick" class="demo-btn">
                点击计数 ({{ counter }})
              </button>
              <p class="description">使用方法处理器，适合复杂逻辑</p>
            </div>
            
            <!-- 内联处理器 -->
            <div class="event-example">
              <h5>内联处理器</h5>
              <button @click="counter++" class="demo-btn">
                内联 +1 ({{ counter }})
              </button>
              <button @click="handleInlineClick('内联事件被触发')" class="demo-btn">
                内联带参数
              </button>
              <p class="description">简单逻辑可以直接内联</p>
            </div>
            
            <!-- 带参数的事件处理 -->
            <div class="event-example">
              <h5>带参数的事件处理</h5>
              <button @click="handleParameterClick('参数A', 42, $event)" class="demo-btn">
                传递参数和事件对象
              </button>
              <p class="description">$event 可以获取原始事件对象</p>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 鼠标事件 -->
    <div class="section">
      <h3>2. 鼠标事件</h3>
      <div class="card">
        <div class="demo-group">
          <h4>鼠标交互演示</h4>
          
          <div class="mouse-demo">
            <div class="mouse-info">
              <p>鼠标位置: ({{ mouseData.position.x }}, {{ mouseData.position.y }})</p>
              <p>鼠标状态: {{ mouseData.isMouseDown ? '按下' : '松开' }}</p>
              <p>拖拽状态: {{ mouseData.dragData.isDragging ? '正在拖拽' : '未拖拽' }}</p>
              <p>悬停次数: {{ mouseData.hoverCount }}</p>
              <p>点击总数: {{ mouseData.clickCount }}</p>
            </div>
            
            <div 
              class="mouse-area"
              @mousemove="handleMouseMove"
              @mousedown="handleMouseDown"
              @mouseup="handleMouseUp"
              @mouseenter="handleMouseEnter"
              @mouseleave="handleMouseLeave"
              @click="mouseData.clickCount++"
              @dblclick="handleDoubleClick"
            >
              <div class="mouse-instructions">
                <p>🖱️ 在这个区域内测试鼠标事件</p>
                <p>• 移动鼠标查看坐标</p>
                <p>• 按住鼠标拖拽</p>
                <p>• 单击和双击</p>
                <p>• 鼠标进入和离开</p>
              </div>
              
              <div 
                v-if="mouseData.dragData.isDragging" 
                class="drag-indicator"
                :style="{
                  left: mouseData.position.x - 50 + 'px',
                  top: mouseData.position.y - 50 + 'px'
                }"
              >
                拖拽中...
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 键盘事件 -->
    <div class="section">
      <h3>3. 键盘事件</h3>
      <div class="card">
        <div class="demo-group">
          <h4>键盘输入和快捷键</h4>
          
          <div class="keyboard-demo">
            <div class="keyboard-info">
              <p>最后按键: {{ keyboardData.lastKey }}</p>
              <p>按键序列: {{ keyboardData.keySequence.join(' → ') }}</p>
              <p>当前按下: {{ Array.from(keyboardData.pressedKeys).join(', ') }}</p>
            </div>
            
            <div class="keyboard-inputs">
              <div class="input-group">
                <label>基础输入框:</label>
                <input 
                  v-model="keyboardData.inputValue"
                  @keydown="handleKeyDown"
                  @keyup="handleKeyUp"
                  @input="handleInput"
                  placeholder="在这里输入，观察按键事件"
                  class="demo-input"
                >
              </div>
              
              <div class="input-group">
                <label>特殊按键测试:</label>
                <input 
                  @keydown="handleSpecialKey"
                  placeholder="尝试按 Enter, Esc, Tab, 方向键等"
                  class="demo-input"
                >
              </div>
              
              <div class="shortcuts-info">
                <h5>支持的快捷键:</h5>
                <div class="shortcuts-list">
                  <span v-for="(desc, key) in keyboardData.shortcuts" :key="key" class="shortcut-item">
                    <kbd>{{ key }}</kbd> = {{ desc }}
                  </span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 表单事件 -->
    <div class="section">
      <h3>4. 表单事件</h3>
      <div class="card">
        <div class="demo-group">
          <h4>表单输入事件</h4>
          
          <form @submit="handleSubmit" class="demo-form">
            <div class="form-row">
              <label>用户名:</label>
              <input 
                v-model="formData.username"
                @focus="handleFocus"
                @blur="handleBlur"
                name="username"
                placeholder="输入用户名"
                class="demo-input"
              >
            </div>
            
            <div class="form-row">
              <label>邮箱:</label>
              <input 
                v-model="formData.email"
                @focus="handleFocus"
                @blur="handleBlur"
                type="email"
                name="email"
                placeholder="输入邮箱"
                class="demo-input"
              >
            </div>
            
            <div class="form-row">
              <label>选择选项:</label>
              <select v-model="formData.selectedOption" @change="handleChange" class="demo-select">
                <option value="">请选择</option>
                <option value="option1">选项1</option>
                <option value="option2">选项2</option>
                <option value="option3">选项3</option>
              </select>
            </div>
            
            <div class="form-row">
              <label>消息:</label>
              <textarea 
                v-model="formData.message"
                @focus="handleFocus"
                @blur="handleBlur"
                name="message"
                placeholder="输入消息"
                class="demo-textarea"
              ></textarea>
            </div>
            
            <div class="form-row">
              <label>文件上传:</label>
              <input type="file" @change="handleFileSelect" multiple class="demo-input">
            </div>
            
            <div class="form-row">
              <label>
                <input type="checkbox" v-model="formData.agreedToTerms">
                同意服务条款
              </label>
            </div>
            
            <div class="form-row">
              <button type="submit" class="submit-btn">提交表单</button>
            </div>
          </form>
        </div>
      </div>
    </div>
    
    <!-- 自定义事件 -->
    <div class="section">
      <h3>5. 自定义事件系统</h3>
      <div class="card">
        <div class="demo-group">
          <h4>通知和消息系统</h4>
          
          <div class="custom-events">
            <div class="notification-controls">
              <button @click="sendNotification('info', '这是一条信息')" class="info-btn">
                发送信息
              </button>
              <button @click="sendNotification('success', '操作成功')" class="success-btn">
                发送成功消息
              </button>
              <button @click="sendNotification('warning', '警告信息')" class="warning-btn">
                发送警告
              </button>
              <button @click="sendNotification('error', '错误信息')" class="error-btn">
                发送错误
              </button>
            </div>
            
            <!-- 通知列表 -->
            <div class="notifications-container">
              <transition-group name="notification" tag="div" class="notifications">
                <div 
                  v-for="notification in customEvents.notifications" 
                  :key="notification.id"
                  :class="['notification', notification.type]"
                >
                  <span class="notification-message">{{ notification.message }}</span>
                  <span class="notification-time">{{ notification.timestamp }}</span>
                  <button @click="removeNotification(notification.id)" class="close-btn">×</button>
                </div>
              </transition-group>
            </div>
            
            <!-- 游戏系统 -->
            <div class="game-system">
              <h5>游戏得分系统</h5>
              <div class="game-controls">
                <button @click="handleGameAction('击败敌人', 10)" class="game-btn">
                  击败敌人 (+10分)
                </button>
                <button @click="handleGameAction('收集道具', 5)" class="game-btn">
                  收集道具 (+5分)
                </button>
                <button @click="handleGameAction('完成任务', 50)" class="game-btn">
                  完成任务 (+50分)
                </button>
                <button @click="handleGameAction('发现秘密', 100)" class="game-btn">
                  发现秘密 (+100分)
                </button>
              </div>
              
              <div class="game-status">
                <p class="score">当前分数: {{ customEvents.gameScore }}</p>
                
                <div class="achievements">
                  <h6>已解锁成就:</h6>
                  <div v-if="customEvents.achievements.length === 0" class="no-achievements">
                    暂无成就，继续努力！
                  </div>
                  <div v-else class="achievement-list">
                    <div v-for="achievement in customEvents.achievements" :key="achievement.title" class="achievement">
                      <span class="achievement-title">🏆 {{ achievement.title }}</span>
                      <span class="achievement-desc">{{ achievement.description }}</span>
                      <span class="achievement-time">{{ achievement.unlockedAt }}</span>
                    </div>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 事件修饰符 -->
    <div class="section">
      <h3>6. 事件修饰符</h3>
      <div class="card">
        <div class="demo-group">
          <h4>修饰符演示</h4>
          
          <div class="modifiers-demo">
            <!-- 事件冒泡 -->
            <div class="bubble-demo">
              <h5>事件冒泡演示</h5>
              <div class="bubble-log">
                <div v-for="(log, index) in modifiersDemo.bubbleLog" :key="index" class="log-item">
                  {{ log }}
                </div>
              </div>
              
              <div @click="handleBubbleParent" class="parent-element">
                父元素
                <div @click="handleBubbleChild" class="child-element">
                  子元素（冒泡）
                </div>
                <div @click.stop="handleBubbleChildStop" class="child-element stop">
                  子元素（.stop）
                </div>
              </div>
              <button @click="modifiersDemo.bubbleLog = []" class="clear-btn">清除日志</button>
            </div>
            
            <!-- 阻止默认行为 -->
            <div class="prevent-demo">
              <h5>阻止默认行为</h5>
              <div class="prevent-log">
                <div v-for="(log, index) in modifiersDemo.preventDefaultLog" :key="index" class="log-item">
                  {{ log }}
                </div>
              </div>
              
              <div class="link-demo">
                <a href="https://vuejs.org" target="_blank">正常链接（会跳转）</a>
                <a href="https://vuejs.org" @click.prevent="handlePreventDefault">阻止跳转的链接（.prevent）</a>
              </div>
            </div>
            
            <!-- once 修饰符 -->
            <div class="once-demo">
              <h5>once 修饰符</h5>
              <button @click.once="handleOnce" :disabled="modifiersDemo.onceTriggered" class="once-btn">
                {{ modifiersDemo.onceTriggered ? '已触发（只能点击一次）' : '点击我（只能触发一次）' }}
              </button>
            </div>
            
            <!-- 按键修饰符 -->
            <div class="key-modifiers">
              <h5>按键修饰符</h5>
              <div class="key-inputs">
                <input @keyup.enter="logEvent('key-modifier', 'Enter键被按下')" placeholder="按Enter键" class="demo-input">
                <input @keyup.esc="logEvent('key-modifier', 'Esc键被按下')" placeholder="按Esc键" class="demo-input">
                <input @keyup.space="logEvent('key-modifier', '空格键被按下')" placeholder="按空格键" class="demo-input">
                <input @keyup.ctrl.s="logEvent('key-modifier', 'Ctrl+S被按下')" placeholder="按Ctrl+S" class="demo-input">
              </div>
            </div>
            
            <!-- 鼠标按键修饰符 -->
            <div class="mouse-modifiers">
              <h5>鼠标按键修饰符</h5>
              <div class="mouse-buttons">
                <div @click.left="logEvent('mouse-modifier', '左键点击')" class="mouse-btn">左键点击</div>
                <div @click.right="logEvent('mouse-modifier', '右键点击')" class="mouse-btn">右键点击</div>
                <div @click.middle="logEvent('mouse-modifier', '中键点击')" class="mouse-btn">中键点击</div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 事件日志 -->
    <div class="section">
      <h3>7. 事件日志</h3>
      <div class="card">
        <div class="demo-group">
          <h4>实时事件监控</h4>
          
          <div class="event-log">
            <div class="log-controls">
              <button @click="clearEventLog" class="clear-btn">清除日志</button>
              <button @click="resetAllData" class="reset-btn">重置所有数据</button>
              <span class="log-count">日志数量: {{ eventLog.length }}</span>
            </div>
            
            <div class="log-container">
              <div v-if="eventLog.length === 0" class="empty-log">
                暂无事件日志，与上面的元素交互来生成日志
              </div>
              <div v-else class="log-list">
                <div v-for="log in eventLog" :key="log.id" class="log-entry">
                  <span class="log-time">{{ log.timestamp }}</span>
                  <span class="log-type">{{ log.type }}</span>
                  <span class="log-details">{{ log.details }}</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 总结 -->
    <div class="section summary">
      <h3>📝 事件处理总结</h3>
      <div class="card">
        <div class="summary-grid">
          <div class="summary-item">
            <h4>🎯 基础语法</h4>
            <ul>
              <li><code>@click="handler"</code> - 方法处理器</li>
              <li><code>@click="count++"</code> - 内联处理器</li>
              <li><code>@click="fn($event)"</code> - 传递事件对象</li>
              <li><code>@click="fn(param, $event)"</code> - 带参数</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>🖱️ 鼠标事件</h4>
            <ul>
              <li><code>@click</code> - 点击事件</li>
              <li><code>@dblclick</code> - 双击事件</li>
              <li><code>@mousedown/up</code> - 鼠标按下/松开</li>
              <li><code>@mousemove</code> - 鼠标移动</li>
              <li><code>@mouseenter/leave</code> - 鼠标进入/离开</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>⌨️ 键盘事件</h4>
            <ul>
              <li><code>@keydown/up</code> - 按键按下/松开</li>
              <li><code>@keyup.enter</code> - 回车键</li>
              <li><code>@keyup.esc</code> - ESC键</li>
              <li><code>@keyup.ctrl.s</code> - 组合键</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>🔧 事件修饰符</h4>
            <ul>
              <li><code>.stop</code> - 阻止事件冒泡</li>
              <li><code>.prevent</code> - 阻止默认行为</li>
              <li><code>.once</code> - 只触发一次</li>
              <li><code>.capture</code> - 使用捕获模式</li>
              <li><code>.self</code> - 只在自身触发</li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.event-handling {
  padding: 20px;
  max-width: 1400px;
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

.demo-group h4 {
  color: #333;
  margin-bottom: 15px;
}

/* 基础事件样式 */
.basic-events {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
}

.event-example {
  padding: 15px;
  background: #f9f9f9;
  border-radius: 8px;
  border: 2px solid #ddd;
}

.event-example h5 {
  margin: 0 0 10px 0;
  color: #333;
}

.demo-btn {
  padding: 10px 20px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s;
  margin: 5px;
}

.demo-btn:hover {
  background: #35a372;
  transform: translateY(-1px);
}

.description {
  margin: 10px 0 0 0;
  color: #666;
  font-size: 14px;
  font-style: italic;
}

/* 鼠标事件样式 */
.mouse-demo {
  max-width: 800px;
}

.mouse-info {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 10px;
  margin-bottom: 20px;
  padding: 15px;
  background: #f5f5f5;
  border-radius: 8px;
}

.mouse-info p {
  margin: 5px 0;
  font-family: monospace;
  background: white;
  padding: 5px 10px;
  border-radius: 4px;
}

.mouse-area {
  position: relative;
  width: 100%;
  height: 300px;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-radius: 8px;
  display: flex;
  align-items: center;
  justify-content: center;
  cursor: crosshair;
  user-select: none;
  overflow: hidden;
}

.mouse-instructions {
  text-align: center;
  color: white;
  background: rgba(0, 0, 0, 0.3);
  padding: 20px;
  border-radius: 8px;
}

.mouse-instructions p {
  margin: 5px 0;
}

.drag-indicator {
  position: fixed;
  background: #ff6b6b;
  color: white;
  padding: 5px 10px;
  border-radius: 20px;
  font-size: 12px;
  pointer-events: none;
  z-index: 1000;
  transform: translate(-50%, -50%);
}

/* 键盘事件样式 */
.keyboard-demo {
  max-width: 800px;
}

.keyboard-info {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 10px;
  margin-bottom: 20px;
  padding: 15px;
  background: #f5f5f5;
  border-radius: 8px;
}

.keyboard-info p {
  margin: 5px 0;
  font-family: monospace;
  background: white;
  padding: 5px 10px;
  border-radius: 4px;
}

.keyboard-inputs {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.input-group {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.input-group label {
  font-weight: bold;
  color: #333;
}

.demo-input {
  padding: 10px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.demo-input:focus {
  outline: none;
  border-color: #42b883;
}

.shortcuts-info {
  margin-top: 20px;
}

.shortcuts-info h5 {
  margin: 0 0 10px 0;
  color: #333;
}

.shortcuts-list {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
}

.shortcut-item {
  display: flex;
  align-items: center;
  gap: 5px;
  padding: 5px 10px;
  background: #f0f0f0;
  border-radius: 4px;
  font-size: 14px;
}

kbd {
  background: #333;
  color: white;
  padding: 2px 6px;
  border-radius: 3px;
  font-family: monospace;
  font-size: 12px;
}

/* 表单事件样式 */
.demo-form {
  max-width: 600px;
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.form-row {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.form-row label {
  font-weight: bold;
  color: #333;
}

.demo-select {
  padding: 10px;
  border: 2px solid #ddd;
  border-radius: 4px;
  background: white;
  font-size: 14px;
}

.demo-textarea {
  padding: 10px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  min-height: 80px;
  resize: vertical;
}

.submit-btn {
  padding: 12px 24px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  transition: background 0.3s;
}

.submit-btn:hover {
  background: #35a372;
}

/* 自定义事件样式 */
.custom-events {
  max-width: 800px;
}

.notification-controls {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.info-btn {
  background: #2196f3;
}

.success-btn {
  background: #4caf50;
}

.warning-btn {
  background: #ff9800;
}

.error-btn {
  background: #f44336;
}

.notifications-container {
  margin-bottom: 30px;
}

.notifications {
  display: flex;
  flex-direction: column;
  gap: 10px;
  max-height: 300px;
  overflow-y: auto;
}

.notification {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px;
  border-radius: 4px;
  transition: all 0.3s;
}

.notification.info {
  background: #e3f2fd;
  border-left: 4px solid #2196f3;
  color: #1565c0;
}

.notification.success {
  background: #e8f5e9;
  border-left: 4px solid #4caf50;
  color: #2e7d32;
}

.notification.warning {
  background: #fff3e0;
  border-left: 4px solid #ff9800;
  color: #e65100;
}

.notification.error {
  background: #ffebee;
  border-left: 4px solid #f44336;
  color: #c62828;
}

.notification-message {
  flex: 1;
}

.notification-time {
  font-size: 12px;
  opacity: 0.7;
}

.close-btn {
  background: none;
  border: none;
  font-size: 18px;
  cursor: pointer;
  padding: 0;
  width: 20px;
  height: 20px;
  display: flex;
  align-items: center;
  justify-content: center;
  border-radius: 50%;
  transition: background 0.3s;
}

.close-btn:hover {
  background: rgba(0, 0, 0, 0.1);
}

/* 通知动画 */
.notification-enter-active,
.notification-leave-active {
  transition: all 0.3s;
}

.notification-enter-from {
  opacity: 0;
  transform: translateX(100px);
}

.notification-leave-to {
  opacity: 0;
  transform: translateX(-100px);
}

/* 游戏系统样式 */
.game-system {
  margin-top: 20px;
  padding: 20px;
  background: #f5f5f5;
  border-radius: 8px;
}

.game-system h5 {
  margin: 0 0 15px 0;
  color: #333;
}

.game-controls {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
  flex-wrap: wrap;
}

.game-btn {
  padding: 8px 16px;
  background: #673ab7;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s;
}

.game-btn:hover {
  background: #5e35b1;
}

.game-status {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.score {
  font-size: 18px;
  font-weight: bold;
  color: #42b883;
  margin: 0;
}

.achievements h6 {
  margin: 0 0 10px 0;
  color: #333;
}

.no-achievements {
  color: #999;
  font-style: italic;
}

.achievement-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.achievement {
  display: flex;
  flex-direction: column;
  gap: 2px;
  padding: 10px;
  background: linear-gradient(135deg, #ffd700, #ffed4e);
  border-radius: 4px;
  border: 1px solid #f39c12;
}

.achievement-title {
  font-weight: bold;
  color: #d68910;
}

.achievement-desc {
  font-size: 14px;
  color: #b7950b;
}

.achievement-time {
  font-size: 12px;
  color: #85650d;
}

/* 事件修饰符样式 */
.modifiers-demo {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
}

.bubble-demo,
.prevent-demo,
.once-demo,
.key-modifiers,
.mouse-modifiers {
  padding: 15px;
  background: #f9f9f9;
  border-radius: 8px;
  border: 2px solid #ddd;
}

.bubble-demo h5,
.prevent-demo h5,
.once-demo h5,
.key-modifiers h5,
.mouse-modifiers h5 {
  margin: 0 0 10px 0;
  color: #333;
}

.bubble-log,
.prevent-log {
  min-height: 60px;
  max-height: 100px;
  overflow-y: auto;
  background: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 10px;
  margin-bottom: 10px;
}

.log-item {
  padding: 2px 0;
  border-bottom: 1px solid #eee;
  font-size: 14px;
  color: #333;
}

.parent-element {
  background: #e3f2fd;
  border: 2px solid #2196f3;
  padding: 20px;
  border-radius: 8px;
  cursor: pointer;
  transition: background 0.3s;
}

.parent-element:hover {
  background: #bbdefb;
}

.child-element {
  background: #fff3e0;
  border: 2px solid #ff9800;
  padding: 10px;
  margin: 10px;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s;
}

.child-element:hover {
  background: #ffe0b2;
}

.child-element.stop {
  background: #e8f5e9;
  border-color: #4caf50;
}

.child-element.stop:hover {
  background: #c8e6c9;
}

.link-demo {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.link-demo a {
  display: inline-block;
  padding: 8px 16px;
  background: #2196f3;
  color: white;
  text-decoration: none;
  border-radius: 4px;
  transition: background 0.3s;
}

.link-demo a:hover {
  background: #1976d2;
}

.once-btn {
  padding: 10px 20px;
  background: #ff9800;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s;
}

.once-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.key-inputs {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.mouse-buttons {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.mouse-btn {
  padding: 10px 15px;
  background: #9c27b0;
  color: white;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s;
  user-select: none;
}

.mouse-btn:hover {
  background: #7b1fa2;
}

.clear-btn {
  background: #f44336;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
}

/* 事件日志样式 */
.event-log {
  max-width: 1000px;
}

.log-controls {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
  align-items: center;
  flex-wrap: wrap;
}

.reset-btn {
  background: #ff9800;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
}

.log-count {
  background: #f0f0f0;
  padding: 6px 12px;
  border-radius: 4px;
  font-size: 14px;
  color: #333;
}

.log-container {
  max-height: 400px;
  overflow-y: auto;
  border: 1px solid #ddd;
  border-radius: 4px;
  background: white;
}

.empty-log {
  padding: 40px 20px;
  text-align: center;
  color: #999;
  font-style: italic;
}

.log-list {
  display: flex;
  flex-direction: column;
}

.log-entry {
  display: grid;
  grid-template-columns: 100px 120px 1fr;
  gap: 15px;
  padding: 10px 15px;
  border-bottom: 1px solid #eee;
  font-size: 14px;
  align-items: center;
}

.log-entry:nth-child(even) {
  background: #f9f9f9;
}

.log-time {
  color: #666;
  font-family: monospace;
  font-size: 12px;
}

.log-type {
  background: #42b883;
  color: white;
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 12px;
  text-align: center;
  white-space: nowrap;
}

.log-details {
  color: #333;
}

/* 总结样式 */
.summary {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  padding: 20px;
  border-radius: 8px;
}

.summary h3 {
  color: white;
}

.summary-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
}

.summary-item {
  background: rgba(255, 255, 255, 0.1);
  padding: 15px;
  border-radius: 8px;
}

.summary-item h4 {
  color: white;
  margin-bottom: 10px;
}

.summary-item ul {
  list-style: none;
  padding: 0;
  margin: 0;
}

.summary-item li {
  padding: 5px 0;
  color: rgba(255, 255, 255, 0.9);
}

.summary-item code {
  background: rgba(255, 255, 255, 0.2);
  padding: 2px 6px;
  border-radius: 3px;
  color: white;
  font-family: 'Courier New', monospace;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .basic-events {
    grid-template-columns: 1fr;
  }
  
  .modifiers-demo {
    grid-template-columns: 1fr;
  }
  
  .mouse-buttons {
    flex-direction: column;
  }
  
  .game-controls {
    flex-direction: column;
  }
  
  .notification-controls {
    flex-direction: column;
  }
  
  .log-entry {
    grid-template-columns: 1fr;
    gap: 5px;
  }
  
  .log-entry .log-time,
  .log-entry .log-type {
    justify-self: start;
  }
  
  .summary-grid {
    grid-template-columns: 1fr;
  }
  
  .log-controls {
    flex-direction: column;
    align-items: stretch;
  }
}
</style>