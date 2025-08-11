<script setup>
// Day 2 - computed 计算属性示例
import { ref, reactive, computed } from 'vue'

// ==================== 基础示例 ====================

const firstName = ref('张')
const lastName = ref('三')

// 只读计算属性
const fullName = computed(() => {
  console.log('fullName 计算执行')
  return firstName.value + ' ' + lastName.value
})

// 可写计算属性
const fullNameWritable = computed({
  get() {
    return firstName.value + ' ' + lastName.value
  },
  set(newValue) {
    const names = newValue.split(' ')
    firstName.value = names[0] || ''
    lastName.value = names[1] || ''
  }
})

function updateWritableName() {
  fullNameWritable.value = '李 四'
}

// ==================== computed vs methods 对比 ====================

const count = ref(0)
let computedCallCount = 0
let methodCallCount = 0

// computed：有缓存，依赖不变不会重新计算
const doubleComputed = computed(() => {
  computedCallCount++
  console.log(`computed 执行了，第 ${computedCallCount} 次`)
  return count.value * 2
})

// method：每次调用都会执行
function doubleMethod() {
  methodCallCount++
  console.log(`method 执行了，第 ${methodCallCount} 次`)
  return count.value * 2
}

// 触发更新（不改变count）
const triggerUpdate = ref(0)
function forceUpdate() {
  triggerUpdate.value++
}

// ==================== 购物车示例 ====================

const cart = reactive({
  items: [
    { id: 1, name: '🍎 苹果', price: 10, quantity: 2, checked: true },
    { id: 2, name: '🍌 香蕉', price: 5, quantity: 3, checked: true },
    { id: 3, name: '🍊 橙子', price: 8, quantity: 1, checked: false }
  ],
  discount: 0.9,  // 9折
  coupon: 0       // 优惠券金额
})

// 计算选中的商品
const selectedItems = computed(() => {
  return cart.items.filter(item => item.checked)
})

// 计算原始总价
const originalPrice = computed(() => {
  return selectedItems.value.reduce((sum, item) => {
    return sum + item.price * item.quantity
  }, 0)
})

// 计算折后价
const discountedPrice = computed(() => {
  return originalPrice.value * cart.discount
})

// 计算最终价格
const finalPrice = computed(() => {
  const price = discountedPrice.value - cart.coupon
  return Math.max(0, price).toFixed(2)
})

// 计算节省金额
const savedAmount = computed(() => {
  return (originalPrice.value - finalPrice.value).toFixed(2)
})

// 计算总数量
const totalQuantity = computed(() => {
  return selectedItems.value.reduce((sum, item) => sum + item.quantity, 0)
})

// 操作方法
function increaseQuantity(item) {
  item.quantity++
}

function decreaseQuantity(item) {
  if (item.quantity > 1) {
    item.quantity--
  }
}

function toggleItem(item) {
  item.checked = !item.checked
}

function applyCoupon() {
  cart.coupon = 10  // 应用10元优惠券
}

function removeCoupon() {
  cart.coupon = 0
}

// ==================== 搜索过滤示例 ====================

const products = reactive([
  { id: 1, name: 'iPhone 14', category: '手机', price: 5999 },
  { id: 2, name: 'iPad Pro', category: '平板', price: 8999 },
  { id: 3, name: 'MacBook Pro', category: '笔记本', price: 14999 },
  { id: 4, name: 'AirPods Pro', category: '耳机', price: 1999 },
  { id: 5, name: 'Apple Watch', category: '手表', price: 2999 }
])

const searchQuery = ref('')
const selectedCategory = ref('全部')
const priceRange = reactive({
  min: 0,
  max: 20000
})

// 获取所有分类
const categories = computed(() => {
  const cats = ['全部', ...new Set(products.map(p => p.category))]
  return cats
})

// 过滤产品
const filteredProducts = computed(() => {
  let result = products
  
  // 按关键词过滤
  if (searchQuery.value) {
    result = result.filter(p => 
      p.name.toLowerCase().includes(searchQuery.value.toLowerCase())
    )
  }
  
  // 按分类过滤
  if (selectedCategory.value !== '全部') {
    result = result.filter(p => p.category === selectedCategory.value)
  }
  
  // 按价格范围过滤
  result = result.filter(p => 
    p.price >= priceRange.min && p.price <= priceRange.max
  )
  
  return result
})

