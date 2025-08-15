# Day 3 实践练习指南

## 🎯 练习总览

Day 3 包含三个综合性练习项目，每个项目都结合了多个 Vue3 模板语法概念，帮助你在实战中巩固学习成果。

## 📝 练习1：动态表单构建器 (DynamicForm.vue)

### 🎪 功能介绍
一个完整的可视化表单设计器，支持拖拽操作、实时预览、模板系统等高级功能。

### 🔧 核心技术点
- **拖拽功能**：HTML5 Drag & Drop API
- **动态组件**：component :is 的使用
- **数据序列化**：JSON 导入导出
- **响应式布局**：CSS Grid 和 Flexbox
- **状态管理**：复杂表单状态的处理

### 📚 学习重点

#### 1. 拖拽实现原理
```vue
<template>
  <!-- 字段库 -->
  <div class="field-library">
    <div 
      v-for="field in fieldTypes" 
      :key="field.type"
      draggable="true"
      @dragstart="handleDragStart($event, field)"
      class="field-item"
    >
      {{ field.label }}
    </div>
  </div>
  
  <!-- 表单设计区 -->
  <div 
    class="form-builder"
    @drop="handleDrop"
    @dragover.prevent
  >
    <!-- 动态表单字段 -->
  </div>
</template>
```

#### 2. 动态组件渲染
```vue
<component 
  :is="getFieldComponent(field.type)"
  :field="field"
  @update="updateField"
/>
```

#### 3. 表单验证集成
```javascript
// 验证规则定义
const validationRules = {
  required: (value) => !!value || '此字段为必填项',
  email: (value) => /.+@.+\..+/.test(value) || '请输入有效邮箱',
  minLength: (min) => (value) => value?.length >= min || `最少输入${min}个字符`
}
```

### 🎮 实践任务

#### 任务1：基础功能体验
1. 从字段库拖拽不同类型的字段到设计区
2. 修改字段属性（标签、占位符、验证规则）
3. 调整字段顺序
4. 预览生成的表单

#### 任务2：模板系统
1. 使用预设模板快速创建表单
2. 将自定义表单保存为模板
3. 导出表单配置为 JSON

#### 任务3：高级定制
1. 添加自定义验证规则
2. 实现条件显示逻辑
3. 创建多步骤表单

### 💡 扩展思考
- 如何实现字段的嵌套关系？
- 如何支持更多的字段类型？
- 如何优化大型表单的性能？

---

## 📊 练习2：可过滤排序表格 (FilterableTable.vue)

### 🎪 功能介绍
一个功能完整的企业级数据表格组件，支持多维度过滤、灵活排序、分页、数据导出等功能。

### 🔧 核心技术点
- **计算属性**：复杂数据过滤和排序逻辑
- **组件设计**：可复用的表格组件架构
- **性能优化**：大数据量的处理技巧
- **用户体验**：交互反馈和加载状态

### 📚 学习重点

#### 1. 复杂过滤逻辑
```javascript
const filteredData = computed(() => {
  let result = [...tableData.value]
  
  // 全局搜索
  if (globalSearch.value) {
    result = result.filter(row => 
      searchableColumns.some(col => 
        String(row[col]).toLowerCase().includes(globalSearch.value.toLowerCase())
      )
    )
  }
  
  // 列过滤
  Object.entries(columnFilters).forEach(([column, filter]) => {
    if (filter.value) {
      result = result.filter(row => 
        applyColumnFilter(row[column], filter)
      )
    }
  })
  
  return result
})
```

#### 2. 动态排序实现
```javascript
const sortedData = computed(() => {
  if (!sortColumn.value) return filteredData.value
  
  return [...filteredData.value].sort((a, b) => {
    const aVal = a[sortColumn.value]
    const bVal = b[sortColumn.value]
    
    // 根据数据类型处理排序
    if (typeof aVal === 'number') {
      return sortDirection.value === 'asc' ? aVal - bVal : bVal - aVal
    }
    
    return sortDirection.value === 'asc' 
      ? String(aVal).localeCompare(String(bVal))
      : String(bVal).localeCompare(String(aVal))
  })
})
```

