<template>
  <div class="container">
    <h2>📋 第3步：v-for 列表渲染基础</h2>
    
    <!-- 在这里练习 v-for 指令 -->
    <h3>🍎 我的水果清单：</h3>
    <div v-for="fruit in fruits" :key="fruit" class="fruit-item">
      {{ fruit }}
    </div>
    <hr style="margin:20px 0;">
    <h3>🎮 试试添加新水果：</h3>
    <button @click="addFruit"style="padding: 10px 20px; margin: 10px; 
  background: #42b883; color: white; border: none; border-radius: 4px;">      
    ➕ 添加草莓
    </button>
    <button @click="removeFruit" style="padding: 10px 20px; margin: 10px;       
  background: #e74c3c; color: white; border: none; border-radius: 4px;">
    ➖ 移除最后一个
</button>


  </div>
</template>

<script setup>
import { ref } from 'vue'

// 在这里定义数据
    const fruits = ref([
    '🍎 苹果',
    '🍌 香蕉',
    '🍊 橙子',
    '🍇 葡萄',
    '🥝 猕猴桃'
  ])
  function addFruit(){
    fruits.value.push('🍓 草莓')
  }
  function removeFruit(){
    if(fruits.value.length >0){
      fruits.value.pop()
    }
  }
  /**
     相关的数组方法：
  // 添加和删除方法
  fruits.value.push('🍓')    // 在末尾添加
  fruits.value.pop()         // 删除最后一个
  fruits.value.unshift('🥭' ) // 在开头添加  
  fruits.value.shift()       // 删除第一个
   */
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

.fruit-item {
  padding: 10px 15px;
  margin: 5px 0;
  background: #f8f9fa;
  border-left: 4px solid #42b883;
  border-radius: 4px;
}
</style>