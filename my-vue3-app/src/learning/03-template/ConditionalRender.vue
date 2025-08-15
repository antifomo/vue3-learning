<script setup>
// Day 3 - 条件渲染深入学习
import { ref, reactive, computed } from 'vue'

// ==================== 基础状态 ====================

// v-if vs v-show 对比
const showWithIf = ref(true)
const showWithShow = ref(true)

// 多条件分支
const userRole = ref('guest')  // guest, user, admin, vip
const loginStatus = ref('loggedOut')  // loggedOut, loggedIn, expired

// 复杂条件示例
const weather = reactive({
  temperature: 25,
  humidity: 60,
  isRaining: false,
  windSpeed: 5
})

// 用户权限系统
const user = reactive({
  name: '',
  age: 18,
  isLoggedIn: false,
  permissions: [],
  subscription: 'free'  // free, premium, enterprise
})

// 游戏状态
const gameState = reactive({
  isPlaying: false,
  level: 1,
  score: 0,
  lives: 3,
  powerUps: [],
  gameMode: 'normal'  // normal, hard, nightmare
})

// 表单验证状态
const formValidation = reactive({
  username: { value: '', isValid: true, error: '' },
  email: { value: '', isValid: true, error: '' },
  password: { value: '', isValid: true, error: '' },
  confirmPassword: { value: '', isValid: true, error: '' }
})

// 性能测试
const performanceTest = reactive({
  itemCount: 1000,
  showItems: true,
  toggleCount: 0,
  renderType: 'v-if'  // v-if 或 v-show
})

// ==================== 计算属性 ====================

// 天气状态判断
const weatherStatus = computed(() => {
  if (weather.isRaining) {
    return 'rainy'
  } else if (weather.temperature > 30) {
    return 'hot'
  } else if (weather.temperature < 10) {
    return 'cold'
  } else if (weather.humidity > 80) {
    return 'humid'
  } else {
    return 'pleasant'
  }
})

// 天气建议
const weatherAdvice = computed(() => {
  const status = weatherStatus.value
  switch (status) {
    case 'rainy':
      return '记得带伞！'
    case 'hot':
      return '天气炎热，注意防晒！'
    case 'cold':
      return '天气寒冷，多穿衣服！'
    case 'humid':
      return '湿度较高，注意通风！'
    default:
      return '天气不错，适合出行！'
  }
})

// 用户等级
const userLevel = computed(() => {
  if (user.age < 13) return 'child'
  if (user.age < 18) return 'teen'
  if (user.age < 60) return 'adult'
  return 'senior'
})

// 表单整体验证状态
const isFormValid = computed(() => {
  return Object.values(formValidation).every(field => field.isValid)
})

// 游戏难度描述
const difficultyDescription = computed(() => {
  const { level, gameMode } = gameState
  
  if (gameMode === 'nightmare') {
    return '噩梦模式：极端困难！'
  } else if (gameMode === 'hard') {
    return '困难模式：挑战性很高'
  } else {
    if (level <= 3) return '新手友好'
    if (level <= 6) return '中等难度'
    if (level <= 9) return '有挑战性'
    return '专家级别'
  }
})

// ==================== 方法 ====================

// 切换显示状态
function toggleIf() {
  showWithIf.value = !showWithIf.value
}

function toggleShow() {
  showWithShow.value = !showWithShow.value
}

// 用户角色切换
function changeUserRole() {
  const roles = ['guest', 'user', 'admin', 'vip']
  const currentIndex = roles.indexOf(userRole.value)
  const nextIndex = (currentIndex + 1) % roles.length
  userRole.value = roles[nextIndex]
}

// 登录状态切换
function toggleLoginStatus() {
  const statuses = ['loggedOut', 'loggedIn', 'expired']
  const currentIndex = statuses.indexOf(loginStatus.value)
  const nextIndex = (currentIndex + 1) % statuses.length
  loginStatus.value = statuses[nextIndex]
}

// 天气控制
function toggleRain() {
  weather.isRaining = !weather.isRaining
}

function increaseTemperature() {
  weather.temperature += 5
}

function decreaseTemperature() {
  weather.temperature -= 5
}

function increaseHumidity() {
  weather.humidity = Math.min(100, weather.humidity + 10)
}

function decreaseHumidity() {
  weather.humidity = Math.max(0, weather.humidity - 10)
}

// 用户系统
function toggleLogin() {
  user.isLoggedIn = !user.isLoggedIn
  if (user.isLoggedIn) {
    user.name = 'John Doe'
    user.permissions = ['read', 'write']
  } else {
    user.name = ''
    user.permissions = []
  }
}

