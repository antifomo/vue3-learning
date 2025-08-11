<script setup>
// Day 2 练习1：购物车系统
import { reactive, computed, watch } from 'vue'

// ==================== 状态管理 ====================

// 商品列表（模拟商品数据库）
const store = reactive({
  products: [
    { id: 1, name: '🍎 苹果 iPhone 14', price: 5999, stock: 10, category: '手机' },
    { id: 2, name: '💻 MacBook Pro', price: 14999, stock: 5, category: '电脑' },
    { id: 3, name: '🎧 AirPods Pro', price: 1999, stock: 20, category: '耳机' },
    { id: 4, name: '⌚ Apple Watch', price: 2999, stock: 15, category: '手表' },
    { id: 5, name: '📱 iPad Pro', price: 8999, stock: 8, category: '平板' },
    { id: 6, name: '🖱️ Magic Mouse', price: 599, stock: 30, category: '配件' },
    { id: 7, name: '⌨️ Magic Keyboard', price: 999, stock: 25, category: '配件' },
    { id: 8, name: '🔌 充电器 20W', price: 149, stock: 50, category: '配件' }
  ]
})

// 购物车状态
const cart = reactive({
  items: [],  // 购物车商品列表
  couponCode: '',  // 输入的优惠券码
  appliedCoupon: null,  // 已应用的优惠券
  shippingFee: 0  // 运费
})

// 优惠券列表（模拟）
const coupons = [
  { code: 'SAVE10', type: 'percent', value: 10, minAmount: 100, description: '满100减10%' },
  { code: 'SAVE50', type: 'fixed', value: 50, minAmount: 200, description: '满200减50' },
  { code: 'SAVE100', type: 'fixed', value: 100, minAmount: 500, description: '满500减100' },
  { code: 'FREE', type: 'percent', value: 100, minAmount: 10000, description: '满10000免单' }
]

// 同步状态（模拟）
const syncStatus = reactive({
  isSyncing: false,
  lastSyncTime: null,
  syncMessage: ''
})

// ==================== 购物车操作 ====================

// 添加到购物车
function addToCart(product) {
  const existingItem = cart.items.find(item => item.id === product.id)
  
  if (existingItem) {
    // 商品已存在，增加数量
    if (existingItem.quantity < product.stock) {
      existingItem.quantity++
    } else {
      alert(`库存不足！最多可购买 ${product.stock} 件`)
    }
  } else {
    // 新商品，添加到购物车
    cart.items.push({
      id: product.id,
      name: product.name,
      price: product.price,
      quantity: 1,
      maxStock: product.stock,
      category: product.category
    })
  }
}

// 增加数量
function increaseQuantity(item) {
  if (item.quantity < item.maxStock) {
    item.quantity++
  } else {
    alert('已达到库存上限')
  }
}

// 减少数量
function decreaseQuantity(item) {
  if (item.quantity > 1) {
    item.quantity--
  }
}

// 从购物车移除
function removeFromCart(itemId) {
  const index = cart.items.findIndex(item => item.id === itemId)
  if (index > -1) {
    cart.items.splice(index, 1)
  }
}

// 清空购物车
function clearCart() {
  if (confirm('确定要清空购物车吗？')) {
    cart.items = []
    cart.appliedCoupon = null
    cart.couponCode = ''
  }
}

// 应用优惠券
function applyCoupon() {
  const coupon = coupons.find(c => c.code === cart.couponCode.toUpperCase())
  
  if (!coupon) {
    alert('无效的优惠券码')
    return
  }
  
  if (subtotal.value < coupon.minAmount) {
    alert(`需要满 ¥${coupon.minAmount} 才能使用此优惠券`)
    return
  }
  
  cart.appliedCoupon = coupon
  alert('优惠券应用成功！')
}

// 移除优惠券
function removeCoupon() {
  cart.appliedCoupon = null
  cart.couponCode = ''
}

// ==================== 计算属性 ====================

// 小计（优惠前）
const subtotal = computed(() => {
  return cart.items.reduce((sum, item) => {
    return sum + item.price * item.quantity
  }, 0)
})

// 总数量
const totalQuantity = computed(() => {
  return cart.items.reduce((sum, item) => sum + item.quantity, 0)
})

