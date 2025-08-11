<script setup>
// Day 2 练习3：表单验证
import { reactive, computed, watch, ref } from 'vue'

// ==================== 表单数据 ====================

const formData = reactive({
  username: '',
  email: '',
  password: '',
  confirmPassword: '',
  phone: '',
  age: '',
  gender: '',
  country: '',
  bio: '',
  agree: false,
  newsletter: false
})

// 表单状态
const formState = reactive({
  isSubmitting: false,
  submitSuccess: false,
  submitError: '',
  touchedFields: new Set(),  // 已触碰的字段
  focusedField: null,  // 当前聚焦的字段
  showPassword: false,  // 显示密码
  showConfirmPassword: false
})

// ==================== 验证规则 ====================

const validationRules = {
  username: {
    required: true,
    minLength: 3,
    maxLength: 20,
    pattern: /^[a-zA-Z0-9_]+$/,
    messages: {
      required: '用户名不能为空',
      minLength: '用户名至少需要3个字符',
      maxLength: '用户名不能超过20个字符',
      pattern: '用户名只能包含字母、数字和下划线'
    }
  },
  email: {
    required: true,
    pattern: /^[^\s@]+@[^\s@]+\.[^\s@]+$/,
    messages: {
      required: '邮箱不能为空',
      pattern: '请输入有效的邮箱地址'
    }
  },
  password: {
    required: true,
    minLength: 8,
    maxLength: 30,
    strongPassword: true,
    messages: {
      required: '密码不能为空',
      minLength: '密码至少需要8个字符',
      maxLength: '密码不能超过30个字符',
      strongPassword: '密码必须包含大小写字母、数字和特殊字符'
    }
  },
  confirmPassword: {
    required: true,
    match: 'password',
    messages: {
      required: '请确认密码',
      match: '两次输入的密码不一致'
    }
  },
  phone: {
    required: true,
    pattern: /^1[3-9]\d{9}$/,
    messages: {
      required: '手机号不能为空',
      pattern: '请输入有效的11位手机号'
    }
  },
  age: {
    required: true,
    min: 18,
    max: 120,
    messages: {
      required: '年龄不能为空',
      min: '年龄必须大于等于18岁',
      max: '年龄不能超过120岁'
    }
  },
  gender: {
    required: true,
    messages: {
      required: '请选择性别'
    }
  },
  country: {
    required: true,
    messages: {
      required: '请选择国家'
    }
  },
  bio: {
    maxLength: 200,
    messages: {
      maxLength: '个人简介不能超过200个字符'
    }
  },
  agree: {
    required: true,
    mustBeTrue: true,
    messages: {
      required: '请同意服务条款',
      mustBeTrue: '必须同意服务条款才能注册'
    }
  }
}

// ==================== 验证函数 ====================

