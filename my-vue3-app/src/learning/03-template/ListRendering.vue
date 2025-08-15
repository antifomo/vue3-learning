<script setup>
// Day 3 - 列表渲染进阶学习
import { ref, reactive, computed, onMounted } from 'vue'

// ==================== 基础数据 ====================

// 基础数组
const fruits = reactive(['苹果', '香蕉', '橙子', '葡萄', '草莓'])

// 对象数组
const users = reactive([
  { id: 1, name: '张三', age: 25, email: 'zhangsan@example.com', avatar: '👨', status: 'active' },
  { id: 2, name: '李四', age: 30, email: 'lisi@example.com', avatar: '👩', status: 'inactive' },
  { id: 3, name: '王五', age: 28, email: 'wangwu@example.com', avatar: '👨', status: 'active' },
  { id: 4, name: '赵六', age: 35, email: 'zhaoliu@example.com', avatar: '👩', status: 'pending' }
])

// 嵌套对象数组
const products = reactive([
  {
    id: 1,
    name: 'iPhone 14',
    price: 5999,
    category: '手机',
    tags: ['热销', '新品', '5G'],
    specs: {
      screen: '6.1英寸',
      storage: '128GB',
      color: '蓝色'
    },
    reviews: [
      { user: '用户A', rating: 5, comment: '非常好用' },
      { user: '用户B', rating: 4, comment: '性价比不错' }
    ]
  },
  {
    id: 2,
    name: 'MacBook Pro',
    price: 14999,
    category: '笔记本',
    tags: ['专业', '高性能'],
    specs: {
      screen: '14英寸',
      storage: '512GB',
      color: '深空灰'
    },
    reviews: [
      { user: '用户C', rating: 5, comment: '性能强劲' },
      { user: '用户D', rating: 5, comment: '做工精良' }
    ]
  },
  {
    id: 3,
    name: 'iPad Air',
    price: 4399,
    category: '平板',
    tags: ['轻薄', '便携'],
    specs: {
      screen: '10.9英寸',
      storage: '64GB',
      color: '玫瑰金'
    },
    reviews: [
      { user: '用户E', rating: 4, comment: '很好用' }
    ]
  }
])

// 对象属性遍历
const userProfile = reactive({
  name: '张三',
  age: 25,
  email: 'zhangsan@example.com',
  phone: '13800138000',
  address: '北京市朝阳区',
  hobbies: ['阅读', '游泳', '音乐'],
  skills: {
    frontend: 'Vue, React',
    backend: 'Node.js, Python',
    database: 'MySQL, MongoDB'
  }
})

// 动态列表数据
const dynamicItems = reactive([])

// 性能测试数据
const performanceTest = reactive({
  items: [],
  itemCount: 1000,
  useKey: true,
  shuffle: false
})

// 过滤和排序
const filterSort = reactive({
  searchQuery: '',
  selectedCategory: 'all',
  sortBy: 'name',  // name, price, rating
  sortOrder: 'asc'  // asc, desc
})

// ==================== 计算属性 ====================

// 筛选后的产品
const filteredProducts = computed(() => {
  let result = [...products]
  
  // 搜索过滤
  if (filterSort.searchQuery) {
    const query = filterSort.searchQuery.toLowerCase()
    result = result.filter(product => 
      product.name.toLowerCase().includes(query) ||
      product.category.toLowerCase().includes(query) ||
      product.tags.some(tag => tag.toLowerCase().includes(query))
    )
  }
  
  // 分类过滤
  if (filterSort.selectedCategory !== 'all') {
    result = result.filter(product => product.category === filterSort.selectedCategory)
  }
  
  // 排序
  result.sort((a, b) => {
    let compareValue = 0
    
    if (filterSort.sortBy === 'name') {
      compareValue = a.name.localeCompare(b.name)
    } else if (filterSort.sortBy === 'price') {
      compareValue = a.price - b.price
    } else if (filterSort.sortBy === 'rating') {
      const avgRatingA = a.reviews.reduce((sum, r) => sum + r.rating, 0) / a.reviews.length
      const avgRatingB = b.reviews.reduce((sum, r) => sum + r.rating, 0) / b.reviews.length
      compareValue = avgRatingA - avgRatingB
    }
    
    return filterSort.sortOrder === 'asc' ? compareValue : -compareValue
  })
  
  return result
})

