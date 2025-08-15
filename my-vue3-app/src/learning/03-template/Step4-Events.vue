<template>
  <div class="container">
    <h2>🎪 第4步：事件处理进阶</h2>
    
    <!-- 在这里练习各种事件处理 -->
     <!-- 1. 点击事件计数器 -->
     <div class="event-demo">
        <h3>🖱️ 点击事件 </h3>
        <button @click="clickCount++" class="btn-primary">
          点击我！已点击{{ clickCount }}次
        </button>
        <button @click="clickCount = 0" class="btn-warning">
          重置计数
        </button>
     </div> 
      <!-- 2. 鼠标悬停事件 -->
       <div class="event-demo">
          <h3>👆 鼠标悬停事件</h3>
          <div @mouseenter="message= '🎉 鼠标进入了！'"
               @mouseleave="message= '👋 鼠标离开了...'"
               style="padding: 20px; background: #e3f2fd; border: 2px dashed 
  #2196f3; text-align: center; cursor: pointer;"
               >
               悬停在我身上试试！
          </div>
          <div class="message-box">{{ message }}</div>
       </div>
       <!-- 3. 鼠标移动追踪 -->
        <div class="event-demo">
          <h3>🖱️ 鼠标位置追踪 </h3>
          <div @mousemove="trackMouse"  style="height: 120px; background: #f5f5f5; border: 1px solid #ccc;      
  position: relative; cursor: crosshair;"
    >
            <p style="margin: 10px;">在这个区域移动鼠标</p>
            <p style="margin: 10px;">鼠标位置：X: {{ mousePosition.x }}, Y: {{      
  mousePosition.y }}</p>
          </div>
        </div>
        <!-- 4. 键盘事件 -->
      <div class="event-demo">
        <h3>⌨️ 键盘事件</h3>
        <input 
          @keydown="handleKeyDown"
          @keyup="handleKeyUp"
          placeholder="在这里按任意键试试..."
          style="width: 100%; padding: 10px; border: 1px solid #ccc; 
      border-radius: 4px; margin-bottom: 10px;"
        >
        <div class="message-box">
          <p v-if="keyPressed">你刚才按下了：<strong>{{ keyPressed
      }}</strong></p>
          <p v-else>还没有按键...</p>
        </div>
      </div>

      <!-- 5. 输入事件（实时同步） -->
      <div class="event-demo">
        <h3>📝 输入事件（实时同步）</h3>
        <input 
          v-model="inputText"
          placeholder="输入一些文字..."
          style="width: 100%; padding: 10px; border: 1px solid #ccc; 
      border-radius: 4px; margin-bottom: 10px;"
        >
        <div class="message-box">
          <p>你正在输入：{{ inputText }}</p>
          <p>字符长度：{{ inputText.length }}</p>
        </div>
      </div>

      <!-- 6. 特殊按键检测 -->
      <div class="event-demo">
        <h3>🔍 特殊按键检测</h3>
        <input 
          @keyup.enter="handleEnter"
          @keyup.esc="handleEscape"
          @keyup.space="handleSpace"
          placeholder="试试按 Enter、Esc 或空格键..."
          style="width: 100%; padding: 10px; border: 1px solid #ccc; 
      border-radius: 4px; margin-bottom: 10px;"
        >
        <div class="message-box" id="special-key-message">
          按下特殊按键后，这里会显示消息
        </div>
      </div>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// 在这里定义响应式数据
const message = ref('事件处理') //显示消息
const mousePosition = ref({x:0,y:0}) //记录鼠标位置
const keyPressed = ref('') //记录按下的按键
const inputText = ref('') //输入框内容
const clickCount = ref(0) //点击次数计数器

function trackMouse(event){
  mousePosition.value.x = event.offsetX
  mousePosition.value.y = event.offsetY
}
function handleKeyDown(event) {
    keyPressed.value = `${event.key} (按下)`
  }

  function handleKeyUp(event) {
    keyPressed.value = `${event.key} (释放)`
  }

  function handleEnter() {
    message.value = '🎯 你按下了 Enter 键！'
  }

  function handleEscape() {
    message.value = '🚪 你按下了 Esc 键！'
  }

  function handleSpace() {
    message.value = '🚀 你按下了空格键！'
  }

</script>

<style scoped>
.container {
  max-width: 700px;
  margin: 0 auto;
  padding: 20px;
  background: white;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

h2 {
  color: #333;
  border-bottom: 2px solid #42b883;
  padding-bottom: 10px;
}

.event-demo {
  margin: 20px 0;
  padding: 15px;
  border: 2px solid #f0f0f0;
  border-radius: 8px;
}

.event-demo h3 {
  margin-top: 0;
  color: #666;
}

button {
  padding: 10px 15px;
  margin: 5px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.btn-primary {
  background: #42b883;
  color: white;
}

.btn-warning {
  background: #ffc107;
  color: #333;
}

.btn-danger {
  background: #e74c3c;
  color: white;
}

.message-box {
  margin: 10px 0;
  padding: 10px;
  background: #f8f9fa;
  border-left: 4px solid #42b883;
  border-radius: 4px;
}
</style>