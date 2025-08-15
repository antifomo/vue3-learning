<template>
  <div class="container">
    <h2>🎯 第1步：学习 v-if 基础</h2>
    
    <!-- 在这里练习 v-if 指令 -->
    <button @click="isVisible = !isVisible">
        {{ isVisible? '隐藏':'显示' }}
    </button>
    <p v-if="isVisible" style="margin-top: 20px; padding: 15px; background:
  #f0f8ff; border-left: 4px solid #42b883;">
        🎉 这段文字受 v-if 控制！<br />
        当 isVisible=true 时才会出现在DOM中。
    </p>
  </div>
</template>

<script setup>
import { ref } from 'vue'

// 在这里定义响应式数据
const isVisible = ref(false)

</script>

<style scoped>
.container {
  max-width: 600px;
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
</style>