// 产品分类列表
const categories = computed(() => {
  const cats = new Set(products.map(p => p.category))
  return ['all', ...Array.from(cats)]
})

// 用户统计
const userStats = computed(() => {
  const stats = {
    total: users.length,
    active: users.filter(u => u.status === 'active').length,
    inactive: users.filter(u => u.status === 'inactive').length,
    pending: users.filter(u => u.status === 'pending').length
  }
  
  const avgAge = users.reduce((sum, u) => sum + u.age, 0) / users.length
  stats.avgAge = Math.round(avgAge)
  
  return stats
})

// ==================== 方法 ====================

// 数组操作
function addFruit() {
  const newFruits = ['芒果', '菠萝', '西瓜', '桃子', '梨', '柚子']
  const randomFruit = newFruits[Math.floor(Math.random() * newFruits.length)]
  if (!fruits.includes(randomFruit)) {
    fruits.push(randomFruit)
  }
}

function removeFruit(index) {
  fruits.splice(index, 1)
}

function shuffleFruits() {
  for (let i = fruits.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [fruits[i], fruits[j]] = [fruits[j], fruits[i]]
  }
}

function reverseFruits() {
  fruits.reverse()
}

function sortFruits() {
  fruits.sort()
}

// 用户操作
function addUser() {
  const names = ['小明', '小红', '小刚', '小美', '小华', '小丽']
  const avatars = ['👨', '👩', '🧑', '👴', '👵']
  const statuses = ['active', 'inactive', 'pending']
  
  const newUser = {
    id: Date.now(),
    name: names[Math.floor(Math.random() * names.length)],
    age: Math.floor(Math.random() * 40) + 20,
    email: `user${Date.now()}@example.com`,
    avatar: avatars[Math.floor(Math.random() * avatars.length)],
    status: statuses[Math.floor(Math.random() * statuses.length)]
  }
  
  users.push(newUser)
}

function removeUser(id) {
  const index = users.findIndex(u => u.id === id)
  if (index > -1) {
    users.splice(index, 1)
  }
}

function toggleUserStatus(user) {
  const statuses = ['active', 'inactive', 'pending']
  const currentIndex = statuses.indexOf(user.status)
  const nextIndex = (currentIndex + 1) % statuses.length
  user.status = statuses[nextIndex]
}

// 产品操作
function addReview(product) {
  const users = ['用户X', '用户Y', '用户Z']
  const comments = ['很满意', '不错的产品', '值得购买', '质量很好', '性价比高']
  
  const newReview = {
    user: users[Math.floor(Math.random() * users.length)],
    rating: Math.floor(Math.random() * 5) + 1,
    comment: comments[Math.floor(Math.random() * comments.length)]
  }
  
  product.reviews.push(newReview)
}

function removeReview(product, index) {
  product.reviews.splice(index, 1)
}

// 动态列表操作
function generateDynamicItems() {
  dynamicItems.splice(0) // 清空数组
  
  for (let i = 0; i < 50; i++) {
    dynamicItems.push({
      id: i + 1,
      name: `Item ${i + 1}`,
      value: Math.floor(Math.random() * 100),
      color: `hsl(${Math.random() * 360}, 70%, 60%)`,
      size: Math.floor(Math.random() * 3) + 1
    })
  }
}

function shuffleDynamicItems() {
  for (let i = dynamicItems.length - 1; i > 0; i--) {
    const j = Math.floor(Math.random() * (i + 1));
    [dynamicItems[i], dynamicItems[j]] = [dynamicItems[j], dynamicItems[i]]
  }
}

function addDynamicItem() {
  const newItem = {
    id: dynamicItems.length + 1,
    name: `New Item ${dynamicItems.length + 1}`,
    value: Math.floor(Math.random() * 100),
    color: `hsl(${Math.random() * 360}, 70%, 60%)`,
    size: Math.floor(Math.random() * 3) + 1
  }
  
  const insertIndex = Math.floor(Math.random() * (dynamicItems.length + 1))
  dynamicItems.splice(insertIndex, 0, newItem)
}

function removeDynamicItem() {
  if (dynamicItems.length > 0) {
    const removeIndex = Math.floor(Math.random() * dynamicItems.length)
    dynamicItems.splice(removeIndex, 1)
  }
}