// 优惠金额
const discountAmount = computed(() => {
  if (!cart.appliedCoupon) return 0
  
  const coupon = cart.appliedCoupon
  if (coupon.type === 'percent') {
    return subtotal.value * (coupon.value / 100)
  } else {
    return Math.min(coupon.value, subtotal.value)
  }
})

// 运费计算
const shippingFee = computed(() => {
  if (subtotal.value >= 199) return 0  // 满199免运费
  if (cart.items.length === 0) return 0
  return 10
})

// 最终价格
const finalPrice = computed(() => {
  const price = subtotal.value - discountAmount.value + shippingFee.value
  return Math.max(0, price)
})

// 节省金额
const savedAmount = computed(() => {
  return discountAmount.value
})

// 按分类统计
const itemsByCategory = computed(() => {
  const result = {}
  cart.items.forEach(item => {
    if (!result[item.category]) {
      result[item.category] = {
        count: 0,
        total: 0
      }
    }
    result[item.category].count += item.quantity
    result[item.category].total += item.price * item.quantity
  })
  return result
})

// ==================== 侦听器：自动同步 ====================

// 监听购物车变化，模拟同步到后端
let syncTimer = null
watch(
  () => cart.items,
  (newItems) => {
    // 清除之前的定时器
    if (syncTimer) clearTimeout(syncTimer)
    
    syncStatus.isSyncing = true
    syncStatus.syncMessage = '正在同步...'
    
    // 模拟异步同步（延迟1秒）
    syncTimer = setTimeout(() => {
      // 模拟API调用
      console.log('同步购物车到后端:', {
        items: newItems,
        total: finalPrice.value,
        timestamp: new Date().toISOString()
      })
      
      syncStatus.isSyncing = false
      syncStatus.lastSyncTime = new Date().toLocaleTimeString()
      syncStatus.syncMessage = `已同步 (${syncStatus.lastSyncTime})`
      
      // 保存到本地存储
      localStorage.setItem('shopping-cart', JSON.stringify({
        items: newItems,
        coupon: cart.appliedCoupon
      }))
    }, 1000)
  },
  { deep: true }
)

// 从本地存储恢复购物车
function loadCart() {
  const saved = localStorage.getItem('shopping-cart')
  if (saved) {
    const data = JSON.parse(saved)
    cart.items = data.items || []
    cart.appliedCoupon = data.coupon || null
  }
}

// 初始化加载
loadCart()

// ==================== 模拟结算 ====================

function checkout() {
  if (cart.items.length === 0) {
    alert('购物车是空的')
    return
  }
  
  const orderInfo = {
    items: cart.items,
    subtotal: subtotal.value,
    discount: discountAmount.value,
    shipping: shippingFee.value,
    total: finalPrice.value,
    coupon: cart.appliedCoupon?.code,
    timestamp: new Date().toISOString()
  }
  
  console.log('订单信息:', orderInfo)
  alert(`订单提交成功！\n总计：¥${finalPrice.value.toFixed(2)}\n感谢您的购买！`)
  
  // 清空购物车
  cart.items = []
  cart.appliedCoupon = null
  cart.couponCode = ''
}
</script>

