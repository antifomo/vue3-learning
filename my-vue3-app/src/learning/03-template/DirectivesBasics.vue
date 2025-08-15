<script setup>
// Day 3 - Vue3 指令基础大全
import { ref, reactive, computed } from 'vue'

// ==================== 基础数据 ====================

// 文本相关
const message = ref('Hello Vue3!')
const htmlContent = ref('<span style="color: red; font-weight: bold;">这是HTML内容</span>')
const dynamicText = ref('动态文本')

// 显示控制
const isVisible = ref(true)
const isDisplay = ref(true)
const showDetails = ref(false)

// 样式和类
const textColor = ref('blue')
const fontSize = ref(16)
const isActive = ref(false)
const isError = ref(false)
const dynamicClass = ref('highlight')

// 表单数据
const inputValue = ref('')
const textareaValue = ref('')
const selectValue = ref('option1')
const checkboxValue = ref(false)
const radioValue = ref('male')

// 列表数据
const items = reactive([
  { id: 1, name: '苹果', category: '水果', price: 5 },
  { id: 2, name: '香蕉', category: '水果', price: 3 },
  { id: 3, name: '胡萝卜', category: '蔬菜', price: 2 },
  { id: 4, name: '白菜', category: '蔬菜', price: 1 }
])

// 计数器
const counter = ref(0)

// ==================== 方法 ====================

// 切换显示状态
function toggleVisibility() {
  isVisible.value = !isVisible.value
}

function toggleDisplay() {
  isDisplay.value = !isDisplay.value
}

function toggleDetails() {
  showDetails.value = !showDetails.value
}

// 切换样式
function toggleActive() {
  isActive.value = !isActive.value
}

function toggleError() {
  isError.value = !isError.value
}

function changeDynamicClass() {
  const classes = ['highlight', 'warning', 'success', 'info']
  const currentIndex = classes.indexOf(dynamicClass.value)
  const nextIndex = (currentIndex + 1) % classes.length
  dynamicClass.value = classes[nextIndex]
}

// 颜色和大小控制
function changeColor() {
  const colors = ['red', 'green', 'blue', 'purple', 'orange']
  const currentIndex = colors.indexOf(textColor.value)
  const nextIndex = (currentIndex + 1) % colors.length
  textColor.value = colors[nextIndex]
}

function increaseFontSize() {
  fontSize.value += 2
}

function decreaseFontSize() {
  if (fontSize.value > 12) {
    fontSize.value -= 2
  }
}

// 计数器操作
function increment() {
  counter.value++
}

function decrement() {
  counter.value--
}

function reset() {
  counter.value = 0
}

// 事件处理示例
function handleClick() {
  alert('按钮被点击了！')
}

function handleMouseEnter() {
  console.log('鼠标进入了元素')
}

function handleKeyUp(event) {
  console.log('按键松开:', event.key)
}

// 添加列表项
function addItem() {
  const newItem = {
    id: Date.now(),
    name: `新商品${items.length + 1}`,
    category: '其他',
    price: Math.floor(Math.random() * 10) + 1
  }
  items.push(newItem)
}

function removeItem(id) {
  const index = items.findIndex(item => item.id === id)
  if (index > -1) {
    items.splice(index, 1)
  }
}

// 清空表单
function clearForm() {
  inputValue.value = ''
  textareaValue.value = ''
  selectValue.value = 'option1'
  checkboxValue.value = false
  radioValue.value = 'male'
}

// 计算属性
const dynamicStyle = computed(() => {
  return {
    color: textColor.value,
    fontSize: fontSize.value + 'px',
    fontWeight: isActive.value ? 'bold' : 'normal',
    textDecoration: isError.value ? 'line-through' : 'none'
  }
})

const classList = computed(() => {
  return {
    active: isActive.value,
    error: isError.value,
    [dynamicClass.value]: true
  }
})
</script>