// 性能测试
function generatePerformanceItems() {
  performanceTest.items.splice(0)
  
  for (let i = 0; i < performanceTest.itemCount; i++) {
    performanceTest.items.push({
      id: i + 1,
      name: `Performance Item ${i + 1}`,
      value: Math.floor(Math.random() * 1000)
    })
  }
}

function shufflePerformanceItems() {
  performanceTest.shuffle = !performanceTest.shuffle
  
  if (performanceTest.shuffle) {
    for (let i = performanceTest.items.length - 1; i > 0; i--) {
      const j = Math.floor(Math.random() * (i + 1));
      [performanceTest.items[i], performanceTest.items[j]] = [performanceTest.items[j], performanceTest.items[i]]
    }
  }
}

function toggleKey() {
  performanceTest.useKey = !performanceTest.useKey
}

// 获取平均评分
function getAverageRating(reviews) {
  if (reviews.length === 0) return 0
  const sum = reviews.reduce((total, review) => total + review.rating, 0)
  return (sum / reviews.length).toFixed(1)
}

// 获取评星显示
function getStars(rating) {
  const fullStars = Math.floor(rating)
  const hasHalfStar = rating - fullStars >= 0.5
  const emptyStars = 5 - fullStars - (hasHalfStar ? 1 : 0)
  
  return '★'.repeat(fullStars) + 
         (hasHalfStar ? '☆' : '') + 
         '☆'.repeat(emptyStars)
}

// 初始化
onMounted(() => {
  generateDynamicItems()
  generatePerformanceItems()
})
</script>