function upgradeSubscription() {
  const subscriptions = ['free', 'premium', 'enterprise']
  const currentIndex = subscriptions.indexOf(user.subscription)
  const nextIndex = (currentIndex + 1) % subscriptions.length
  user.subscription = subscriptions[nextIndex]
}

function addPermission() {
  const permissions = ['read', 'write', 'delete', 'admin', 'moderate']
  const availablePermissions = permissions.filter(p => !user.permissions.includes(p))
  if (availablePermissions.length > 0) {
    const randomPermission = availablePermissions[Math.floor(Math.random() * availablePermissions.length)]
    user.permissions.push(randomPermission)
  }
}

// 游戏控制
function startGame() {
  gameState.isPlaying = true
  gameState.score = 0
  gameState.lives = 3
  gameState.level = 1
}

function stopGame() {
  gameState.isPlaying = false
}

function levelUp() {
  gameState.level++
  gameState.score += 100
}

function loseLife() {
  if (gameState.lives > 0) {
    gameState.lives--
    if (gameState.lives === 0) {
      gameState.isPlaying = false
    }
  }
}

function changeGameMode() {
  const modes = ['normal', 'hard', 'nightmare']
  const currentIndex = modes.indexOf(gameState.gameMode)
  const nextIndex = (currentIndex + 1) % modes.length
  gameState.gameMode = modes[nextIndex]
}

// 表单验证
function validateUsername() {
  const username = formValidation.username.value
  if (username.length < 3) {
    formValidation.username.isValid = false
    formValidation.username.error = '用户名至少3个字符'
  } else if (username.length > 20) {
    formValidation.username.isValid = false
    formValidation.username.error = '用户名不能超过20个字符'
  } else {
    formValidation.username.isValid = true
    formValidation.username.error = ''
  }
}

function validateEmail() {
  const email = formValidation.email.value
  const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/
  if (!emailRegex.test(email)) {
    formValidation.email.isValid = false
    formValidation.email.error = '请输入有效的邮箱地址'
  } else {
    formValidation.email.isValid = true
    formValidation.email.error = ''
  }
}

function validatePassword() {
  const password = formValidation.password.value
  if (password.length < 6) {
    formValidation.password.isValid = false
    formValidation.password.error = '密码至少6个字符'
  } else {
    formValidation.password.isValid = true
    formValidation.password.error = ''
  }
  
  // 重新验证确认密码
  validateConfirmPassword()
}

function validateConfirmPassword() {
  const password = formValidation.password.value
  const confirmPassword = formValidation.confirmPassword.value
  if (password !== confirmPassword) {
    formValidation.confirmPassword.isValid = false
    formValidation.confirmPassword.error = '两次输入的密码不一致'
  } else {
    formValidation.confirmPassword.isValid = true
    formValidation.confirmPassword.error = ''
  }
}

// 性能测试
function performanceToggle() {
  performanceTest.showItems = !performanceTest.showItems
  performanceTest.toggleCount++
}

function changeRenderType() {
  performanceTest.renderType = performanceTest.renderType === 'v-if' ? 'v-show' : 'v-if'
  performanceTest.toggleCount = 0
}
</script>