#### 3. 分页组件
```vue
<template>
  <div class="pagination">
    <button 
      @click="goToPage(1)" 
      :disabled="currentPage === 1"
    >
      首页
    </button>
    
    <button 
      v-for="page in visiblePages" 
      :key="page"
      @click="goToPage(page)"
      :class="{ active: page === currentPage }"
    >
      {{ page }}
    </button>
    
    <button 
      @click="goToPage(totalPages)" 
      :disabled="currentPage === totalPages"
    >
      末页
    </button>
  </div>
</template>
```

### 🎮 实践任务

#### 任务1：基础表格操作
1. 尝试全局搜索功能
2. 使用列过滤器
3. 点击表头进行排序
4. 测试分页功能

#### 任务2：高级搜索
1. 使用高级搜索构建复杂查询条件
2. 组合多个搜索条件
3. 保存常用搜索条件

#### 任务3：数据管理
1. 选择单行或多行数据
2. 批量删除操作
3. 导出选中数据

#### 任务4：统计分析
1. 查看数据统计面板
2. 分析不同维度的数据分布
3. 理解统计图表的实现

### 💡 扩展思考
- 如何实现虚拟滚动优化性能？
- 如何支持动态列配置？
- 如何实现实时数据更新？

---

## 🎮 练习3：互动游戏集合 (InteractiveGame.vue)

### 🎪 功能介绍
一个包含8款不同类型游戏的集合，展示了Vue3在游戏开发中的应用，包括状态管理、动画效果、用户交互等。

### 🔧 核心技术点
- **游戏状态管理**：复杂状态的设计和管理
- **定时器控制**：setInterval 和 setTimeout 的使用
- **键盘事件**：游戏控制的实现
- **动画效果**：CSS 动画和过渡
- **本地存储**：游戏进度和成就的持久化

### 📚 学习重点

#### 1. 游戏状态设计
```javascript
const gameState = reactive({
  // 通用状态
  isPlaying: false,
  score: 0,
  level: 1,
  
  // 贪吃蛇特定状态
  snake: [{ x: 10, y: 7 }],
  food: { x: 5, y: 5 },
  direction: { x: 1, y: 0 },
  
  // 记忆游戏特定状态
  cards: [],
  flippedCards: [],
  matchedPairs: 0
})
```

#### 2. 游戏循环实现
```javascript
function startGameLoop() {
  gameInterval = setInterval(() => {
    updateGameState()
    checkCollisions()
    renderGame()
  }, gameSpeed)
}

function updateGameState() {
  // 更新游戏对象位置
  // 处理游戏逻辑
  // 检查游戏结束条件
}
```

#### 3. 键盘控制
```javascript
function handleKeyPress(event) {
  if (currentGame.value === 'snake') {
    switch (event.key) {
      case 'ArrowUp':
      case 'w':
        changeDirection({ x: 0, y: -1 })
        break
      case 'ArrowDown':
      case 's':
        changeDirection({ x: 0, y: 1 })
        break
      // ...
    }
  }
}

onMounted(() => {
  document.addEventListener('keydown', handleKeyPress)
})
```

#### 4. 成就系统
```javascript
const achievements = [
  {
    id: 'first_game',
    name: '初次体验',
    description: '完成第一个游戏',
    condition: () => gamesPlayed >= 1
  },
  {
    id: 'high_score',
    name: '高分达成',
    description: '单局得分超过1000',
    condition: () => maxScore >= 1000
  }
]

function checkAchievements() {
  achievements.forEach(achievement => {
    if (!unlockedAchievements.includes(achievement.id) && achievement.condition()) {
      unlockAchievement(achievement)
    }
  })
}
```

### 🎮 实践任务

#### 任务1：游戏体验
1. 尝试每个游戏，了解玩法
2. 观察游戏状态的变化
3. 体验不同的交互方式

