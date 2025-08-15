<template>
  <div class="container">
    <h2>🔍 第2步：v-show vs v-if 的区别</h2>
    
    <!-- 在这里对比 v-show 和 v-if -->
     <button @click="isVisible = !isVisible"  style="padding: 10px 20px; font-size:         
  16px; margin-bottom: 20px;">
      {{ isVisible ? '隐藏':'显示' }} 内容
     </button>
     <div class="demo-section">
      <h3>🎯使用v-if(条件渲染)</h3>
      <p v-if="isVisible" style="padding: 15px; background: #e8f5e8; border-left: 4px     
   solid #42b883;">
      我是 v-if 控制的内容！<br />当条件为 false 时，<br />我会完全从 DOM 中消失。
    </p>
     </div>

     <div class="demo-section">
      <h3>👁️ 使用 v-show（显示切换） </h3>
      <p v-show="isVisible" style="padding: 15px; background: #fff3cd; border-left:       
  4px solid #ffc107;">
      我是 v-show 控制的内容！<br />当条件为 false 时，我只是被 CSS 隐藏，<br />但还在 DOM 中。     
    </p>
     </div>
    
  </div>
</template>

<script setup>
import { ref } from 'vue'

// 在这里定义响应式数据
const isVisible = ref(true)

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

.demo-section {
  margin: 20px 0;
  padding: 15px;
  border: 2px solid #f0f0f0;
  border-radius: 8px;
}

.demo-section h3 {
  margin-top: 0;
  color: #666;
}
</style>