<template>
  <div class="conditional-render">
    <h2>🔀 条件渲染深入学习</h2>
    
    <!-- v-if vs v-show 对比 -->
    <div class="section">
      <h3>1. v-if vs v-show 对比</h3>
      <div class="card">
        <div class="demo-group">
          <h4>基础对比</h4>
          
          <div class="comparison">
            <div class="compare-item">
              <h5>v-if（条件渲染）</h5>
              <div v-if="showWithIf" class="demo-box if-box">
                ✅ v-if: 我在DOM中存在
              </div>
              <div v-else class="demo-box else-box">
                ❌ v-else: showWithIf = false
              </div>
              <button @click="toggleIf">
                切换 v-if ({{ showWithIf ? '显示' : '隐藏' }})
              </button>
              <p class="tip">💡 v-if: {{ showWithIf ? '元素在DOM中' : '元素不在DOM中' }}</p>
            </div>
            
            <div class="compare-item">
              <h5>v-show（显示控制）</h5>
              <div v-show="showWithShow" class="demo-box show-box">
                👁️ v-show: 我始终在DOM中，只是显示/隐藏
              </div>
              <button @click="toggleShow">
                切换 v-show ({{ showWithShow ? '显示' : '隐藏' }})
              </button>
              <p class="tip">💡 v-show: 元素始终在DOM中，通过CSS控制显示</p>
            </div>
          </div>
          
          <div class="performance-note">
            <h5>性能对比</h5>
            <ul>
              <li>🚀 <strong>v-show</strong>: 更高的初始渲染开销，但切换开销很小</li>
              <li>⚡ <strong>v-if</strong>: 更低的初始渲染开销，但切换开销较大</li>
              <li>📋 <strong>使用建议</strong>: 频繁切换用v-show，条件很少改变用v-if</li>
            </ul>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 多条件分支 -->
    <div class="section">
      <h3>2. 多条件分支</h3>
      <div class="card">
        <div class="demo-group">
          <h4>v-if / v-else-if / v-else 链式条件</h4>
          
          <div class="multi-condition">
            <div class="user-role-demo">
              <h5>用户角色权限系统</h5>
              
              <!-- 基于用户角色的条件渲染 -->
              <div v-if="userRole === 'guest'" class="role-card guest">
                👤 游客模式
                <p>仅能浏览基础内容</p>
                <div class="permissions">
                  <span class="permission">浏览</span>
                </div>
              </div>
              
              <div v-else-if="userRole === 'user'" class="role-card user">
                🙋‍♂️ 普通用户
                <p>可以发表评论和收藏</p>
                <div class="permissions">
                  <span class="permission">浏览</span>
                  <span class="permission">评论</span>
                  <span class="permission">收藏</span>
                </div>
              </div>
              
              <div v-else-if="userRole === 'admin'" class="role-card admin">
                👑 管理员
                <p>拥有管理权限</p>
                <div class="permissions">
                  <span class="permission">浏览</span>
                  <span class="permission">评论</span>
                  <span class="permission">收藏</span>
                  <span class="permission">管理</span>
                  <span class="permission">删除</span>
                </div>
              </div>
              
              <div v-else-if="userRole === 'vip'" class="role-card vip">
                💎 VIP用户
                <p>享受专属服务</p>
                <div class="permissions">
                  <span class="permission">浏览</span>
                  <span class="permission">评论</span>
                  <span class="permission">收藏</span>
                  <span class="permission">专享内容</span>
                  <span class="permission">优先支持</span>
                </div>
              </div>
              
              <div v-else class="role-card unknown">
                ❓ 未知角色
                <p>请联系管理员</p>
              </div>
              
              <div class="controls">
                <button @click="changeUserRole">切换角色</button>
                <span class="current-role">当前: {{ userRole }}</span>
              </div>
            </div>
            
            <div class="login-status-demo">
              <h5>登录状态管理</h5>
              
              <div v-if="loginStatus === 'loggedOut'" class="status-card logged-out">
                🚪 未登录
                <p>请登录以获取完整功能</p>
                <button class="login-btn">登录</button>
              </div>
              
              <div v-else-if="loginStatus === 'loggedIn'" class="status-card logged-in">
                ✅ 已登录
                <p>欢迎回来！享受所有功能</p>
                <button class="logout-btn">退出登录</button>
              </div>
              
              <div v-else-if="loginStatus === 'expired'" class="status-card expired">
                ⏰ 登录已过期
                <p>为了您的安全，请重新登录</p>
                <button class="relogin-btn">重新登录</button>
              </div>
              
              <div class="controls">
                <button @click="toggleLoginStatus">切换状态</button>
                <span class="current-status">当前: {{ loginStatus }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 复杂条件逻辑 -->
    <div class="section">
      <h3>3. 复杂条件逻辑</h3>
      <div class="card">
        <div class="demo-group">
          <h4>多维度条件判断</h4>
          
          <div class="weather-system">
            <h5>智能天气系统</h5>
            
            <div class="weather-info">
              <div class="weather-params">
                <div class="param">
                  <label>温度: {{ weather.temperature }}°C</label>
                  <div class="param-controls">
                    <button @click="decreaseTemperature">-</button>
                    <button @click="increaseTemperature">+</button>
                  </div>
                </div>
                
                <div class="param">
                  <label>湿度: {{ weather.humidity }}%</label>
                  <div class="param-controls">
                    <button @click="decreaseHumidity">-</button>
                    <button @click="increaseHumidity">+</button>
                  </div>
                </div>
                
                <div class="param">
                  <label>是否下雨</label>
                  <button @click="toggleRain">
                    {{ weather.isRaining ? '☔ 雨天' : '☀️ 晴天' }}
                  </button>
                </div>
              </div>
              
              <!-- 基于复杂条件的天气状态 -->
              <div class="weather-status">
                <div v-if="weather.isRaining" class="weather-card rainy">
                  ☔ 雨天
                  <p>{{ weatherAdvice }}</p>
                  <div v-if="weather.temperature < 15" class="weather-detail">
                    🧥 寒冷的雨天，记得保暖
                  </div>
                  <div v-else-if="weather.temperature > 25" class="weather-detail">
                    💧 温暖的雨天，空气清新
                  </div>
                </div>
                
                <div v-else-if="weatherStatus === 'hot'" class="weather-card hot">
                  🔥 炎热
                  <p>{{ weatherAdvice }}</p>
                  <div v-if="weather.humidity > 70" class="weather-detail">
                    💦 又热又湿，注意防暑
                  </div>
                </div>
                
                <div v-else-if="weatherStatus === 'cold'" class="weather-card cold">
                  ❄️ 寒冷
                  <p>{{ weatherAdvice }}</p>
                  <div v-if="weather.humidity < 30" class="weather-detail">
                    🍂 干冷天气，注意保湿
                  </div>
                </div>
                
                <div v-else-if="weatherStatus === 'humid'" class="weather-card humid">
                  💨 湿润
                  <p>{{ weatherAdvice }}</p>
                </div>
                
                <div v-else class="weather-card pleasant">
                  😊 舒适
                  <p>{{ weatherAdvice }}</p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 嵌套条件和权限系统 -->
    <div class="section">
      <h3>4. 嵌套条件和权限系统</h3>
      <div class="card">
        <div class="demo-group">
          <h4>用户权限和订阅系统</h4>
          
          <div class="user-system">
            <div class="user-info">
              <h5>用户信息</h5>
              
              <!-- 登录状态检查 -->
              <div v-if="user.isLoggedIn">
                <div class="user-profile">
                  <h6>👋 欢迎, {{ user.name }}!</h6>
                  <p>年龄分组: {{ userLevel }}</p>
                  <p>订阅类型: {{ user.subscription }}</p>
                  
                  <!-- 基于订阅类型的功能 -->
                  <div class="subscription-features">
                    <div v-if="user.subscription === 'free'" class="subscription-card free">
                      🆓 免费版
                      <ul>
                        <li>基础功能</li>
                        <li>有广告</li>
                        <li v-if="userLevel === 'child'">儿童安全模式</li>
                      </ul>
                    </div>
                    
                    <div v-else-if="user.subscription === 'premium'" class="subscription-card premium">
                      ⭐ 高级版
                      <ul>
                        <li>所有基础功能</li>
                        <li>无广告</li>
                        <li>高清内容</li>
                        <li v-if="userLevel === 'adult'">成人内容解锁</li>
                      </ul>
                    </div>
                    
                    <div v-else-if="user.subscription === 'enterprise'" class="subscription-card enterprise">
                      🏢 企业版
                      <ul>
                        <li>所有高级功能</li>
                        <li>团队管理</li>
                        <li>优先支持</li>
                        <li>数据分析</li>
                      </ul>
                    </div>
                  </div>
                  
                  <!-- 权限显示 -->
                  <div class="permissions-display">
                    <h6>当前权限:</h6>
                    <div v-if="user.permissions.length > 0" class="permission-list">
                      <span 
                        v-for="permission in user.permissions" 
                        :key="permission"
                        class="permission-tag"
                      >
                        {{ permission }}
                      </span>
                    </div>
                    <p v-else class="no-permissions">暂无特殊权限</p>
                  </div>
                  
                  <!-- 基于权限的操作按钮 -->
                  <div class="actions">
                    <button v-if="user.permissions.includes('read')" class="action-btn read">
                      📖 阅读内容
                    </button>
                    <button v-if="user.permissions.includes('write')" class="action-btn write">
                      ✍️ 发表内容
                    </button>
                    <button v-if="user.permissions.includes('delete')" class="action-btn delete">
                      🗑️ 删除内容
                    </button>
                    <button v-if="user.permissions.includes('admin')" class="action-btn admin">
                      ⚙️ 管理面板
                    </button>
                  </div>
                </div>
              </div>
              
              <div v-else class="login-prompt">
                <h6>🔐 请先登录</h6>
                <p>登录后可享受个性化服务</p>
              </div>
              
              <div class="user-controls">
                <button @click="toggleLogin">
                  {{ user.isLoggedIn ? '退出登录' : '登录' }}
                </button>
                <button v-if="user.isLoggedIn" @click="upgradeSubscription">
                  升级订阅
                </button>
                <button v-if="user.isLoggedIn" @click="addPermission">
                  添加权限
                </button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 动态内容和游戏状态 -->
    <div class="section">
      <h3>5. 动态内容和游戏状态</h3>
      <div class="card">
        <div class="demo-group">
          <h4>游戏状态管理</h4>
          
          <div class="game-system">
            <!-- 游戏开始前 -->
            <div v-if="!gameState.isPlaying" class="game-start">
              <h5>🎮 游戏大厅</h5>
              <p>准备开始新游戏？</p>
              
              <div class="game-mode-selection">
                <h6>选择难度:</h6>
                <button 
                  @click="changeGameMode"
                  :class="['mode-btn', gameState.gameMode]"
                >
                  {{ gameState.gameMode === 'normal' ? '🟢 简单' : 
                     gameState.gameMode === 'hard' ? '🟡 困难' : '🔴 噩梦' }}
                </button>
                <p class="difficulty-desc">{{ difficultyDescription }}</p>
              </div>
              
              <button @click="startGame" class="start-btn">开始游戏</button>
            </div>
            
            <!-- 游戏进行中 -->
            <div v-else class="game-playing">
              <h5>🎯 游戏进行中</h5>
              
              <div class="game-stats">
                <div class="stat">
                  <span class="label">等级:</span>
                  <span class="value">{{ gameState.level }}</span>
                </div>
                <div class="stat">
                  <span class="label">分数:</span>
                  <span class="value">{{ gameState.score }}</span>
                </div>
                <div class="stat">
                  <span class="label">生命:</span>
                  <span class="value">
                    <span v-for="n in gameState.lives" :key="n" class="life">❤️</span>
                    <span v-for="n in (3 - gameState.lives)" :key="n" class="life-lost">💔</span>
                  </span>
                </div>
              </div>
              
              <!-- 基于游戏状态的提示 -->
              <div class="game-alerts">
                <div v-if="gameState.lives === 1" class="alert danger">
                  ⚠️ 危险！只剩最后一条生命！
                </div>
                <div v-else-if="gameState.lives === 2" class="alert warning">
                  💀 小心！生命值偏低！
                </div>
                
                <div v-if="gameState.level >= 5" class="alert success">
                  🌟 太棒了！你已经是高手了！
                </div>
                
                <div v-if="gameState.score >= 1000" class="alert info">
                  🎉 恭喜！突破1000分大关！
                </div>
              </div>
              
              <!-- 游戏模式特殊效果 -->
              <div v-if="gameState.gameMode === 'nightmare'" class="nightmare-mode">
                <p class="nightmare-text">💀 噩梦模式激活 💀</p>
                <div class="nightmare-effects">
                  <span class="effect">👻</span>
                  <span class="effect">⚡</span>
                  <span class="effect">🔥</span>
                </div>
              </div>
              
              <div v-else-if="gameState.gameMode === 'hard'" class="hard-mode">
                <p class="hard-text">⚡ 困难模式 - 挑战自我 ⚡</p>
              </div>
              
              <div class="game-controls">
                <button @click="levelUp" class="game-btn level-up">升级</button>
                <button @click="loseLife" class="game-btn lose-life">失去生命</button>
                <button @click="stopGame" class="game-btn stop">结束游戏</button>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 表单验证条件渲染 -->
    <div class="section">
      <h3>6. 表单验证条件渲染</h3>
      <div class="card">
        <div class="demo-group">
          <h4>动态表单验证</h4>
          
          <div class="validation-form">
            <div class="form-field">
              <label>用户名:</label>
              <input 
                v-model="formValidation.username.value"
                @input="validateUsername"
                :class="{ 
                  'valid': formValidation.username.isValid && formValidation.username.value,
                  'invalid': !formValidation.username.isValid 
                }"
                placeholder="输入用户名"
              >
              <div v-if="!formValidation.username.isValid" class="error-message">
                ❌ {{ formValidation.username.error }}
              </div>
              <div v-else-if="formValidation.username.value" class="success-message">
                ✅ 用户名格式正确
              </div>
            </div>
            
            <div class="form-field">
              <label>邮箱:</label>
              <input 
                v-model="formValidation.email.value"
                @input="validateEmail"
                :class="{ 
                  'valid': formValidation.email.isValid && formValidation.email.value,
                  'invalid': !formValidation.email.isValid 
                }"
                placeholder="输入邮箱地址"
                type="email"
              >
              <div v-if="!formValidation.email.isValid && formValidation.email.value" class="error-message">
                ❌ {{ formValidation.email.error }}
              </div>
              <div v-else-if="formValidation.email.isValid && formValidation.email.value" class="success-message">
                ✅ 邮箱格式正确
              </div>
            </div>
            
            <div class="form-field">
              <label>密码:</label>
              <input 
                v-model="formValidation.password.value"
                @input="validatePassword"
                :class="{ 
                  'valid': formValidation.password.isValid && formValidation.password.value,
                  'invalid': !formValidation.password.isValid 
                }"
                placeholder="输入密码"
                type="password"
              >
              <div v-if="!formValidation.password.isValid && formValidation.password.value" class="error-message">
                ❌ {{ formValidation.password.error }}
              </div>
              <div v-else-if="formValidation.password.isValid && formValidation.password.value" class="success-message">
                ✅ 密码强度合格
              </div>
            </div>
            
            <div class="form-field">
              <label>确认密码:</label>
              <input 
                v-model="formValidation.confirmPassword.value"
                @input="validateConfirmPassword"
                :class="{ 
                  'valid': formValidation.confirmPassword.isValid && formValidation.confirmPassword.value,
                  'invalid': !formValidation.confirmPassword.isValid 
                }"
                placeholder="再次输入密码"
                type="password"
              >
              <div v-if="!formValidation.confirmPassword.isValid && formValidation.confirmPassword.value" class="error-message">
                ❌ {{ formValidation.confirmPassword.error }}
              </div>
              <div v-else-if="formValidation.confirmPassword.isValid && formValidation.confirmPassword.value" class="success-message">
                ✅ 密码确认一致
              </div>
            </div>
            
            <!-- 整体表单状态 -->
            <div class="form-status">
              <div v-if="isFormValid" class="form-valid">
                ✅ 表单验证通过，可以提交
              </div>
              <div v-else class="form-invalid">
                ❌ 请完善表单信息
              </div>
              
              <button 
                :disabled="!isFormValid"
                :class="{ 'enabled': isFormValid, 'disabled': !isFormValid }"
                class="submit-btn"
              >
                {{ isFormValid ? '提交表单' : '请完善信息' }}
              </button>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- template 上使用 v-if -->
    <div class="section">
      <h3>7. template 标签条件渲染</h3>
      <div class="card">
        <div class="demo-group">
          <h4>不产生额外DOM元素的条件渲染</h4>
          
          <div class="template-demo">
            <h5>普通div条件渲染（会产生额外div）</h5>
            <div v-if="showWithIf" class="wrapper">
              <p>内容1</p>
              <p>内容2</p>
              <p>内容3</p>
            </div>
            
            <h5>template条件渲染（不会产生额外元素）</h5>
            <template v-if="showWithIf">
              <p>内容1 - 直接在父元素下</p>
              <p>内容2 - 没有包裹div</p>
              <p>内容3 - 保持结构清洁</p>
            </template>
            
            <p class="tip">💡 使用 template 可以避免不必要的DOM元素</p>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 总结 -->
    <div class="section summary">
      <h3>📝 条件渲染总结</h3>
      <div class="card">
        <div class="summary-grid">
          <div class="summary-item">
            <h4>🎯 选择指南</h4>
            <ul>
              <li><strong>v-if</strong>: 条件很少改变</li>
              <li><strong>v-show</strong>: 频繁切换显示</li>
              <li><strong>v-else-if</strong>: 多分支条件</li>
              <li><strong>template</strong>: 避免额外DOM</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>⚡ 性能考虑</h4>
            <ul>
              <li>v-if 有条件地渲染元素</li>
              <li>v-show 始终渲染但控制显示</li>
              <li>切换频率决定选择</li>
              <li>SEO需求影响决策</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>🔧 最佳实践</h4>
            <ul>
              <li>复杂条件用计算属性</li>
              <li>避免在v-for中使用v-if</li>
              <li>条件表达式保持简洁</li>
              <li>使用template减少DOM层级</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>📋 应用场景</h4>
            <ul>
              <li>权限控制</li>
              <li>状态管理</li>
              <li>表单验证</li>
              <li>响应式布局</li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.conditional-render {
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

/* 基础对比样式 */
.comparison {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin: 20px 0;
}

.compare-item {
  padding: 15px;
  border: 2px solid #ddd;
  border-radius: 8px;
}

.demo-box {
  padding: 15px;
  border-radius: 4px;
  margin: 10px 0;
  font-weight: bold;
}

.if-box {
  background: #e8f5e9;
  border: 2px solid #4caf50;
  color: #2e7d32;
}

.else-box {
  background: #ffebee;
  border: 2px solid #f44336;
  color: #c62828;
}

.show-box {
  background: #e3f2fd;
  border: 2px solid #2196f3;
  color: #1565c0;
}

.tip {
  color: #666;
  font-size: 14px;
  font-style: italic;
  margin-top: 10px;
}

.performance-note {
  background: #f5f5f5;
  padding: 15px;
  border-radius: 8px;
  margin-top: 20px;
}

.performance-note ul {
  margin: 10px 0;
  padding-left: 20px;
}

/* 多条件分支样式 */
.multi-condition {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 30px;
}

.role-card {
  padding: 20px;
  border-radius: 8px;
  margin: 10px 0;
  border: 2px solid;
}

.role-card.guest {
  background: #f5f5f5;
  border-color: #999;
  color: #333;
}

.role-card.user {
  background: #e3f2fd;
  border-color: #2196f3;
  color: #1565c0;
}

.role-card.admin {
  background: #ffeaa7;
  border-color: #fdcb6e;
  color: #d63031;
}

.role-card.vip {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-color: #667eea;
  color: white;
}

.role-card.unknown {
  background: #ffebee;
  border-color: #f44336;
  color: #c62828;
}

.permissions {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  margin-top: 10px;
}

.permission {
  background: rgba(255, 255, 255, 0.2);
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 12px;
}

/* 登录状态样式 */
.status-card {
  padding: 20px;
  border-radius: 8px;
  margin: 10px 0;
  text-align: center;
}

.status-card.logged-out {
  background: #ffebee;
  border: 2px solid #f44336;
  color: #c62828;
}

.status-card.logged-in {
  background: #e8f5e9;
  border: 2px solid #4caf50;
  color: #2e7d32;
}

.status-card.expired {
  background: #fff3e0;
  border: 2px solid #ff9800;
  color: #e65100;
}

.login-btn, .logout-btn, .relogin-btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-top: 10px;
}