// 统计信息
const filterStats = computed(() => {
  return {
    total: filteredProducts.value.length,
    avgPrice: filteredProducts.value.length > 0
      ? (filteredProducts.value.reduce((sum, p) => sum + p.price, 0) / filteredProducts.value.length).toFixed(2)
      : 0
  }
})
</script>

<template>
  <div class="computed-example">
    <h2>🧮 Computed 计算属性示例</h2>
    
    <!-- 基础示例 -->
    <div class="section">
      <h3>1. 基础用法</h3>
      <div class="card">
        <div class="input-group">
          <input v-model="firstName" placeholder="姓">
          <input v-model="lastName" placeholder="名">
        </div>
        <p>只读计算属性: {{ fullName }}</p>
        <p>可写计算属性: {{ fullNameWritable }}</p>
        <button @click="updateWritableName">设置可写属性为"李 四"</button>
      </div>
    </div>
    
    <!-- computed vs methods -->
    <div class="section">
      <h3>2. Computed vs Methods 对比</h3>
      <div class="card">
        <p>基础值 count: {{ count }}</p>
        <button @click="count++">Count++</button>
        <button @click="forceUpdate">强制更新（不改变count）</button>
        
        <div class="comparison">
          <div class="computed-demo">
            <h4>Computed（有缓存）</h4>
            <p>结果: {{ doubleComputed }}</p>
            <p>结果: {{ doubleComputed }}</p>
            <p>结果: {{ doubleComputed }}</p>
            <p class="tip">调用3次，执行{{ computedCallCount }}次</p>
          </div>
          
          <div class="method-demo">
            <h4>Method（无缓存）</h4>
            <p>结果: {{ doubleMethod() }}</p>
            <p>结果: {{ doubleMethod() }}</p>
            <p>结果: {{ doubleMethod() }}</p>
            <p class="tip">调用3次，执行{{ methodCallCount }}次</p>
          </div>
        </div>
        
        <p class="tip">💡 打开控制台查看执行次数。Computed 只在依赖变化时重新计算！</p>
      </div>
    </div>
    
    <!-- 购物车示例 -->
    <div class="section">
      <h3>3. 购物车计算</h3>
      <div class="card">
        <table class="cart-table">
          <thead>
            <tr>
              <th>选择</th>
              <th>商品</th>
              <th>单价</th>
              <th>数量</th>
              <th>小计</th>
            </tr>
          </thead>
          <tbody>
            <tr v-for="item in cart.items" :key="item.id">
              <td>
                <input 
                  type="checkbox" 
                  :checked="item.checked"
                  @change="toggleItem(item)"
                >
              </td>
              <td>{{ item.name }}</td>
              <td>¥{{ item.price }}</td>
              <td>
                <button @click="decreaseQuantity(item)" class="small">-</button>
                <span class="quantity">{{ item.quantity }}</span>
                <button @click="increaseQuantity(item)" class="small">+</button>
              </td>
              <td>¥{{ item.price * item.quantity }}</td>
            </tr>
          </tbody>
        </table>
        
        <div class="cart-summary">
          <p>选中商品: {{ selectedItems.length }} 件</p>
          <p>总数量: {{ totalQuantity }} 个</p>
          <p>原价: ¥{{ originalPrice }}</p>
          <p>折扣 ({{ (cart.discount * 10).toFixed(0) }}折): ¥{{ discountedPrice.toFixed(2) }}</p>
          <p>优惠券: -¥{{ cart.coupon }}</p>
          <p class="final-price">最终价格: ¥{{ finalPrice }}</p>
          <p class="saved">已节省: ¥{{ savedAmount }}</p>
          
          <div class="coupon-actions">
            <button @click="applyCoupon" v-if="cart.coupon === 0">使用10元优惠券</button>
            <button @click="removeCoupon" v-else>取消优惠券</button>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 搜索过滤示例 -->
    <div class="section">
      <h3>4. 搜索过滤计算</h3>
      <div class="card">
        <div class="filter-controls">
          <input 
            v-model="searchQuery" 
            placeholder="搜索产品..."
            class="search-input"
          >
          
          <select v-model="selectedCategory">
            <option v-for="cat in categories" :key="cat">{{ cat }}</option>
          </select>
          
          <div class="price-range">
            <label>价格范围:</label>
            <input 
              type="number" 
              v-model.number="priceRange.min" 
              placeholder="最低价"
              class="small-input"
            >
            <span>-</span>
            <input 
              type="number" 
              v-model.number="priceRange.max" 
              placeholder="最高价"
              class="small-input"
            >
          </div>
        </div>
        
        <div class="filter-stats">
          <p>找到 {{ filterStats.total }} 个产品，平均价格: ¥{{ filterStats.avgPrice }}</p>
        </div>
        
        <div class="product-list">
          <div 
            v-for="product in filteredProducts" 
            :key="product.id"
            class="product-item"
          >
            <span class="product-name">{{ product.name }}</span>
            <span class="product-category">{{ product.category }}</span>
            <span class="product-price">¥{{ product.price }}</span>
          </div>
          
          <p v-if="filteredProducts.length === 0" class="no-results">
            没有找到匹配的产品
          </p>
        </div>
      </div>
    </div>
    
    <!-- 总结 -->
    <div class="section summary">
      <h3>📝 Computed 特点总结</h3>
      <ul>
        <li>✅ 基于响应式依赖自动缓存</li>
        <li>✅ 只有依赖变化时才重新计算</li>
        <li>✅ 适合复杂的数据转换和过滤</li>
        <li>✅ 可以创建可写的计算属性</li>
        <li>⚡ 比方法调用性能更好（有缓存）</li>
        <li>💡 应该是纯函数，避免副作用</li>
      </ul>
    </div>
  </div>