#### 任务2：控制机制
1. 使用键盘控制贪吃蛇
2. 测试反应游戏的响应速度
3. 体验触摸和点击交互

#### 任务3：数据分析
1. 查看游戏统计数据
2. 解锁不同的成就
3. 比较不同游戏的得分

#### 任务4：个性化
1. 调整游戏难度
2. 查看游戏历史记录
3. 重置游戏数据

### 🎯 游戏详解

#### 🧠 记忆翻牌游戏
- **核心逻辑**：卡片状态管理
- **关键技术**：数组操作、定时器
- **学习重点**：状态同步、动画效果

#### 🎯 猜数字游戏
- **核心逻辑**：区间搜索算法
- **关键技术**：输入验证、提示系统
- **学习重点**：用户反馈、游戏平衡

#### ⚡ 反应测试
- **核心逻辑**：时间测量
- **关键技术**：事件监听、性能分析
- **学习重点**：精确计时、统计分析

#### ⌨️ 打字游戏
- **核心逻辑**：文本比较
- **关键技术**：输入处理、实时验证
- **学习重点**：字符串操作、准确率计算

#### 🐍 贪吃蛇
- **核心逻辑**：网格移动、碰撞检测
- **关键技术**：游戏循环、方向控制
- **学习重点**：游戏物理、状态机

#### ✂️ 石头剪刀布
- **核心逻辑**：规则判断、随机算法
- **关键技术**：策略实现、历史记录
- **学习重点**：游戏规则、AI 对手

#### 🧭 知识问答
- **核心逻辑**：题目管理、计分系统
- **关键技术**：数据结构、时间限制
- **学习重点**：内容管理、进度跟踪

#### 🎨 颜色挑战
- **核心逻辑**：视觉识别、干扰设计
- **关键技术**：颜色处理、连击系统
- **学习重点**：视觉设计、难度递增

### 💡 扩展思考
- 如何实现多人游戏？
- 如何添加音效和动画？
- 如何设计更复杂的关卡系统？

---

## 🚀 练习建议

### 学习顺序
1. **先理论后实践**：阅读相关概念文档
2. **循序渐进**：从简单功能开始尝试
3. **深入分析**：理解代码实现原理
4. **自主创新**：尝试添加新功能

### 调试技巧
1. **使用 Vue DevTools**：观察组件状态变化
2. **Console 调试**：输出关键变量值
3. **断点调试**：逐步跟踪代码执行
4. **错误处理**：添加适当的错误边界

### 性能优化
1. **合理使用计算属性**：避免重复计算
2. **优化事件监听**：及时清理事件监听器
3. **减少不必要的重渲染**：使用 v-memo 等优化指令
4. **懒加载**：大数据分批加载

### 代码质量
1. **命名规范**：使用有意义的变量名
2. **注释文档**：添加必要的代码注释
3. **函数拆分**：保持函数的单一职责
4. **错误处理**：添加边界情况处理

## 📋 学习检查清单

### 基础功能理解
- [ ] 理解拖拽功能的实现原理
- [ ] 掌握动态组件的使用方法
- [ ] 熟悉表格数据的处理流程
- [ ] 了解游戏状态管理的设计

### 进阶技能掌握
- [ ] 能够实现复杂的数据过滤逻辑
- [ ] 掌握性能优化的基本方法
- [ ] 理解事件处理的最佳实践
- [ ] 能够设计合理的组件架构

### 实战应用能力
- [ ] 能够独立完成类似功能开发
- [ ] 具备问题分析和解决能力
- [ ] 掌握调试和优化技巧
- [ ] 能够写出可维护的代码

## 🎓 总结

通过这三个练习项目，你将深入理解 Vue3 模板语法的实际应用，掌握复杂交互的实现技巧，并培养良好的编程习惯。

记住：
- **多思考**：为什么这样设计？有什么优缺点？
- **多实践**：尝试修改代码，观察结果变化
- **多总结**：记录学习过程中的收获和问题
- **多创新**：基于现有功能开发新特性

祝你学习愉快！🎉