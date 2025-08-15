<script setup>
// Day 1 - 基础练习组件
import HelloWorld from '@/learning/01-basics/HelloWorld.vue'
import UserCard from '@/learning/01-basics/UserCard.vue'

// Day 2 - 响应式进阶组件
import ReactiveBasics from '@/learning/02-reactive/ReactiveBasics.vue'
import ComputedExample from '@/learning/02-reactive/ComputedExample.vue'
import WatchExample from '@/learning/02-reactive/WatchExample.vue'
import TodoListAdvanced from '@/learning/02-reactive/TodoListAdvanced.vue'

// Day 2 - 练习组件
import ShoppingCart from '@/learning/02-reactive/exercises/ShoppingCart.vue'
import RealtimeSearch from '@/learning/02-reactive/exercises/RealtimeSearch.vue'
import FormValidation from '@/learning/02-reactive/exercises/FormValidation.vue'

// Day 3 - 模板语法组件（循序渐进学习）
import Step1VIf from '@/learning/03-template/Step1-VIf.vue'
import Step2VShow from '@/learning/03-template/Step2-VShow.vue'
import DirectivesBasics from '@/learning/03-template/DirectivesBasics.vue'
import ConditionalRender from '@/learning/03-template/ConditionalRender.vue'
import ListRendering from '@/learning/03-template/ListRendering.vue'
import EventHandling from '@/learning/03-template/EventHandling.vue'
import FormBinding from '@/learning/03-template/FormBinding.vue'

// Day 3 - 练习组件
import DynamicForm from '@/learning/03-template/exercises/DynamicForm.vue'
import FilterableTable from '@/learning/03-template/exercises/FilterableTable.vue'
import InteractiveGame from '@/learning/03-template/exercises/InteractiveGame.vue'

import { ref, computed } from 'vue'

// 控制显示哪个组件
const currentDay = ref('day1')
const currentView = ref('hello')

// 组件映射
const components = {
  day1: {
    hello: { component: HelloWorld, title: 'Hello World', icon: '👋' },
    user: { component: UserCard, title: '用户卡片', icon: '👤' }
  },
  day2: {
    reactive: { component: ReactiveBasics, title: 'Reactive 基础', icon: '⚡' },
    computed: { component: ComputedExample, title: 'Computed 计算属性', icon: '🧮' },
    watch: { component: WatchExample, title: 'Watch 侦听器', icon: '👁️' },
    todo: { component: TodoListAdvanced, title: '综合实践：待办列表', icon: '📝' },
    cart: { component: ShoppingCart, title: '练习1：购物车系统', icon: '🛒' },
    search: { component: RealtimeSearch, title: '练习2：实时搜索', icon: '🔍' },
    form: { component: FormValidation, title: '练习3：表单验证', icon: '📋' }
  },
  day3: {
    step1: { component: Step1VIf, title: '第1步：v-if 基础', icon: '🚀' },
    step2: { component: Step2VShow, title: '第2步：v-show vs v-if', icon: '🔍' },
    directives: { component: DirectivesBasics, title: '指令基础大全', icon: '🎯' },
    conditional: { component: ConditionalRender, title: '条件渲染深入', icon: '🔀' },
    list: { component: ListRendering, title: '列表渲染进阶', icon: '📋' },
    events: { component: EventHandling, title: '事件处理详解', icon: '🎪' },
    binding: { component: FormBinding, title: '表单双向绑定', icon: '🔗' },
    dynamicForm: { component: DynamicForm, title: '练习1：动态表单构建器', icon: '📝' },
    table: { component: FilterableTable, title: '练习2：可过滤排序表格', icon: '📊' },
    games: { component: InteractiveGame, title: '练习3：互动游戏集合', icon: '🎮' }
  }
}

// 当前组件信息
const currentComponent = computed(() => {
  return components[currentDay.value]?.[currentView.value] || components.day1.hello
})

// 切换到 Day 1
function switchToDay1(view = 'hello') {
  currentDay.value = 'day1'
  currentView.value = view
}