</template>

<style scoped>
.computed-example {
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

.input-group {
  display: flex;
  gap: 10px;
  margin-bottom: 15px;
}

input {
  padding: 8px 12px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

input:focus {
  outline: none;
  border-color: #42b883;
}

button {
  padding: 8px 16px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  margin-right: 10px;
  transition: background 0.3s;
}

button:hover {
  background: #35a372;
}

button.small {
  padding: 4px 8px;
  font-size: 12px;
}

.comparison {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
  margin: 20px 0;
}

.computed-demo,
.method-demo {
  padding: 15px;
  background: #f5f5f5;
  border-radius: 4px;
}

.computed-demo h4,
.method-demo h4 {
  margin-top: 0;
  color: #333;
}

.tip {
  color: #666;
  font-size: 14px;
  margin-top: 10px;
  font-style: italic;
}

/* 购物车样式 */
.cart-table {
  width: 100%;
  border-collapse: collapse;
  margin-bottom: 20px;
}

.cart-table th,
.cart-table td {
  padding: 10px;
  text-align: left;
  border-bottom: 1px solid #ddd;
}

.cart-table th {
  background: #f5f5f5;
  font-weight: bold;
}

.quantity {
  display: inline-block;
  min-width: 30px;
  text-align: center;
  margin: 0 5px;
}

.cart-summary {
  padding: 15px;
  background: #f9f9f9;
  border-radius: 4px;
}

.cart-summary p {
  margin: 8px 0;
}

.final-price {
  font-size: 20px;
  font-weight: bold;
  color: #42b883;
  margin: 15px 0;
}

.saved {
  color: #ff6b6b;
  font-weight: bold;
}

.coupon-actions {
  margin-top: 15px;
}

/* 搜索过滤样式 */
.filter-controls {
  display: flex;
  gap: 15px;
  margin-bottom: 20px;
  flex-wrap: wrap;
  align-items: center;
}

.search-input {
  flex: 1;
  min-width: 200px;
}

select {
  padding: 8px 12px;
  border: 2px solid #ddd;
  border-radius: 4px;
  background: white;
  cursor: pointer;
}

.price-range {
  display: flex;
  align-items: center;
  gap: 10px;
}

.small-input {
  width: 100px;
}

.filter-stats {
  padding: 10px;
  background: #e8f5e9;
  border-radius: 4px;
  margin-bottom: 15px;
}

.product-list {
  max-height: 300px;
  overflow-y: auto;
}

.product-item {
  display: flex;
  justify-content: space-between;
  padding: 10px;
  background: #f5f5f5;
  margin-bottom: 5px;
  border-radius: 4px;
}

.product-name {
  font-weight: bold;
}

.product-category {
  color: #666;
  font-size: 14px;
}

.product-price {
  color: #42b883;
  font-weight: bold;
}

.no-results {
  text-align: center;
  color: #999;
  padding: 30px;
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

.summary ul {
  margin: 0;
  padding-left: 20px;
}

.summary li {
  margin: 8px 0;
}

@media (max-width: 768px) {
  .comparison {
    grid-template-columns: 1fr;
  }
  
  .filter-controls {
    flex-direction: column;
    align-items: stretch;
  }
}
</style>