<template>
  <div class="directives-basics">
    <h2>📚 Vue3 指令基础大全</h2>
    
    <!-- v-text 和 v-html -->
    <div class="section">
      <h3>1. 文本渲染指令</h3>
      <div class="card">
        <div class="demo-group">
          <h4>v-text - 纯文本渲染</h4>
          <p v-text="message"></p>
          <p>等同于：{{ message }}</p>
          
          <h4>v-html - HTML 渲染</h4>
          <div v-html="htmlContent"></div>
          <p class="warning">⚠️ 注意：v-html 可能导致XSS攻击，只在可信内容上使用</p>
          
          <h4>插值表达式</h4>
          <p>{{ dynamicText }} - 当前时间：{{ new Date().toLocaleTimeString() }}</p>
          
          <div class="controls">
            <input v-model="dynamicText" placeholder="修改文本">
            <button @click="dynamicText = '重置文本'">重置</button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- v-show 和 v-if -->
    <div class="section">
      <h3>2. 条件渲染指令</h3>
      <div class="card">
        <div class="demo-group">
          <h4>v-show - 控制显示（CSS display）</h4>
          <p v-show="isVisible" class="demo-box show-demo">
            v-show 控制的内容 - 始终存在DOM中
          </p>
          
          <h4>v-if - 条件渲染（添加/移除DOM）</h4>
          <p v-if="isDisplay" class="demo-box if-demo">
            v-if 控制的内容 - 条件为真时才存在DOM中
          </p>
          
          <h4>详细信息展开</h4>
          <button @click="toggleDetails">
            {{ showDetails ? '隐藏' : '显示' }}详细信息
          </button>
          <div v-if="showDetails" class="details">
            <h5>详细信息：</h5>
            <ul>
              <li>v-show 适合频繁切换</li>
              <li>v-if 适合条件很少改变的情况</li>
              <li>v-if 有更高的切换开销</li>
              <li>v-show 有更高的初始渲染开销</li>
            </ul>
          </div>
          
          <div class="controls">
            <button @click="toggleVisibility">
              切换 v-show ({{ isVisible ? '显示' : '隐藏' }})
            </button>
            <button @click="toggleDisplay">
              切换 v-if ({{ isDisplay ? '显示' : '隐藏' }})
            </button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- v-for -->
    <div class="section">
      <h3>3. 列表渲染指令</h3>
      <div class="card">
        <div class="demo-group">
          <h4>v-for - 渲染列表</h4>
          
          <h5>数组遍历</h5>
          <ul class="items-list">
            <li v-for="item in items" :key="item.id" class="item">
              <span class="item-name">{{ item.name }}</span>
              <span class="item-category">{{ item.category }}</span>
              <span class="item-price">¥{{ item.price }}</span>
              <button @click="removeItem(item.id)" class="remove-btn">删除</button>
            </li>
          </ul>
          
          <h5>带索引的遍历</h5>
          <ol>
            <li v-for="(item, index) in items" :key="item.id">
              第{{ index + 1 }}项：{{ item.name }}
            </li>
          </ol>
          
          <h5>数字遍历</h5>
          <div class="number-list">
            <span v-for="n in 10" :key="n" class="number">{{ n }}</span>
          </div>
          
          <div class="controls">
            <button @click="addItem">添加商品</button>
            <span class="count">总计：{{ items.length }} 项</span>
          </div>
        </div>
      </div>
    </div>
    
    <!-- v-bind -->
    <div class="section">
      <h3>4. 属性绑定指令</h3>
      <div class="card">
        <div class="demo-group">
          <h4>v-bind - 动态属性绑定</h4>
          
          <h5>样式绑定</h5>
          <p :style="dynamicStyle">
            动态样式文本 - 颜色：{{ textColor }}，大小：{{ fontSize }}px
          </p>
          
          <h5>类绑定</h5>
          <p :class="classList">
            动态类名文本 - 当前状态：
            {{ isActive ? 'active' : '' }} 
            {{ isError ? 'error' : '' }}
            {{ dynamicClass }}
          </p>
          
          <h5>属性绑定</h5>
          <input 
            :placeholder="'输入框 - 计数器值：' + counter"
            :disabled="counter > 10"
            :title="'当前计数：' + counter"
            class="demo-input"
          >
          
          <div class="controls">
            <button @click="changeColor">切换颜色</button>
            <button @click="increaseFontSize">字体+</button>
            <button @click="decreaseFontSize">字体-</button>
            <button @click="toggleActive">切换活跃</button>
            <button @click="toggleError">切换错误</button>
            <button @click="changeDynamicClass">切换类名</button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- v-on -->
    <div class="section">
      <h3>5. 事件监听指令</h3>
      <div class="card">
        <div class="demo-group">
          <h4>v-on - 事件处理</h4>
          
          <h5>基础事件</h5>
          <div class="event-demo">
            <button @click="handleClick">点击事件</button>
            <button @click="increment">计数 +1</button>
            <button @click="decrement">计数 -1</button>
            <button @click="reset">重置</button>
            <span class="counter-display">当前计数：{{ counter }}</span>
          </div>
          
          <h5>鼠标事件</h5>
          <div 
            class="mouse-area"
            @mouseenter="handleMouseEnter"
            @mouseleave="() => console.log('鼠标离开了元素')"
            @click="() => console.log('区域被点击')"
          >
            鼠标悬停区域 - 打开控制台查看事件
          </div>
          
          <h5>键盘事件</h5>
          <input 
            v-model="inputValue"
            @keyup="handleKeyUp"
            @keyup.enter="() => console.log('回车键被按下')"
            placeholder="输入内容，按回车试试"
            class="demo-input"
          >
          
          <p>当前输入：{{ inputValue }}</p>
        </div>
      </div>
    </div>
    
    <!-- v-model -->
    <div class="section">
      <h3>6. 双向绑定指令</h3>
      <div class="card">
        <div class="demo-group">
          <h4>v-model - 表单双向绑定</h4>
          
          <div class="form-demo">
            <div class="form-group">
              <label>文本输入：</label>
              <input v-model="inputValue" placeholder="输入文本">
              <span class="value-display">值：{{ inputValue }}</span>
            </div>
            
            <div class="form-group">
              <label>多行文本：</label>
              <textarea v-model="textareaValue" placeholder="输入多行文本"></textarea>
              <span class="value-display">值：{{ textareaValue }}</span>
            </div>
            
            <div class="form-group">
              <label>下拉选择：</label>
              <select v-model="selectValue">
                <option value="option1">选项1</option>
                <option value="option2">选项2</option>
                <option value="option3">选项3</option>
              </select>
              <span class="value-display">值：{{ selectValue }}</span>
            </div>
            
            <div class="form-group">
              <label>复选框：</label>
              <input type="checkbox" v-model="checkboxValue" id="checkbox">
              <label for="checkbox">同意条款</label>
              <span class="value-display">值：{{ checkboxValue }}</span>
            </div>
            
            <div class="form-group">
              <label>单选按钮：</label>
              <input type="radio" v-model="radioValue" value="male" id="male">
              <label for="male">男性</label>
              <input type="radio" v-model="radioValue" value="female" id="female">
              <label for="female">女性</label>
              <span class="value-display">值：{{ radioValue }}</span>
            </div>
            
            <button @click="clearForm" class="clear-btn">清空表单</button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- v-pre 和 v-once -->
    <div class="section">
      <h3>7. 特殊指令</h3>
      <div class="card">
        <div class="demo-group">
          <h4>v-pre - 跳过编译</h4>
          <p v-pre>{{ message }} - 这里的插值不会被编译</p>
          <p>对比：{{ message }} - 这里会被编译</p>
          
          <h4>v-once - 只渲染一次</h4>
          <p v-once>只渲染一次：{{ counter }}</p>
          <p>实时更新：{{ counter }}</p>
          <button @click="increment">增加计数器观察差异</button>
        </div>
      </div>
    </div>
    
    <!-- 总结 -->
    <div class="section summary">
      <h3>📝 指令总结</h3>
      <div class="card">
        <div class="summary-grid">
          <div class="summary-item">
            <h4>文本渲染</h4>
            <ul>
              <li><code>v-text</code> - 纯文本</li>
              <li><code>v-html</code> - HTML内容</li>
              <li><code>{{}}</code> - 插值表达式</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>条件渲染</h4>
            <ul>
              <li><code>v-if</code> - 条件渲染</li>
              <li><code>v-show</code> - 显示控制</li>
              <li><code>v-else</code> - 否则分支</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>列表渲染</h4>
            <ul>
              <li><code>v-for</code> - 循环渲染</li>
              <li><code>:key</code> - 唯一标识</li>
              <li>支持数组、对象、数字</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>属性绑定</h4>
            <ul>
              <li><code>v-bind / :</code> - 属性绑定</li>
              <li><code>:class</code> - 类绑定</li>
              <li><code>:style</code> - 样式绑定</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>事件处理</h4>
            <ul>
              <li><code>v-on / @</code> - 事件监听</li>
              <li>支持修饰符</li>
              <li>内联处理器</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>双向绑定</h4>
            <ul>
              <li><code>v-model</code> - 表单绑定</li>
              <li>支持修饰符</li>
              <li>自动类型转换</li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.directives-basics {
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

.demo-group {
  margin-bottom: 20px;
}

.demo-group h4 {
  color: #333;
  margin-bottom: 15px;
}

.demo-group h5 {
  color: #666;
  margin: 15px 0 10px 0;
  font-size: 16px;
}

/* 基础样式 */
.demo-box {
  padding: 10px;
  border-radius: 4px;
  margin: 10px 0;
}

.show-demo {
  background: #e8f5e9;
  border: 1px solid #4caf50;
}

.if-demo {
  background: #e3f2fd;
  border: 1px solid #2196f3;
}

.details {
  background: #f5f5f5;
  padding: 15px;
  border-radius: 4px;
  margin-top: 10px;
}

.warning {
  color: #ff6b6b;
  font-size: 14px;
  margin: 10px 0;
}

/* 控制按钮 */
.controls {
  display: flex;
  gap: 10px;
  margin-top: 15px;
  flex-wrap: wrap;
  align-items: center;
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

.remove-btn, .clear-btn {
  background: #ff6b6b;
}

.remove-btn:hover, .clear-btn:hover {
  background: #ff5252;
}

/* 列表样式 */
.items-list {
  list-style: none;
  padding: 0;
}

.item {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 10px;
  background: #f5f5f5;
  margin-bottom: 5px;
  border-radius: 4px;
}

.item-name {
  font-weight: bold;
  min-width: 80px;
}

.item-category {
  color: #666;
  min-width: 60px;
}

.item-price {
  color: #42b883;
  font-weight: bold;
  min-width: 50px;
}

.number-list {
  display: flex;
  gap: 10px;
  flex-wrap: wrap;
}

.number {
  display: inline-block;
  width: 30px;
  height: 30px;
  line-height: 30px;
  text-align: center;
  background: #42b883;
  color: white;
  border-radius: 50%;
}

.count {
  color: #666;
  font-size: 14px;
}

/* 动态样式类 */
.active {
  background: #e8f5e9;
  border: 2px solid #4caf50;
}

.error {
  background: #ffebee;
  border: 2px solid #f44336;
}

.highlight {
  background: linear-gradient(45deg, #ffd700, #ffed4e);
}

.warning {
  background: linear-gradient(45deg, #ff9800, #ffb74d);
}

.success {
  background: linear-gradient(45deg, #4caf50, #81c784);
}

.info {
  background: linear-gradient(45deg, #2196f3, #64b5f6);
}

/* 输入框样式 */
.demo-input {
  padding: 8px 12px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  width: 200px;
}

.demo-input:focus {
  outline: none;
  border-color: #42b883;
}

.demo-input:disabled {
  background: #f5f5f5;
  color: #999;
}

/* 事件演示区域 */
.event-demo {
  display: flex;
  gap: 10px;
  align-items: center;
  flex-wrap: wrap;
  margin: 10px 0;
}

.counter-display {
  background: #e3f2fd;
  padding: 8px 12px;
  border-radius: 4px;
  font-weight: bold;
  color: #1976d2;
}

.mouse-area {
  background: #f5f5f5;
  border: 2px dashed #ddd;
  padding: 20px;
  text-align: center;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s;
  margin: 10px 0;
}

.mouse-area:hover {
  background: #e8f5e9;
  border-color: #4caf50;
}

/* 表单样式 */
.form-demo {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.form-group {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}

.form-group label {
  min-width: 80px;
  font-weight: bold;
  color: #333;
}

.form-group input, 
.form-group textarea, 
.form-group select {
  padding: 8px 12px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.form-group input:focus, 
.form-group textarea:focus, 
.form-group select:focus {
  outline: none;
  border-color: #42b883;
}

.form-group textarea {
  min-height: 80px;
  resize: vertical;
}

.value-display {
  background: #f0f0f0;
  padding: 4px 8px;
  border-radius: 3px;
  font-family: monospace;
  font-size: 14px;
  color: #666;
}

/* 总结网格 */
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
  .controls {
    flex-direction: column;
    align-items: stretch;
  }
  
  .controls button {
    width: 100%;
  }
  
  .event-demo {
    flex-direction: column;
    align-items: stretch;
  }
  
  .item {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .summary-grid {
    grid-template-columns: 1fr;
  }
  
  .form-group {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .form-group label {
    min-width: auto;
  }
}
</style>