<template>
  <div class="list-rendering">
    <h2>📋 列表渲染进阶学习</h2>
    
    <!-- 基础数组渲染 -->
    <div class="section">
      <h3>1. 基础数组渲染</h3>
      <div class="card">
        <div class="demo-group">
          <h4>简单数组 - 水果列表</h4>
          
          <div class="fruit-list">
            <div class="fruit-display">
              <div v-for="(fruit, index) in fruits" :key="index" class="fruit-item">
                <span class="fruit-icon">🍎</span>
                <span class="fruit-name">{{ fruit }}</span>
                <span class="fruit-index">#{{ index }}</span>
                <button @click="removeFruit(index)" class="remove-btn">删除</button>
              </div>
            </div>
            
            <div class="fruit-controls">
              <button @click="addFruit">添加水果</button>
              <button @click="shuffleFruits">随机排序</button>
              <button @click="reverseFruits">倒序</button>
              <button @click="sortFruits">字母排序</button>
            </div>
            
            <div class="fruit-info">
              <p>总计: {{ fruits.length }} 种水果</p>
              <p>列表: {{ fruits.join(', ') }}</p>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 对象数组渲染 -->
    <div class="section">
      <h3>2. 对象数组渲染</h3>
      <div class="card">
        <div class="demo-group">
          <h4>用户列表 - 复杂对象数组</h4>
          
          <div class="user-stats">
            <div class="stat-item">
              <span class="stat-label">总用户:</span>
              <span class="stat-value">{{ userStats.total }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">活跃:</span>
              <span class="stat-value active">{{ userStats.active }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">非活跃:</span>
              <span class="stat-value inactive">{{ userStats.inactive }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">待审核:</span>
              <span class="stat-value pending">{{ userStats.pending }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">平均年龄:</span>
              <span class="stat-value">{{ userStats.avgAge }}岁</span>
            </div>
          </div>
          
          <div class="user-list">
            <div v-for="user in users" :key="user.id" class="user-card">
              <div class="user-avatar">{{ user.avatar }}</div>
              <div class="user-info">
                <h5 class="user-name">{{ user.name }}</h5>
                <p class="user-details">{{ user.age }}岁 | {{ user.email }}</p>
                <span :class="['user-status', user.status]">
                  {{ user.status === 'active' ? '活跃' : 
                     user.status === 'inactive' ? '非活跃' : '待审核' }}
                </span>
              </div>
              <div class="user-actions">
                <button @click="toggleUserStatus(user)" class="status-btn">
                  切换状态
                </button>
                <button @click="removeUser(user.id)" class="remove-btn">删除</button>
              </div>
            </div>
          </div>
          
          <div class="user-controls">
            <button @click="addUser">添加用户</button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 嵌套对象渲染 -->
    <div class="section">
      <h3>3. 嵌套对象渲染</h3>
      <div class="card">
        <div class="demo-group">
          <h4>产品列表 - 深层嵌套结构</h4>
          
          <!-- 筛选控制 -->
          <div class="filter-controls">
            <div class="filter-group">
              <label>搜索:</label>
              <input v-model="filterSort.searchQuery" placeholder="搜索产品...">
            </div>
            
            <div class="filter-group">
              <label>分类:</label>
              <select v-model="filterSort.selectedCategory">
                <option v-for="category in categories" :key="category" :value="category">
                  {{ category === 'all' ? '全部' : category }}
                </option>
              </select>
            </div>
            
            <div class="filter-group">
              <label>排序:</label>
              <select v-model="filterSort.sortBy">
                <option value="name">名称</option>
                <option value="price">价格</option>
                <option value="rating">评分</option>
              </select>
              <button @click="filterSort.sortOrder = filterSort.sortOrder === 'asc' ? 'desc' : 'asc'">
                {{ filterSort.sortOrder === 'asc' ? '↑' : '↓' }}
              </button>
            </div>
          </div>
          
          <!-- 产品列表 -->
          <div class="product-list">
            <div v-for="product in filteredProducts" :key="product.id" class="product-card">
              <div class="product-header">
                <h5 class="product-name">{{ product.name }}</h5>
                <span class="product-price">¥{{ product.price }}</span>
              </div>
              
              <div class="product-info">
                <span class="product-category">{{ product.category }}</span>
                <div class="product-tags">
                  <span v-for="tag in product.tags" :key="tag" class="tag">
                    {{ tag }}
                  </span>
                </div>
              </div>
              
              <div class="product-specs">
                <h6>规格参数:</h6>
                <div class="specs-list">
                  <div v-for="(value, key) in product.specs" :key="key" class="spec-item">
                    <span class="spec-key">{{ key }}:</span>
                    <span class="spec-value">{{ value }}</span>
                  </div>
                </div>
              </div>
              
              <div class="product-reviews">
                <div class="reviews-header">
                  <h6>用户评价 ({{ product.reviews.length }}条)</h6>
                  <span class="average-rating">
                    {{ getStars(getAverageRating(product.reviews)) }}
                    {{ getAverageRating(product.reviews) }}
                  </span>
                </div>
                
                <div class="reviews-list">
                  <div v-for="(review, index) in product.reviews" :key="index" class="review-item">
                    <div class="review-header">
                      <span class="review-user">{{ review.user }}</span>
                      <span class="review-rating">
                        {{ getStars(review.rating) }}
                      </span>
                    </div>
                    <p class="review-comment">{{ review.comment }}</p>
                    <button @click="removeReview(product, index)" class="remove-review-btn">
                      删除评价
                    </button>
                  </div>
                </div>
                
                <button @click="addReview(product)" class="add-review-btn">
                  添加评价
                </button>
              </div>
            </div>
            
            <div v-if="filteredProducts.length === 0" class="no-products">
              没有找到匹配的产品
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 对象属性遍历 -->
    <div class="section">
      <h3>4. 对象属性遍历</h3>
      <div class="card">
        <div class="demo-group">
          <h4>v-for 遍历对象属性</h4>
          
          <div class="profile-display">
            <h5>基础信息:</h5>
            <div class="profile-basic">
              <div v-for="(value, key) in userProfile" :key="key" class="profile-item">
                <template v-if="typeof value === 'string' || typeof value === 'number'">
                  <span class="profile-key">{{ key }}:</span>
                  <span class="profile-value">{{ value }}</span>
                </template>
              </div>
            </div>
            
            <h5>爱好列表:</h5>
            <div class="hobbies-list">
              <span v-for="(hobby, index) in userProfile.hobbies" :key="index" class="hobby-tag">
                {{ hobby }}
              </span>
            </div>
            
            <h5>技能详情:</h5>
            <div class="skills-list">
              <div v-for="(skill, category) in userProfile.skills" :key="category" class="skill-item">
                <span class="skill-category">{{ category }}:</span>
                <span class="skill-value">{{ skill }}</span>
              </div>
            </div>
            
            <h5>带索引的遍历:</h5>
            <div class="indexed-list">
              <div v-for="(value, key, index) in userProfile" :key="key" class="indexed-item">
                <span class="item-index">{{ index }}.</span>
                <span class="item-key">{{ key }}:</span>
                <span class="item-value">{{ value }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 数字遍历 -->
    <div class="section">
      <h3>5. 数字范围遍历</h3>
      <div class="card">
        <div class="demo-group">
          <h4>v-for 遍历数字</h4>
          
          <div class="number-examples">
            <div class="number-grid">
              <h5>1-12 数字网格:</h5>
              <div class="number-grid-display">
                <div v-for="n in 12" :key="n" class="number-cell">
                  {{ n }}
                </div>
              </div>
            </div>
            
            <div class="progress-bar">
              <h5>进度条 (1-10):</h5>
              <div class="progress-display">
                <div v-for="n in 10" :key="n" class="progress-step" :class="{ active: n <= 7 }">
                  {{ n }}
                </div>
              </div>
            </div>
            
            <div class="star-rating">
              <h5>星级评分:</h5>
              <div class="stars-display">
                <span v-for="n in 5" :key="n" class="star" :class="{ filled: n <= 3 }">
                  ★
                </span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- key 的重要性演示 -->
    <div class="section">
      <h3>6. key 属性的重要性</h3>
      <div class="card">
        <div class="demo-group">
          <h4>使用 key 优化渲染性能</h4>
          
          <div class="key-demo">
            <div class="controls">
              <button @click="shuffleDynamicItems">随机排序</button>
              <button @click="addDynamicItem">随机插入</button>
              <button @click="removeDynamicItem">随机删除</button>
              <button @click="generateDynamicItems">重新生成</button>
            </div>
            
            <div class="dynamic-list">
              <div 
                v-for="item in dynamicItems" 
                :key="item.id" 
                class="dynamic-item"
                :style="{ 
                  backgroundColor: item.color,
                  transform: `scale(${item.size * 0.3 + 0.7})`
                }"
              >
                <span class="item-id">ID: {{ item.id }}</span>
                <span class="item-name">{{ item.name }}</span>
                <span class="item-value">值: {{ item.value }}</span>
              </div>
            </div>
            
            <div class="key-explanation">
              <h5>💡 key 的作用:</h5>
              <ul>
                <li>✅ 提供唯一标识，帮助Vue跟踪节点</li>
                <li>✅ 避免就地复用导致的渲染错误</li>
                <li>✅ 优化列表更新性能</li>
                <li>⚠️ 不要使用数组索引作为key（在动态列表中）</li>
              </ul>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 性能测试 -->
    <div class="section">
      <h3>7. 性能测试</h3>
      <div class="card">
        <div class="demo-group">
          <h4>大列表渲染性能对比</h4>
          
          <div class="performance-controls">
            <div class="control-group">
              <label>列表大小:</label>
              <input 
                v-model.number="performanceTest.itemCount" 
                type="number" 
                min="100" 
                max="10000"
                step="100"
              >
              <button @click="generatePerformanceItems">生成列表</button>
            </div>
            
            <div class="control-group">
              <label>使用key:</label>
              <button @click="toggleKey" :class="{ active: performanceTest.useKey }">
                {{ performanceTest.useKey ? '✅ 使用key' : '❌ 不使用key' }}
              </button>
            </div>
            
            <div class="control-group">
              <button @click="shufflePerformanceItems">
                {{ performanceTest.shuffle ? '恢复顺序' : '随机排序' }}
              </button>
            </div>
          </div>
          
          <div class="performance-info">
            <p>当前列表: {{ performanceTest.items.length }} 项</p>
            <p>渲染模式: {{ performanceTest.useKey ? '使用唯一key' : '使用索引key' }}</p>
            <p class="tip">💡 打开浏览器开发者工具，观察DOM更新情况</p>
          </div>
          
          <div class="performance-list">
            <div 
              v-for="(item, index) in performanceTest.items" 
              :key="performanceTest.useKey ? item.id : index"
              class="performance-item"
            >
              <span class="item-id">{{ item.id }}</span>
              <span class="item-name">{{ item.name }}</span>
              <span class="item-value">{{ item.value }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 总结 -->
    <div class="section summary">
      <h3>📝 列表渲染总结</h3>
      <div class="card">
        <div class="summary-grid">
          <div class="summary-item">
            <h4>🎯 基础语法</h4>
            <ul>
              <li><code>v-for="item in items"</code> - 数组遍历</li>
              <li><code>v-for="(value, key) in object"</code> - 对象遍历</li>
              <li><code>v-for="n in 10"</code> - 数字遍历</li>
              <li><code>:key="unique-id"</code> - 唯一标识</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>🔑 key 的重要性</h4>
            <ul>
              <li>提供稳定的节点标识</li>
              <li>避免就地复用问题</li>
              <li>优化更新性能</li>
              <li>使用唯一值而非索引</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>⚡ 性能优化</h4>
            <ul>
              <li>合理使用计算属性过滤</li>
              <li>避免在模板中进行复杂计算</li>
              <li>大列表考虑虚拟滚动</li>
              <li>使用Object.freeze冻结大数据</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>📋 最佳实践</h4>
            <ul>
              <li>始终提供key属性</li>
              <li>避免在v-for中使用v-if</li>
              <li>使用template标签分组</li>
              <li>合理组织数据结构</li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.list-rendering {
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

/* 水果列表样式 */
.fruit-display {
  display: flex;
  flex-wrap: wrap;
  gap: 10px;
  margin: 15px 0;
}

.fruit-item {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 12px;
  background: #f5f5f5;
  border-radius: 20px;
  border: 2px solid #ddd;
  transition: all 0.3s;
}

.fruit-item:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.fruit-icon {
  font-size: 16px;
}

.fruit-name {
  font-weight: bold;
  color: #333;
}

.fruit-index {
  font-size: 12px;
  color: #666;
  background: #ddd;
  padding: 2px 6px;
  border-radius: 10px;
}

.fruit-controls {
  display: flex;
  gap: 10px;
  margin: 15px 0;
  flex-wrap: wrap;
}

.fruit-info {
  background: #f9f9f9;
  padding: 10px;
  border-radius: 4px;
  margin-top: 15px;
}

/* 用户列表样式 */
.user-stats {
  display: flex;
  gap: 15px;
  margin: 15px 0;
  flex-wrap: wrap;
}

.stat-item {
  display: flex;
  flex-direction: column;
  align-items: center;
  padding: 10px;
  background: #f5f5f5;
  border-radius: 8px;
  min-width: 80px;
}

.stat-label {
  font-size: 12px;
  color: #666;
  margin-bottom: 4px;
}

.stat-value {
  font-size: 18px;
  font-weight: bold;
  color: #333;
}

.stat-value.active {
  color: #4caf50;
}

.stat-value.inactive {
  color: #f44336;
}

.stat-value.pending {
  color: #ff9800;
}

.user-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
  gap: 15px;
  margin: 20px 0;
}

.user-card {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 15px;
  background: #f9f9f9;
  border-radius: 8px;
  border: 2px solid #ddd;
  transition: all 0.3s;
}

.user-card:hover {
  border-color: #42b883;
  box-shadow: 0 4px 12px rgba(66, 184, 131, 0.2);
}

.user-avatar {
  font-size: 24px;
  width: 40px;
  height: 40px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #e0e0e0;
  border-radius: 50%;
}

.user-info {
  flex: 1;
}

.user-name {
  margin: 0 0 5px 0;
  color: #333;
}

.user-details {
  margin: 0;
  color: #666;
  font-size: 14px;
}

.user-status {
  padding: 2px 8px;
  border-radius: 12px;
  font-size: 12px;
  font-weight: bold;
}

.user-status.active {
  background: #e8f5e9;
  color: #2e7d32;
}

.user-status.inactive {
  background: #ffebee;
  color: #c62828;
}

.user-status.pending {
  background: #fff3e0;
  color: #e65100;
}

.user-actions {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

/* 产品列表样式 */
.filter-controls {
  display: flex;
  gap: 20px;
  margin: 15px 0;
  flex-wrap: wrap;
  align-items: center;
}

.filter-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.filter-group label {
  font-weight: bold;
  color: #333;
}

.filter-group input,
.filter-group select {
  padding: 6px 10px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.filter-group input:focus,
.filter-group select:focus {
  outline: none;
  border-color: #42b883;
}

.product-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(400px, 1fr));
  gap: 20px;
  margin: 20px 0;
}

.product-card {
  background: #f9f9f9;
  padding: 20px;
  border-radius: 8px;
  border: 2px solid #ddd;
  transition: all 0.3s;
}

.product-card:hover {
  border-color: #42b883;
  box-shadow: 0 4px 12px rgba(66, 184, 131, 0.2);
}

.product-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.product-name {
  margin: 0;
  color: #333;
}

.product-price {
  font-size: 18px;
  font-weight: bold;
  color: #42b883;
}

.product-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.product-category {
  background: #e0e0e0;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 12px;
  color: #333;
}

.product-tags {
  display: flex;
  gap: 5px;
}

.tag {
  background: #42b883;
  color: white;
  padding: 2px 6px;
  border-radius: 10px;
  font-size: 11px;
}

.product-specs {
  margin: 15px 0;
}

.product-specs h6 {
  margin: 0 0 8px 0;
  color: #333;
}

.specs-list {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 5px;
}

.spec-item {
  display: flex;
  justify-content: space-between;
  padding: 4px;
  background: #f0f0f0;
  border-radius: 4px;
  font-size: 14px;
}

.spec-key {
  color: #666;
}

.spec-value {
  font-weight: bold;
  color: #333;
}

.product-reviews {
  margin-top: 15px;
  padding-top: 15px;
  border-top: 1px solid #ddd;
}

.reviews-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.reviews-header h6 {
  margin: 0;
  color: #333;
}

.average-rating {
  font-size: 14px;
  color: #ff9800;
}

.reviews-list {
  max-height: 200px;
  overflow-y: auto;
  margin-bottom: 10px;
}

.review-item {
  background: #f0f0f0;
  padding: 10px;
  border-radius: 4px;
  margin-bottom: 8px;
}

.review-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 5px;
}

.review-user {
  font-weight: bold;
  color: #333;
}

.review-rating {
  color: #ff9800;
}

.review-comment {
  margin: 0;
  color: #666;
  font-size: 14px;
}

.remove-review-btn {
  background: #f44336;
  color: white;
  border: none;
  padding: 4px 8px;
  border-radius: 4px;
  font-size: 12px;
  cursor: pointer;
  margin-top: 5px;
}

.add-review-btn {
  background: #42b883;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
}

.no-products {
  text-align: center;
  padding: 40px;
  color: #999;
  font-size: 18px;
}

/* 对象属性遍历样式 */
.profile-display {
  max-width: 800px;
}

.profile-basic {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 10px;
  margin: 15px 0;
}

.profile-item {
  display: flex;
  justify-content: space-between;
  padding: 8px 12px;
  background: #f5f5f5;
  border-radius: 4px;
}

.profile-key {
  font-weight: bold;
  color: #333;
}

.profile-value {
  color: #666;
}

.hobbies-list {
  display: flex;
  gap: 8px;
  flex-wrap: wrap;
  margin: 15px 0;
}

.hobby-tag {
  background: #42b883;
  color: white;
  padding: 4px 8px;
  border-radius: 12px;
  font-size: 14px;
}

.skills-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin: 15px 0;
}

.skill-item {
  display: flex;
  gap: 10px;
  padding: 8px 12px;
  background: #f0f8ff;
  border-radius: 4px;
}

.skill-category {
  font-weight: bold;
  color: #1976d2;
  min-width: 80px;
}

.skill-value {
  color: #333;
}

.indexed-list {
  display: flex;
  flex-direction: column;
  gap: 5px;
  margin: 15px 0;
}

.indexed-item {
  display: flex;
  gap: 10px;
  padding: 6px 10px;
  background: #f9f9f9;
  border-radius: 4px;
  font-size: 14px;
}

.item-index {
  font-weight: bold;
  color: #666;
  min-width: 20px;
}

.item-key {
  font-weight: bold;
  color: #333;
  min-width: 100px;
}

.item-value {
  color: #666;
}

/* 数字遍历样式 */
.number-examples {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.number-grid-display {
  display: grid;
  grid-template-columns: repeat(6, 1fr);
  gap: 10px;
  max-width: 400px;
}

.number-cell {
  width: 50px;
  height: 50px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #42b883;
  color: white;
  border-radius: 8px;
  font-weight: bold;
  transition: all 0.3s;
  cursor: pointer;
}

.number-cell:hover {
  transform: scale(1.1);
  background: #35a372;
}

.progress-display {
  display: flex;
  gap: 5px;
  max-width: 600px;
}

.progress-step {
  flex: 1;
  height: 30px;
  display: flex;
  align-items: center;
  justify-content: center;
  background: #e0e0e0;
  color: #999;
  border-radius: 4px;
  transition: all 0.3s;
  position: relative;
}

.progress-step.active {
  background: #42b883;
  color: white;
}

.progress-step.active::after {
  content: '✓';
  position: absolute;
  top: -5px;
  right: -5px;
  background: #4caf50;
  color: white;
  width: 16px;
  height: 16px;
  border-radius: 50%;
  display: flex;
  align-items: center;
  justify-content: center;
  font-size: 10px;
}

.stars-display {
  display: flex;
  gap: 5px;
}

.star {
  font-size: 24px;
  color: #ddd;
  cursor: pointer;
  transition: all 0.3s;
}

.star.filled {
  color: #ff9800;
}

.star:hover {
  transform: scale(1.2);
}

/* 动态列表样式 */
.key-demo {
  max-width: 1000px;
}

.dynamic-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 10px;
  margin: 20px 0;
  max-height: 400px;
  overflow-y: auto;
}

.dynamic-item {
  padding: 10px;
  border-radius: 8px;
  color: white;
  font-weight: bold;
  text-shadow: 1px 1px 2px rgba(0, 0, 0, 0.5);
  transition: all 0.3s ease;
  cursor: pointer;
}

.dynamic-item:hover {
  transform: translateY(-2px) !important;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.3);
}

.item-id {
  display: block;
  font-size: 12px;
  opacity: 0.8;
}

.item-name {
  display: block;
  font-size: 14px;
  margin: 5px 0;
}

.item-value {
  display: block;
  font-size: 16px;
}

.key-explanation {
  background: #f0f8ff;
  padding: 15px;
  border-radius: 8px;
  margin-top: 20px;
}

.key-explanation ul {
  margin: 10px 0;
  padding-left: 20px;
}

.key-explanation li {
  margin: 5px 0;
}

/* 性能测试样式 */
.performance-controls {
  display: flex;
  gap: 20px;
  margin: 15px 0;
  flex-wrap: wrap;
  align-items: center;
}

.control-group {
  display: flex;
  align-items: center;
  gap: 8px;
}

.control-group label {
  font-weight: bold;
  color: #333;
}

.control-group input {
  width: 100px;
  padding: 6px 10px;
  border: 2px solid #ddd;
  border-radius: 4px;
}

.control-group button.active {
  background: #4caf50;
}

.performance-info {
  background: #f9f9f9;
  padding: 15px;
  border-radius: 8px;
  margin: 15px 0;
}

.performance-info .tip {
  color: #666;
  font-style: italic;
  margin-top: 8px;
}

.performance-list {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(250px, 1fr));
  gap: 5px;
  max-height: 300px;
  overflow-y: auto;
  border: 1px solid #ddd;
  padding: 10px;
  border-radius: 4px;
}

.performance-item {
  display: flex;
  gap: 10px;
  padding: 5px 10px;
  background: #f5f5f5;
  border-radius: 4px;
  font-size: 14px;
  align-items: center;
}

.performance-item .item-id {
  color: #666;
  font-size: 12px;
  min-width: 30px;
}

.performance-item .item-name {
  flex: 1;
  color: #333;
}

.performance-item .item-value {
  color: #42b883;
  font-weight: bold;
  min-width: 40px;
}

/* 按钮样式 */
button {
  padding: 8px 16px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s;
  font-size: 14px;
}

button:hover {
  background: #35a372;
}

.remove-btn {
  background: #f44336;
  padding: 4px 8px;
  font-size: 12px;
}

.remove-btn:hover {
  background: #d32f2f;
}

.status-btn {
  background: #ff9800;
  font-size: 12px;
}

.status-btn:hover {
  background: #f57c00;
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
  .filter-controls {
    flex-direction: column;
    align-items: stretch;
  }
  
  .filter-group {
    justify-content: space-between;
  }
  
  .product-list {
    grid-template-columns: 1fr;
  }
  
  .user-list {
    grid-template-columns: 1fr;
  }
  
  .dynamic-list {
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  }
  
  .performance-list {
    grid-template-columns: 1fr;
  }
  
  .number-grid-display {
    grid-template-columns: repeat(4, 1fr);
  }
  
  .summary-grid {
    grid-template-columns: 1fr;
  }
  
  .performance-controls {
    flex-direction: column;
    align-items: stretch;
  }
  
  .control-group {
    justify-content: space-between;
  }
}
</style>