// 切换到 Day 2
function switchToDay2(view = 'reactive') {
  currentDay.value = 'day2'
  currentView.value = view
}

// 切换到 Day 3
function switchToDay3(view = 'step1') {
  currentDay.value = 'day3'
  currentView.value = view
}
</script>

<template>
  <div id="app">
    <!-- 主导航 -->
    <div class="main-nav">
      <div class="day-selector">
        <button 
          @click="switchToDay1()"
          :class="{ active: currentDay === 'day1' }"
          class="day-btn"
        >
          📚 Day 1: Vue3 基础
        </button>
        <button 
          @click="switchToDay2()"
          :class="{ active: currentDay === 'day2' }"
          class="day-btn"
        >
          🚀 Day 2: 响应式进阶
        </button>
        <button 
          @click="switchToDay3()"
          :class="{ active: currentDay === 'day3' }"
          class="day-btn"
        >
          🎨 Day 3: 模板语法
        </button>
      </div>
    </div>
    
    <!-- 子导航 -->
    <div class="sub-nav">
      <!-- Day 1 导航 -->
      <div v-if="currentDay === 'day1'" class="nav-buttons">
        <button 
          @click="currentView = 'hello'" 
          :class="{ active: currentView === 'hello' }"
        >
          👋 Hello World
        </button>
        <button 
          @click="currentView = 'user'" 
          :class="{ active: currentView === 'user' }"
        >
          👤 用户卡片
        </button>
      </div>
      
      <!-- Day 2 导航 -->
      <div v-else-if="currentDay === 'day2'" class="nav-buttons">
        <button 
          @click="currentView = 'reactive'" 
          :class="{ active: currentView === 'reactive' }"
        >
          ⚡ Reactive
        </button>
        <button 
          @click="currentView = 'computed'" 
          :class="{ active: currentView === 'computed' }"
        >
          🧮 Computed
        </button>
        <button 
          @click="currentView = 'watch'" 
          :class="{ active: currentView === 'watch' }"
        >
          👁️ Watch
        </button>
        <button 
          @click="currentView = 'todo'" 
          :class="{ active: currentView === 'todo' }"
        >
          📝 待办列表
        </button>
        <button 
          @click="currentView = 'cart'" 
          :class="{ active: currentView === 'cart' }"
        >
          🛒 购物车
        </button>
        <button 
          @click="currentView = 'search'" 
          :class="{ active: currentView === 'search' }"
        >
          🔍 实时搜索
        </button>
        <button 
          @click="currentView = 'form'" 
          :class="{ active: currentView === 'form' }"
        >
          📋 表单验证
        </button>
      </div>
      
      <!-- Day 3 导航 -->
      <div v-else-if="currentDay === 'day3'" class="nav-buttons">
        <button 
          @click="currentView = 'step1'" 
          :class="{ active: currentView === 'step1' }"
        >
          🚀 第1步
        </button>
        <button 
          @click="currentView = 'step2'" 
          :class="{ active: currentView === 'step2' }"
        >
          🔍 第2步
        </button>
        <button 
          @click="currentView = 'directives'" 
          :class="{ active: currentView === 'directives' }"
        >
          🎯 指令基础
        </button>
        <button 
          @click="currentView = 'conditional'" 
          :class="{ active: currentView === 'conditional' }"
        >
          🔀 条件渲染
        </button>
        <button 
          @click="currentView = 'list'" 
          :class="{ active: currentView === 'list' }"
        >
          📋 列表渲染
        </button>
        <button 
          @click="currentView = 'events'" 
          :class="{ active: currentView === 'events' }"
        >
          🎪 事件处理
        </button>
        <button 
          @click="currentView = 'binding'" 
          :class="{ active: currentView === 'binding' }"
        >
          🔗 表单绑定
        </button>
        <button 
          @click="currentView = 'dynamicForm'" 
          :class="{ active: currentView === 'dynamicForm' }"
        >
          📝 动态表单
        </button>
        <button 
          @click="currentView = 'table'" 
          :class="{ active: currentView === 'table' }"
        >
          📊 数据表格
        </button>
        <button 
          @click="currentView = 'games'" 
          :class="{ active: currentView === 'games' }"
        >
          🎮 互动游戏
        </button>
      </div>
    </div>
    
    <!-- 标题显示 -->
    <div class="title-section">
      <h1>
        <span class="day-label">
          {{ currentDay === 'day1' ? 'Day 1' : 
             currentDay === 'day2' ? 'Day 2' : 'Day 3' }}
        </span>
        <span class="separator">|</span>
        <span class="component-title">
          {{ currentComponent.icon }} {{ currentComponent.title }}
        </span>
      </h1>
    </div>
    
    <!-- 组件切换显示 -->
    <transition name="fade" mode="out-in">
      <component 
        :is="currentComponent.component" 
        :key="`${currentDay}-${currentView}`"
      />
    </transition>
  </div>