.login-btn {
  background: #4caf50;
  color: white;
}

.logout-btn {
  background: #f44336;
  color: white;
}

.relogin-btn {
  background: #ff9800;
  color: white;
}

/* 控制按钮 */
.controls {
  display: flex;
  gap: 10px;
  margin-top: 15px;
  align-items: center;
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

.current-role, .current-status {
  background: #f0f0f0;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 14px;
  color: #333;
}

/* 天气系统样式 */
.weather-system {
  max-width: 600px;
}

.weather-params {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 15px;
  margin-bottom: 20px;
}

.param {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.param-controls {
  display: flex;
  gap: 5px;
}

.param-controls button {
  width: 30px;
  height: 30px;
  padding: 0;
  display: flex;
  align-items: center;
  justify-content: center;
}

.weather-card {
  padding: 20px;
  border-radius: 8px;
  margin: 10px 0;
  border: 2px solid;
}

.weather-card.rainy {
  background: #e8eaf6;
  border-color: #3f51b5;
  color: #1a237e;
}

.weather-card.hot {
  background: #ffebee;
  border-color: #f44336;
  color: #b71c1c;
}

.weather-card.cold {
  background: #e1f5fe;
  border-color: #03a9f4;
  color: #01579b;
}

.weather-card.humid {
  background: #f3e5f5;
  border-color: #9c27b0;
  color: #4a148c;
}

.weather-card.pleasant {
  background: #e8f5e9;
  border-color: #4caf50;
  color: #1b5e20;
}

.weather-detail {
  margin-top: 10px;
  padding: 8px;
  background: rgba(255, 255, 255, 0.3);
  border-radius: 4px;
  font-size: 14px;
}

/* 用户系统样式 */
.user-profile {
  padding: 20px;
  background: #f9f9f9;
  border-radius: 8px;
}

.subscription-card {
  padding: 15px;
  border-radius: 8px;
  margin: 10px 0;
  border: 2px solid;
}

.subscription-card.free {
  background: #f5f5f5;
  border-color: #999;
  color: #333;
}

.subscription-card.premium {
  background: linear-gradient(135deg, #ffd700, #ffed4e);
  border-color: #f39c12;
  color: #d68910;
}

.subscription-card.enterprise {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  border-color: #667eea;
  color: white;
}

.permissions-display {
  margin: 15px 0;
}

.permission-list {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
}

.permission-tag {
  background: #42b883;
  color: white;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 12px;
}

.no-permissions {
  color: #999;
  font-style: italic;
}

.actions {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
  margin-top: 15px;
}

.action-btn {
  padding: 8px 12px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.action-btn.read {
  background: #2196f3;
  color: white;
}

.action-btn.write {
  background: #4caf50;
  color: white;
}

.action-btn.delete {
  background: #f44336;
  color: white;
}

.action-btn.admin {
  background: #ff9800;
  color: white;
}

.login-prompt {
  padding: 20px;
  background: #ffebee;
  border-radius: 8px;
  text-align: center;
  color: #c62828;
}

/* 游戏系统样式 */
.game-system {
  max-width: 600px;
}

.game-start, .game-playing {
  padding: 20px;
  border-radius: 8px;
  background: #f5f5f5;
}

.game-mode-selection {
  margin: 15px 0;
}

.mode-btn {
  padding: 10px 20px;
  font-size: 16px;
  border-radius: 8px;
}

.mode-btn.normal {
  background: #4caf50;
}

.mode-btn.hard {
  background: #ff9800;
}

.mode-btn.nightmare {
  background: #f44336;
}

.difficulty-desc {
  color: #666;
  font-size: 14px;
  margin-top: 8px;
}

.start-btn {
  background: #4caf50;
  color: white;
  padding: 12px 24px;
  font-size: 18px;
  border-radius: 8px;
}

.game-stats {
  display: flex;
  gap: 20px;
  margin: 15px 0;
  flex-wrap: wrap;
}

.stat {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 10px;
  background: white;
  border-radius: 8px;
  min-width: 80px;
}

.stat .label {
  font-size: 12px;
  color: #666;
}

.stat .value {
  font-size: 18px;
  font-weight: bold;
  color: #333;
}

.life {
  font-size: 16px;
}

.life-lost {
  font-size: 16px;
  opacity: 0.3;
}

.game-alerts {
  margin: 15px 0;
}

.alert {
  padding: 10px;
  border-radius: 4px;
  margin: 5px 0;
  font-weight: bold;
}

.alert.danger {
  background: #ffebee;
  color: #c62828;
  border: 2px solid #f44336;
}

.alert.warning {
  background: #fff3e0;
  color: #e65100;
  border: 2px solid #ff9800;
}

.alert.success {
  background: #e8f5e9;
  color: #2e7d32;
  border: 2px solid #4caf50;
}

.alert.info {
  background: #e3f2fd;
  color: #1565c0;
  border: 2px solid #2196f3;
}

.nightmare-mode {
  text-align: center;
  background: linear-gradient(45deg, #000, #333);
  color: #ff6b6b;
  padding: 15px;
  border-radius: 8px;
  margin: 15px 0;
}

.nightmare-text {
  font-size: 18px;
  font-weight: bold;
  margin: 0;
  text-shadow: 0 0 10px #ff6b6b;
}

.nightmare-effects {
  display: flex;
  justify-content: center;
  gap: 10px;
  margin-top: 10px;
}

.effect {
  font-size: 24px;
  animation: pulse 1s infinite alternate;
}

@keyframes pulse {
  from { opacity: 0.5; transform: scale(0.8); }
  to { opacity: 1; transform: scale(1.2); }
}

.hard-mode {
  text-align: center;
  background: linear-gradient(45deg, #ff9800, #ffb74d);
  color: white;
  padding: 15px;
  border-radius: 8px;
  margin: 15px 0;
}

.hard-text {
  font-size: 16px;
  font-weight: bold;
  margin: 0;
}

.game-controls {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.game-btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.game-btn.level-up {
  background: #4caf50;
  color: white;
}

.game-btn.lose-life {
  background: #f44336;
  color: white;
}

.game-btn.stop {
  background: #999;
  color: white;
}

/* 表单验证样式 */
.validation-form {
  max-width: 500px;
}

.form-field {
  margin: 15px 0;
}

.form-field label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #333;
}

.form-field input {
  width: 100%;
  padding: 10px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.form-field input:focus {
  outline: none;
  border-color: #42b883;
}

.form-field input.valid {
  border-color: #4caf50;
}

.form-field input.invalid {
  border-color: #f44336;
}

.error-message {
  color: #f44336;
  font-size: 14px;
  margin-top: 5px;
}

.success-message {
  color: #4caf50;
  font-size: 14px;
  margin-top: 5px;
}

.form-status {
  margin-top: 20px;
  padding: 15px;
  border-radius: 8px;
}

.form-valid {
  background: #e8f5e9;
  color: #2e7d32;
  padding: 10px;
  border-radius: 4px;
  margin-bottom: 10px;
}

.form-invalid {
  background: #ffebee;
  color: #c62828;
  padding: 10px;
  border-radius: 4px;
  margin-bottom: 10px;
}

.submit-btn {
  width: 100%;
  padding: 12px;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s;
}

.submit-btn.enabled {
  background: #4caf50;
  color: white;
}

.submit-btn.disabled {
  background: #ccc;
  color: #999;
  cursor: not-allowed;
}

/* template 演示样式 */
.template-demo {
  max-width: 600px;
}

.wrapper {
  border: 2px dashed #42b883;
  padding: 10px;
  margin: 10px 0;
  background: #f0f8f0;
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

/* 响应式设计 */
@media (max-width: 768px) {
  .comparison {
    grid-template-columns: 1fr;
  }
  
  .multi-condition {
    grid-template-columns: 1fr;
  }
  
  .weather-params {
    grid-template-columns: 1fr;
  }
  
  .game-stats {
    justify-content: center;
  }
  
  .summary-grid {
    grid-template-columns: 1fr;
  }
  
  .controls {
    flex-direction: column;
    align-items: stretch;
  }
  
  .actions {
    flex-direction: column;
  }
}
</style>