<template>
  <div class="shopping-cart">
    <h2>🛒 练习1：购物车系统</h2>
    
    <!-- 商品列表 -->
    <div class="products-section">
      <h3>📦 商品列表</h3>
      <div class="products-grid">
        <div v-for="product in store.products" :key="product.id" class="product-card">
          <div class="product-name">{{ product.name }}</div>
          <div class="product-info">
            <span class="price">¥{{ product.price }}</span>
            <span class="stock">库存: {{ product.stock }}</span>
          </div>
          <button @click="addToCart(product)" class="add-btn">
            加入购物车
          </button>
        </div>
      </div>
    </div>
    
    <!-- 购物车 -->
    <div class="cart-section">
      <div class="cart-header">
        <h3>🛍️ 我的购物车 ({{ totalQuantity }} 件)</h3>
        <div class="sync-status">
          <span :class="{ syncing: syncStatus.isSyncing }">
            {{ syncStatus.syncMessage || '未同步' }}
          </span>
        </div>
      </div>
      
      <!-- 购物车列表 -->
      <div v-if="cart.items.length > 0" class="cart-items">
        <div v-for="item in cart.items" :key="item.id" class="cart-item">
          <div class="item-info">
            <div class="item-name">{{ item.name }}</div>
            <div class="item-price">¥{{ item.price }}</div>
          </div>
          
          <div class="item-controls">
            <button @click="decreaseQuantity(item)" class="btn-small">-</button>
            <span class="quantity">{{ item.quantity }}</span>
            <button @click="increaseQuantity(item)" class="btn-small">+</button>
          </div>
          
          <div class="item-total">
            ¥{{ (item.price * item.quantity).toFixed(2) }}
          </div>
          
          <button @click="removeFromCart(item.id)" class="remove-btn">
            删除
          </button>
        </div>
      </div>
      
      <div v-else class="empty-cart">
        购物车是空的，快去选购商品吧！
      </div>
      
      <!-- 优惠券 -->
      <div class="coupon-section">
        <h4>🎫 优惠券</h4>
        <div v-if="!cart.appliedCoupon" class="coupon-input">
          <input 
            v-model="cart.couponCode" 
            placeholder="输入优惠券码"
            @keyup.enter="applyCoupon"
          >
          <button @click="applyCoupon">应用</button>
        </div>
        
        <div v-else class="applied-coupon">
          <span>已应用: {{ cart.appliedCoupon.code }}</span>
          <span class="coupon-desc">{{ cart.appliedCoupon.description }}</span>
          <button @click="removeCoupon" class="remove-coupon">移除</button>
        </div>
        
        <div class="coupon-hints">
          <p>可用优惠券：</p>
          <span v-for="coupon in coupons" :key="coupon.code" class="hint">
            {{ coupon.code }}: {{ coupon.description }}
          </span>
        </div>
      </div>
      
      <!-- 分类统计 -->
      <div v-if="Object.keys(itemsByCategory).length > 0" class="category-stats">
        <h4>📊 分类统计</h4>
        <div v-for="(stats, category) in itemsByCategory" :key="category" class="category-item">
          <span>{{ category }}</span>
          <span>{{ stats.count }}件 / ¥{{ stats.total.toFixed(2) }}</span>
        </div>
      </div>
      
      <!-- 价格汇总 -->
      <div class="price-summary">
        <div class="price-row">
          <span>商品小计:</span>
          <span>¥{{ subtotal.toFixed(2) }}</span>
        </div>
        
        <div v-if="discountAmount > 0" class="price-row discount">
          <span>优惠金额:</span>
          <span>-¥{{ discountAmount.toFixed(2) }}</span>
        </div>
        
        <div class="price-row">
          <span>运费:</span>
          <span>{{ shippingFee === 0 ? '免运费' : `¥${shippingFee}` }}</span>
        </div>
        
        <div class="price-row total">
          <span>总计:</span>
          <span class="final-price">¥{{ finalPrice.toFixed(2) }}</span>
        </div>
        
        <div v-if="savedAmount > 0" class="saved-info">
          💰 已节省 ¥{{ savedAmount.toFixed(2) }}
        </div>
      </div>
      
      <!-- 操作按钮 -->
      <div class="cart-actions">
        <button @click="clearCart" class="clear-btn">清空购物车</button>
        <button @click="checkout" class="checkout-btn">立即结算</button>
      </div>
    </div>
  </div>
</template>

<style scoped>
.shopping-cart {
  max-width: 1200px;
  margin: 0 auto;
  padding: 20px;
}

h2 {
  text-align: center;
  color: #42b883;
  margin-bottom: 30px;
}

/* 商品列表 */
.products-section {
  margin-bottom: 40px;
}

.products-section h3 {
  color: #333;
  margin-bottom: 20px;
}

.products-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(200px, 1fr));
  gap: 15px;
}

.product-card {
  background: white;
  padding: 15px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  transition: transform 0.3s;
}

.product-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
}

.product-name {
  font-weight: bold;
  margin-bottom: 10px;
}

.product-info {
  display: flex;
  justify-content: space-between;
  margin-bottom: 10px;
  font-size: 14px;
}

.price {
  color: #ff6b6b;
  font-weight: bold;
}

.stock {
  color: #666;
}

.add-btn {
  width: 100%;
  padding: 8px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s;
}

.add-btn:hover {
  background: #35a372;
}

/* 购物车 */
.cart-section {
  background: white;
  padding: 20px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.cart-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding-bottom: 10px;
  border-bottom: 2px solid #f0f0f0;
}

.cart-header h3 {
  margin: 0;
  color: #333;
}