</template>

<style>
/* 全局样式重置 */
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  margin: 0;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, Oxygen, Ubuntu, Cantarell, 'Open Sans', 'Helvetica Neue', sans-serif;
}

#app {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding-top: 160px;
  padding-bottom: 40px;
}

/* 主导航样式 */
.main-nav {
  position: fixed;
  top: 20px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 1000;
  background: white;
  padding: 10px;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.15);
}

.day-selector {
  display: flex;
  gap: 10px;
}

.day-btn {
  padding: 12px 24px;
  background: white;
  border: 2px solid #667eea;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
  font-size: 16px;
  font-weight: 600;
  color: #333;
}

.day-btn:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(102, 126, 234, 0.3);
}

.day-btn.active {
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  color: white;
  border-color: transparent;
}

/* 子导航样式 */
.sub-nav {
  position: fixed;
  top: 90px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 1001;
}

.nav-buttons {
  display: flex;
  gap: 10px;
  background: white;
  padding: 10px;
  border-radius: 12px;
  box-shadow: 0 4px 20px rgba(0, 0, 0, 0.1);
}

.nav-buttons button {
  padding: 12px 24px;
  background: white;
  border: 2px solid #42b883;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
  font-size: 16px;
  font-weight: 500;
  color: #333;
}

.nav-buttons button:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(66, 184, 131, 0.3);
}

/* 确保导航按钮不会被遮挡 */
.nav-buttons button {
  position: relative;
  z-index: 1;
}

.nav-buttons button.active {
  background: linear-gradient(135deg, #42b883 0%, #35a372 100%);
  color: white;
  border-color: transparent;
}

/* 标题样式 */
.title-section {
  text-align: center;
  margin: 20px 0 40px 0;
}

.title-section h1 {
  color: white;
  font-size: 32px;
  text-shadow: 0 2px 10px rgba(0, 0, 0, 0.2);
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 15px;
}

.day-label {
  background: rgba(255, 255, 255, 0.2);
  padding: 5px 15px;
  border-radius: 20px;
  font-size: 24px;
}

.separator {
  opacity: 0.5;
}

.component-title {
  font-size: 28px;
}

/* 过渡动画 */
.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.3s, transform 0.3s;
}

.fade-enter-from {
  opacity: 0;
  transform: translateY(20px);
}

.fade-leave-to {
  opacity: 0;
  transform: translateY(-20px);
}

/* 响应式设计 */
@media (max-width: 768px) {
  .main-nav,
  .sub-nav {
    width: 90%;
  }
  
  .day-selector {
    flex-direction: column;
  }
  
  .day-btn {
    width: 100%;
  }
  
  .nav-buttons {
    flex-wrap: wrap;
  }
  
  .nav-buttons button {
    flex: 1;
    min-width: 120px;
  }
  
  #app {
    padding-top: 200px;
  }
  
  .title-section h1 {
    font-size: 20px;
    padding: 0 20px;
    flex-direction: column;
    gap: 10px;
  }
  
  .day-label {
    font-size: 18px;
  }
  
  .component-title {
    font-size: 20px;
  }
}

@media (max-width: 480px) {
  .nav-buttons {
    flex-direction: column;
  }
  
  .nav-buttons button {
    width: 100%;
  }
}
</style>