// 验证单个字段
function validateField(field, value) {
  const rules = validationRules[field]
  if (!rules) return null
  
  const errors = []
  
  // 必填验证
  if (rules.required && !value && value !== 0) {
    errors.push(rules.messages.required)
  }
  
  // 最小长度
  if (rules.minLength && value && value.length < rules.minLength) {
    errors.push(rules.messages.minLength)
  }
  
  // 最大长度
  if (rules.maxLength && value && value.length > rules.maxLength) {
    errors.push(rules.messages.maxLength)
  }
  
  // 最小值
  if (rules.min && value && parseFloat(value) < rules.min) {
    errors.push(rules.messages.min)
  }
  
  // 最大值
  if (rules.max && value && parseFloat(value) > rules.max) {
    errors.push(rules.messages.max)
  }
  
  // 正则验证
  if (rules.pattern && value && !rules.pattern.test(value)) {
    errors.push(rules.messages.pattern)
  }
  
  // 强密码验证
  if (rules.strongPassword && value) {
    const hasUpperCase = /[A-Z]/.test(value)
    const hasLowerCase = /[a-z]/.test(value)
    const hasNumber = /\d/.test(value)
    const hasSpecialChar = /[!@#$%^&*(),.?":{}|<>]/.test(value)
    
    if (!hasUpperCase || !hasLowerCase || !hasNumber || !hasSpecialChar) {
      errors.push(rules.messages.strongPassword)
    }
  }
  
  // 匹配验证
  if (rules.match && value !== formData[rules.match]) {
    errors.push(rules.messages.match)
  }
  
  // 布尔值必须为真
  if (rules.mustBeTrue && value !== true) {
    errors.push(rules.messages.mustBeTrue)
  }
  
  return errors.length > 0 ? errors : null
}

// ==================== 计算属性：实时验证 ====================

// 验证错误
const validationErrors = computed(() => {
  const errors = {}
  
  // 只验证已触碰的字段
  for (const field of formState.touchedFields) {
    const fieldErrors = validateField(field, formData[field])
    if (fieldErrors) {
      errors[field] = fieldErrors
    }
  }
  
  return errors
})

// 表单是否有效
const isFormValid = computed(() => {
  // 验证所有字段
  for (const field in validationRules) {
    const errors = validateField(field, formData[field])
    if (errors && errors.length > 0) {
      return false
    }
  }
  return true
})

// 密码强度
const passwordStrength = computed(() => {
  const password = formData.password
  if (!password) return { level: 0, text: '', color: '' }
  
  let strength = 0
  
  // 长度
  if (password.length >= 8) strength++
  if (password.length >= 12) strength++
  
  // 字符类型
  if (/[a-z]/.test(password)) strength++
  if (/[A-Z]/.test(password)) strength++
  if (/\d/.test(password)) strength++
  if (/[!@#$%^&*(),.?":{}|<>]/.test(password)) strength++
  
  const levels = [
    { level: 0, text: '', color: '' },
    { level: 1, text: '弱', color: '#f44336' },
    { level: 2, text: '弱', color: '#ff9800' },
    { level: 3, text: '中', color: '#ffeb3b' },
    { level: 4, text: '中', color: '#8bc34a' },
    { level: 5, text: '强', color: '#4caf50' },
    { level: 6, text: '很强', color: '#2196f3' }
  ]
  
  return levels[Math.min(strength, 6)]
})

// 表单完成度
const formProgress = computed(() => {
  const requiredFields = Object.keys(validationRules).filter(
    field => validationRules[field].required
  )
  const filledFields = requiredFields.filter(field => {
    const value = formData[field]
    return value !== '' && value !== false && value !== null
  })
  
  return Math.round((filledFields.length / requiredFields.length) * 100)
})

// ==================== 侦听器：字段联动 ====================

// 监听密码变化，清除确认密码的错误
watch(() => formData.password, () => {
  if (formState.touchedFields.has('confirmPassword') && formData.confirmPassword) {
    // 重新验证确认密码
    formState.touchedFields.add('confirmPassword')
  }
})

// 监听邮箱变化，自动建议用户名
watch(() => formData.email, (email) => {
  if (email && !formData.username) {
    // 从邮箱提取用户名建议
    const username = email.split('@')[0].replace(/[^a-zA-Z0-9_]/g, '_')
    if (username.length >= 3 && username.length <= 20) {
      formData.username = username
    }
  }
})

// ==================== 表单操作 ====================

// 标记字段为已触碰
function markAsTouched(field) {
  formState.touchedFields.add(field)
}

// 设置聚焦字段
function setFocusedField(field) {
  formState.focusedField = field
}

// 清除聚焦字段
function clearFocusedField() {
  formState.focusedField = null
}

// 切换密码显示
function togglePasswordVisibility(field) {
  if (field === 'password') {
    formState.showPassword = !formState.showPassword
  } else {
    formState.showConfirmPassword = !formState.showConfirmPassword
  }
}

// 提交表单
async function submitForm() {
  // 标记所有字段为已触碰
  Object.keys(validationRules).forEach(field => {
    formState.touchedFields.add(field)
  })
  
  // 检查表单是否有效
  if (!isFormValid.value) {
    formState.submitError = '请修正表单中的错误后再提交'
    return
  }
  
  formState.isSubmitting = true
  formState.submitError = ''
  
  try {
    // 模拟API调用
    await new Promise(resolve => setTimeout(resolve, 2000))
    
    // 模拟随机错误（10%概率）
    if (Math.random() < 0.1) {
      throw new Error('服务器错误，请稍后重试')
    }
    
    console.log('提交的表单数据:', formData)
    formState.submitSuccess = true
    
    // 3秒后重置成功状态
    setTimeout(() => {
      formState.submitSuccess = false
    }, 3000)
    
  } catch (error) {
    formState.submitError = error.message
  } finally {
    formState.isSubmitting = false
  }
}

// 重置表单
function resetForm() {
  // 重置数据
  Object.keys(formData).forEach(key => {
    if (typeof formData[key] === 'boolean') {
      formData[key] = false
    } else {
      formData[key] = ''
    }
  })
  
  // 重置状态
  formState.touchedFields.clear()
  formState.submitSuccess = false
  formState.submitError = ''
}

// 自动填充（用于测试）
function autoFill() {
  formData.username = 'test_user123'
  formData.email = 'test@example.com'
  formData.password = 'Test123!@#'
  formData.confirmPassword = 'Test123!@#'
  formData.phone = '13800138000'
  formData.age = '25'
  formData.gender = 'male'
  formData.country = 'China'
  formData.bio = '这是一个测试账号的个人简介。'
  formData.agree = true
  formData.newsletter = true
}

// 国家列表
const countries = [
  'China', 'United States', 'United Kingdom', 'Japan', 
  'Germany', 'France', 'Canada', 'Australia', 'Others'
]
</script>

<template>
  <div class="form-validation">
    <h2>📝 练习3：表单验证</h2>
    
    <!-- 表单进度 -->
    <div class="form-progress">
      <div class="progress-bar">
        <div class="progress-fill" :style="{ width: formProgress + '%' }"></div>
      </div>
      <span class="progress-text">表单完成度: {{ formProgress }}%</span>
    </div>
    
    <!-- 快速操作 -->
    <div class="quick-actions">
      <button @click="autoFill" class="auto-fill-btn">🚀 自动填充（测试）</button>
      <button @click="resetForm" class="reset-btn">🔄 重置表单</button>
    </div>
    
    <!-- 表单 -->
    <form @submit.prevent="submitForm" class="registration-form">
      <!-- 用户名 -->
      <div class="form-group">
        <label for="username">
          用户名 <span class="required">*</span>
        </label>
        <input 
          id="username"
          v-model="formData.username"
          @blur="markAsTouched('username')"
          @focus="setFocusedField('username')"
          type="text"
          :class="{ error: validationErrors.username }"
          placeholder="3-20个字符，字母数字下划线"
        >
        <div v-if="validationErrors.username" class="error-messages">
          <span v-for="error in validationErrors.username" :key="error" class="error-msg">
            {{ error }}
          </span>
        </div>
        <div v-if="formState.focusedField === 'username'" class="field-hint">
          💡 用户名将作为您的唯一标识
        </div>
      </div>
      
      <!-- 邮箱 -->
      <div class="form-group">
        <label for="email">
          邮箱 <span class="required">*</span>
        </label>
        <input 
          id="email"
          v-model="formData.email"
          @blur="markAsTouched('email')"
          @focus="setFocusedField('email')"
          type="email"
          :class="{ error: validationErrors.email }"
          placeholder="example@domain.com"
        >
        <div v-if="validationErrors.email" class="error-messages">
          <span v-for="error in validationErrors.email" :key="error" class="error-msg">
            {{ error }}
          </span>
        </div>
        <div v-if="formState.focusedField === 'email'" class="field-hint">
          💡 邮箱将用于账号恢复和通知
        </div>
      </div>
      
      <!-- 密码 -->
      <div class="form-group">
        <label for="password">
          密码 <span class="required">*</span>
        </label>
        <div class="password-input">
          <input 
            id="password"
            v-model="formData.password"
            @blur="markAsTouched('password')"
            @focus="setFocusedField('password')"
            :type="formState.showPassword ? 'text' : 'password'"
            :class="{ error: validationErrors.password }"
            placeholder="至少8位，包含大小写字母、数字和特殊字符"
          >
          <button 
            type="button"
            @click="togglePasswordVisibility('password')"
            class="toggle-password"
          >
            {{ formState.showPassword ? '👁️' : '👁️‍🗨️' }}
          </button>
        </div>
        <div v-if="formData.password" class="password-strength">
          <div class="strength-bar">
            <div 
              class="strength-fill" 
              :style="{ 
                width: (passwordStrength.level / 6 * 100) + '%',
                background: passwordStrength.color
              }"
            ></div>
          </div>
          <span class="strength-text" :style="{ color: passwordStrength.color }">
            密码强度: {{ passwordStrength.text }}
          </span>
        </div>
        <div v-if="validationErrors.password" class="error-messages">
          <span v-for="error in validationErrors.password" :key="error" class="error-msg">
            {{ error }}
          </span>
        </div>
      </div>
      
      <!-- 确认密码 -->
      <div class="form-group">
        <label for="confirmPassword">
          确认密码 <span class="required">*</span>
        </label>
        <div class="password-input">
          <input 
            id="confirmPassword"
            v-model="formData.confirmPassword"
            @blur="markAsTouched('confirmPassword')"
            @focus="setFocusedField('confirmPassword')"
            :type="formState.showConfirmPassword ? 'text' : 'password'"
            :class="{ error: validationErrors.confirmPassword }"
            placeholder="再次输入密码"
          >
          <button 
            type="button"
            @click="togglePasswordVisibility('confirmPassword')"
            class="toggle-password"
          >
            {{ formState.showConfirmPassword ? '👁️' : '👁️‍🗨️' }}
          </button>
        </div>
        <div v-if="validationErrors.confirmPassword" class="error-messages">
          <span v-for="error in validationErrors.confirmPassword" :key="error" class="error-msg">
            {{ error }}
          </span>
        </div>
      </div>
      
      <!-- 手机号 -->
      <div class="form-group">
        <label for="phone">
          手机号 <span class="required">*</span>
        </label>
        <input 
          id="phone"
          v-model="formData.phone"
          @blur="markAsTouched('phone')"
          @focus="setFocusedField('phone')"
          type="tel"
          :class="{ error: validationErrors.phone }"
          placeholder="11位手机号"
        >
        <div v-if="validationErrors.phone" class="error-messages">
          <span v-for="error in validationErrors.phone" :key="error" class="error-msg">
            {{ error }}
          </span>
        </div>
      </div>
      
      <!-- 年龄和性别 -->
      <div class="form-row">
        <div class="form-group">
          <label for="age">
            年龄 <span class="required">*</span>
          </label>
          <input 
            id="age"
            v-model.number="formData.age"
            @blur="markAsTouched('age')"
            @focus="setFocusedField('age')"
            type="number"
            min="18"
            max="120"
            :class="{ error: validationErrors.age }"
            placeholder="18-120"
          >
          <div v-if="validationErrors.age" class="error-messages">
            <span v-for="error in validationErrors.age" :key="error" class="error-msg">
              {{ error }}
            </span>
          </div>
        </div>
        
        <div class="form-group">
          <label for="gender">
            性别 <span class="required">*</span>
          </label>
          <select 
            id="gender"
            v-model="formData.gender"
            @blur="markAsTouched('gender')"
            @focus="setFocusedField('gender')"
            :class="{ error: validationErrors.gender }"
          >
            <option value="">请选择</option>
            <option value="male">男</option>
            <option value="female">女</option>
            <option value="other">其他</option>
          </select>
          <div v-if="validationErrors.gender" class="error-messages">
            <span v-for="error in validationErrors.gender" :key="error" class="error-msg">
              {{ error }}
            </span>
          </div>
        </div>
      </div>
      
      <!-- 国家 -->
      <div class="form-group">
        <label for="country">
          国家/地区 <span class="required">*</span>
        </label>
        <select 
          id="country"
          v-model="formData.country"
          @blur="markAsTouched('country')"
          @focus="setFocusedField('country')"
          :class="{ error: validationErrors.country }"
        >
          <option value="">请选择</option>
          <option v-for="country in countries" :key="country" :value="country">
            {{ country }}
          </option>
        </select>
        <div v-if="validationErrors.country" class="error-messages">
          <span v-for="error in validationErrors.country" :key="error" class="error-msg">
            {{ error }}
          </span>
        </div>
      </div>
      
      <!-- 个人简介 -->
      <div class="form-group">
        <label for="bio">
          个人简介
          <span class="optional">（可选）</span>
        </label>
        <textarea 
          id="bio"
          v-model="formData.bio"
          @blur="markAsTouched('bio')"
          @focus="setFocusedField('bio')"
          :class="{ error: validationErrors.bio }"
          placeholder="简单介绍一下自己..."
          rows="4"
        ></textarea>
        <div class="char-count">
          {{ formData.bio.length }}/200
        </div>
        <div v-if="validationErrors.bio" class="error-messages">
          <span v-for="error in validationErrors.bio" :key="error" class="error-msg">
            {{ error }}
          </span>
        </div>
      </div>
      
      <!-- 复选框 -->
      <div class="form-group checkbox-group">
        <label>
          <input 
            v-model="formData.newsletter"
            type="checkbox"
          >
          订阅新闻通讯
        </label>
      </div>
      
      <div class="form-group checkbox-group">
        <label>
          <input 
            v-model="formData.agree"
            @change="markAsTouched('agree')"
            type="checkbox"
          >
          我同意 <a href="#" @click.prevent>服务条款</a> 和 <a href="#" @click.prevent>隐私政策</a>
          <span class="required">*</span>
        </label>
        <div v-if="validationErrors.agree" class="error-messages">
          <span v-for="error in validationErrors.agree" :key="error" class="error-msg">
            {{ error }}
          </span>
        </div>
      </div>
      
      <!-- 提交错误 -->
      <div v-if="formState.submitError" class="submit-error">
        ❌ {{ formState.submitError }}
      </div>
      
      <!-- 提交成功 -->
      <div v-if="formState.submitSuccess" class="submit-success">
        ✅ 注册成功！欢迎加入我们！
      </div>
      
      <!-- 提交按钮 -->
      <div class="form-actions">
        <button 
          type="submit"
          :disabled="!isFormValid || formState.isSubmitting"
          class="submit-btn"
        >
          <span v-if="formState.isSubmitting">提交中...</span>
          <span v-else>立即注册</span>
        </button>
      </div>
      
      <!-- 调试信息 -->
      <details class="debug-info">
        <summary>🐛 调试信息</summary>
        <pre>{{ JSON.stringify({
          formData,
          validationErrors,
          isFormValid,
          formProgress,
          touchedFields: Array.from(formState.touchedFields)
        }, null, 2) }}</pre>
      </details>
    </form>
  </div>
</template>

<style scoped>
.form-validation {
  max-width: 600px;
  margin: 0 auto;
  padding: 20px;
}

h2 {
  text-align: center;
  color: #42b883;
  margin-bottom: 30px;
}

/* 表单进度 */
.form-progress {
  margin-bottom: 20px;
}

.progress-bar {
  height: 8px;
  background: #e0e0e0;
  border-radius: 4px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: linear-gradient(90deg, #42b883, #35a372);
  transition: width 0.3s;
}

.progress-text {
  display: block;
  text-align: center;
  margin-top: 10px;
  font-size: 14px;
  color: #666;
}

/* 快速操作 */
.quick-actions {
  display: flex;
  gap: 10px;
  margin-bottom: 20px;
}

.auto-fill-btn,
.reset-btn {
  flex: 1;
  padding: 10px;
  border: 1px solid #ddd;
  background: white;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s;
}

.auto-fill-btn:hover {
  background: #42b883;
  color: white;
  border-color: #42b883;
}

.reset-btn:hover {
  background: #ff6b6b;
  color: white;
  border-color: #ff6b6b;
}

/* 表单 */
.registration-form {
  background: white;
  padding: 30px;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
}

.form-group {
  margin-bottom: 20px;
}

.form-row {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 20px;
}

label {
  display: block;
  margin-bottom: 8px;
  font-weight: 500;
  color: #333;
}

.required {
  color: #f44336;
}

.optional {
  color: #999;
  font-size: 12px;
}

input[type="text"],
input[type="email"],
input[type="password"],
input[type="tel"],
input[type="number"],
select,
textarea {
  width: 100%;
  padding: 10px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.3s;
}

input:focus,
select:focus,
textarea:focus {
  outline: none;
  border-color: #42b883;
}

input.error,
select.error,
textarea.error {
  border-color: #f44336;
}

/* 密码输入 */
.password-input {
  position: relative;
}

.toggle-password {
  position: absolute;
  right: 10px;
  top: 50%;
  transform: translateY(-50%);
  background: none;
  border: none;
  cursor: pointer;
  font-size: 18px;
  padding: 5px;
}

/* 密码强度 */
.password-strength {
  margin-top: 10px;
}

.strength-bar {
  height: 4px;
  background: #e0e0e0;
  border-radius: 2px;
  overflow: hidden;
  margin-bottom: 5px;
}

.strength-fill {
  height: 100%;
  transition: all 0.3s;
}

.strength-text {
  font-size: 12px;
  font-weight: bold;
}

/* 字符计数 */
.char-count {
  text-align: right;
  font-size: 12px;
  color: #666;
  margin-top: 5px;
}

/* 复选框 */
.checkbox-group label {
  display: flex;
  align-items: center;
  cursor: pointer;
}

.checkbox-group input[type="checkbox"] {
  width: auto;
  margin-right: 8px;
}

.checkbox-group a {
  color: #42b883;
  text-decoration: none;
  margin: 0 4px;
}

.checkbox-group a:hover {
  text-decoration: underline;
}

/* 错误信息 */
.error-messages {
  margin-top: 5px;
}

.error-msg {
  display: block;
  color: #f44336;
  font-size: 12px;
  margin-top: 2px;
}

/* 字段提示 */
.field-hint {
  margin-top: 5px;
  padding: 8px;
  background: #e3f2fd;
  border-radius: 4px;
  font-size: 12px;
  color: #1976d2;
}

/* 提交状态 */
.submit-error {
  padding: 12px;
  background: #ffebee;
  border: 1px solid #ffcdd2;
  border-radius: 4px;
  color: #c62828;
  margin-bottom: 20px;
}

.submit-success {
  padding: 12px;
  background: #e8f5e9;
  border: 1px solid #c8e6c9;
  border-radius: 4px;
  color: #2e7d32;
  margin-bottom: 20px;
  animation: slideDown 0.3s;
}

@keyframes slideDown {
  from {
    opacity: 0;
    transform: translateY(-10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

/* 表单操作 */
.form-actions {
  margin-top: 30px;
}

.submit-btn {
  width: 100%;
  padding: 14px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 4px;
  font-size: 16px;
  font-weight: bold;
  cursor: pointer;
  transition: background 0.3s;
}

.submit-btn:hover:not(:disabled) {
  background: #35a372;
}

.submit-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

/* 调试信息 */
.debug-info {
  margin-top: 30px;
  padding: 15px;
  background: #f5f5f5;
  border-radius: 4px;
}

.debug-info summary {
  cursor: pointer;
  font-weight: bold;
  color: #666;
}

.debug-info pre {
  margin-top: 10px;
  padding: 10px;
  background: white;
  border-radius: 4px;
  font-size: 12px;
  overflow-x: auto;
}

@media (max-width: 600px) {
  .form-row {
    grid-template-columns: 1fr;
  }
  
  .quick-actions {
    flex-direction: column;
  }
  
  .registration-form {
    padding: 20px;
  }
}
</style>