.sync-status {
  font-size: 14px;
  color: #666;
}

.sync-status .syncing {
  color: #ff9800;
  animation: pulse 1s infinite;
}

@keyframes pulse {
  0%, 100% { opacity: 1; }
  50% { opacity: 0.5; }
}

/* 购物车项目 */
.cart-items {
  margin-bottom: 20px;
}

.cart-item {
  display: flex;
  align-items: center;
  gap: 15px;
  padding: 15px;
  background: #f9f9f9;
  border-radius: 4px;
  margin-bottom: 10px;
}

.item-info {
  flex: 1;
}

.item-name {
  font-weight: bold;
  margin-bottom: 5px;
}

.item-price {
  color: #666;
  font-size: 14px;
}

.item-controls {
  display: flex;
  align-items: center;
  gap: 10px;
}

.btn-small {
  width: 30px;
  height: 30px;
  border: 1px solid #ddd;
  background: white;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s;
}

.btn-small:hover {
  background: #f0f0f0;
}

.quantity {
  min-width: 30px;
  text-align: center;
  font-weight: bold;
}

.item-total {
  min-width: 80px;
  text-align: right;
  font-weight: bold;
  color: #ff6b6b;
}

.remove-btn {
  padding: 6px 12px;
  background: #ff6b6b;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s;
}

.remove-btn:hover {
  background: #ff5252;
}

.empty-cart {
  text-align: center;
  padding: 40px;
  color: #999;
}

/* 优惠券 */
.coupon-section {
  padding: 15px;
  background: #f5f5f5;
  border-radius: 4px;
  margin-bottom: 20px;
}

.coupon-section h4 {
  margin-top: 0;
  color: #333;
}

.coupon-input {
  display: flex;
  gap: 10px;
  margin-bottom: 10px;
}

.coupon-input input {
  flex: 1;
  padding: 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.coupon-input button {
  padding: 8px 20px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.applied-coupon {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 10px;
  background: #e8f5e9;
  border-radius: 4px;
  margin-bottom: 10px;
}

.coupon-desc {
  flex: 1;
  color: #666;
  font-size: 14px;
}

.remove-coupon {
  padding: 4px 10px;
  background: #ff9800;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
}

.coupon-hints {
  font-size: 12px;
  color: #666;
}

.coupon-hints p {
  margin: 5px 0;
}

.hint {
  display: inline-block;
  margin-right: 10px;
  padding: 2px 6px;
  background: white;
  border-radius: 3px;
}

/* 分类统计 */
.category-stats {
  padding: 15px;
  background: #f9f9f9;
  border-radius: 4px;
  margin-bottom: 20px;
}

.category-stats h4 {
  margin-top: 0;
  color: #333;
}

.category-item {
  display: flex;
  justify-content: space-between;
  padding: 5px 0;
  font-size: 14px;
}

/* 价格汇总 */
.price-summary {
  padding: 20px 0;
  border-top: 2px solid #f0f0f0;
  margin-bottom: 20px;
}

.price-row {
  display: flex;
  justify-content: space-between;
  padding: 8px 0;
  font-size: 16px;
}

.price-row.discount {
  color: #4caf50;
}

.price-row.total {
  font-size: 20px;
  font-weight: bold;
  padding-top: 15px;
  border-top: 1px solid #ddd;
  margin-top: 10px;
}

.final-price {
  color: #ff6b6b;
  font-size: 24px;
}

.saved-info {
  text-align: center;
  padding: 10px;
  background: #fff3cd;
  border-radius: 4px;
  color: #856404;
  margin-top: 10px;
}

/* 操作按钮 */
.cart-actions {
  display: flex;
  gap: 15px;
}

.clear-btn {
  flex: 1;
  padding: 12px;
  background: #f5f5f5;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  transition: background 0.3s;
}

.clear-btn:hover {
  background: #e0e0e0;
}

.checkout-btn {
  flex: 2;
  padding: 12px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 16px;
  font-weight: bold;
  transition: background 0.3s;
}

.checkout-btn:hover {
  background: #35a372;
}

@media (max-width: 768px) {
  .products-grid {
    grid-template-columns: repeat(auto-fill, minmax(150px, 1fr));
  }
  
  .cart-item {
    flex-direction: column;
    align-items: flex-start;
  }
  
  .item-controls {
    margin: 10px 0;
  }
}
</style>