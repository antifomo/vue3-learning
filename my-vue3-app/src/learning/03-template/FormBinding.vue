<script setup>
// Day 3 - 表单双向绑定详解
import { ref, reactive, computed, watch } from 'vue'

// ==================== 基础表单数据 ====================

// 文本输入
const textInputs = reactive({
  basicText: '',
  email: '',
  password: '',
  url: '',
  number: 0,
  search: '',
  tel: ''
})

// 多行文本
const textareaData = reactive({
  message: '',
  description: '',
  comments: ''
})

// 复选框
const checkboxData = reactive({
  // 单个复选框
  agreeTerms: false,
  newsletter: true,
  notifications: false,
  
  // 多个复选框数组
  hobbies: [],
  skills: [],
  languages: []
})

// 单选按钮
const radioData = reactive({
  gender: '',
  experience: '',
  size: '',
  paymentMethod: ''
})

// 下拉选择
const selectData = reactive({
  country: '',
  city: '',
  category: '',
  priority: '',
  
  // 多选下拉
  tags: [],
  features: []
})

// 文件上传
const fileData = reactive({
  avatar: null,
  documents: [],
  images: []
})

// 范围输入
const rangeData = reactive({
  volume: 50,
  brightness: 75,
  price: 100,
  ageRange: [18, 65]
})

// 日期时间
const dateTimeData = reactive({
  birthday: '',
  appointmentDate: '',
  appointmentTime: '',
  eventDateTime: '',
  startWeek: '',
  startMonth: ''
})

// ==================== 高级表单示例 ====================

// 用户注册表单
const registrationForm = reactive({
  // 个人信息
  firstName: '',
  lastName: '',
  email: '',
  password: '',
  confirmPassword: '',
  birthDate: '',
  gender: '',
  phone: '',
  
  // 地址信息
  country: '',
  state: '',
  city: '',
  address: '',
  zipCode: '',
  
  // 偏好设置
  interests: [],
  newsletter: false,
  smsNotifications: true,
  emailNotifications: true,
  
  // 账户设置
  username: '',
  securityQuestion: '',
  securityAnswer: '',
  
  // 协议
  agreeTerms: false,
  agreePrivacy: false
})

// 动态表单
const dynamicForm = reactive({
  fields: [
    { id: 1, type: 'text', label: '姓名', value: '', required: true },
    { id: 2, type: 'email', label: '邮箱', value: '', required: true },
    { id: 3, type: 'select', label: '部门', value: '', options: ['技术部', '销售部', '人事部'], required: false }
  ],
  nextId: 4
})

// 表单验证状态
const validation = reactive({
  errors: {},
  touched: {},
  isSubmitting: false
})

// ==================== 修饰符演示 ====================

// .lazy 修饰符
const lazyDemo = reactive({
  normalInput: '',
  lazyInput: '',
  normalCount: 0,
  lazyCount: 0
})

// .number 修饰符
const numberDemo = reactive({
  stringInput: '',
  numberInput: 0,
  priceInput: 0
})

// .trim 修饰符
const trimDemo = reactive({
  normalInput: '',
  trimmedInput: ''
})

// ==================== 选项数据 ====================

const options = {
  hobbies: [
    { value: 'reading', label: '📚 阅读' },
    { value: 'music', label: '🎵 音乐' },
    { value: 'sports', label: '⚽ 运动' },
    { value: 'travel', label: '✈️ 旅行' },
    { value: 'cooking', label: '🍳 烹饪' },
    { value: 'gaming', label: '🎮 游戏' }
  ],
  
  skills: [
    { value: 'javascript', label: 'JavaScript' },
    { value: 'vue', label: 'Vue.js' },
    { value: 'react', label: 'React' },
    { value: 'node', label: 'Node.js' },
    { value: 'python', label: 'Python' },
    { value: 'java', label: 'Java' }
  ],
  
  languages: [
    { value: 'zh', label: '中文' },
    { value: 'en', label: 'English' },
    { value: 'ja', label: '日本語' },
    { value: 'ko', label: '한국어' },
    { value: 'fr', label: 'Français' },
    { value: 'de', label: 'Deutsch' }
  ],
  
  countries: [
    { value: 'cn', label: '中国' },
    { value: 'us', label: '美国' },
    { value: 'jp', label: '日本' },
    { value: 'kr', label: '韩国' },
    { value: 'uk', label: '英国' },
    { value: 'fr', label: '法国' },
    { value: 'de', label: '德国' }
  ],
  
  tags: [
    { value: 'urgent', label: '紧急' },
    { value: 'important', label: '重要' },
    { value: 'bug', label: 'Bug' },
    { value: 'feature', label: '新功能' },
    { value: 'improvement', label: '改进' },
    { value: 'documentation', label: '文档' }
  ]
}

// ==================== 计算属性 ====================

// 表单完成度
const formCompletion = computed(() => {
  const totalFields = Object.keys(registrationForm).length
  const filledFields = Object.values(registrationForm).filter(value => {
    if (Array.isArray(value)) return value.length > 0
    if (typeof value === 'boolean') return true
    return value !== '' && value !== null && value !== undefined
  }).length
  
  return Math.round((filledFields / totalFields) * 100)
})

// 表单验证状态
const isFormValid = computed(() => {
  return Object.keys(validation.errors).length === 0 && 
         registrationForm.agreeTerms &&
         registrationForm.email &&
         registrationForm.password &&
         registrationForm.password === registrationForm.confirmPassword
})

// 动态字段类型选项
const fieldTypeOptions = [
  { value: 'text', label: '文本' },
  { value: 'email', label: '邮箱' },
  { value: 'number', label: '数字' },
  { value: 'password', label: '密码' },
  { value: 'textarea', label: '多行文本' },
  { value: 'select', label: '下拉选择' },
  { value: 'checkbox', label: '复选框' },
  { value: 'radio', label: '单选按钮' },
  { value: 'date', label: '日期' },
  { value: 'file', label: '文件' }
]

// ==================== 方法 ====================

// 处理文件上传
function handleFileUpload(event, field) {
  const files = Array.from(event.target.files)
  
  if (field === 'avatar') {
    fileData.avatar = files[0] || null
  } else if (field === 'documents') {
    fileData.documents = files
  } else if (field === 'images') {
    fileData.images = files
  }
}

// 移除文件
function removeFile(field, index = null) {
  if (field === 'avatar') {
    fileData.avatar = null
  } else if (index !== null) {
    fileData[field].splice(index, 1)
  }
}

// 表单验证
function validateField(field, value) {
  const errors = {}
  
  switch (field) {
    case 'email':
      if (!value) {
        errors.email = '邮箱不能为空'
      } else if (!/^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)) {
        errors.email = '邮箱格式不正确'
      }
      break
      
    case 'password':
      if (!value) {
        errors.password = '密码不能为空'
      } else if (value.length < 6) {
        errors.password = '密码至少6个字符'
      }
      break
      
    case 'confirmPassword':
      if (value !== registrationForm.password) {
        errors.confirmPassword = '两次输入的密码不一致'
      }
      break
      
    case 'phone':
      if (value && !/^1[3-9]\d{9}$/.test(value)) {
        errors.phone = '请输入正确的手机号'
      }
      break
      
    case 'zipCode':
      if (value && !/^\d{6}$/.test(value)) {
        errors.zipCode = '请输入6位邮政编码'
      }
      break
      
    case 'username':
      if (!value) {
        errors.username = '用户名不能为空'
      } else if (value.length < 3) {
        errors.username = '用户名至少3个字符'
      } else if (!/^[a-zA-Z0-9_]+$/.test(value)) {
        errors.username = '用户名只能包含字母、数字和下划线'
      }
      break
  }
  
  // 更新验证状态
  if (Object.keys(errors).length > 0) {
    Object.assign(validation.errors, errors)
  } else {
    delete validation.errors[field]
  }
  
  validation.touched[field] = true
}

// 提交表单
function submitForm() {
  validation.isSubmitting = true
  
  // 验证所有必填字段
  const requiredFields = ['email', 'password', 'confirmPassword', 'username']
  requiredFields.forEach(field => {
    validateField(field, registrationForm[field])
  })
  
  setTimeout(() => {
    validation.isSubmitting = false
    
    if (isFormValid.value) {
      alert('表单提交成功！')
      console.log('提交的数据:', registrationForm)
    } else {
      alert('请检查表单错误')
    }
  }, 2000)
}

// 重置表单
function resetForm() {
  Object.keys(registrationForm).forEach(key => {
    if (Array.isArray(registrationForm[key])) {
      registrationForm[key] = []
    } else if (typeof registrationForm[key] === 'boolean') {
      registrationForm[key] = false
    } else {
      registrationForm[key] = ''
    }
  })
  
  validation.errors = {}
  validation.touched = {}
}

// 动态表单操作
function addField() {
  dynamicForm.fields.push({
    id: dynamicForm.nextId++,
    type: 'text',
    label: '新字段',
    value: '',
    required: false,
    options: []
  })
}

function removeField(id) {
  const index = dynamicForm.fields.findIndex(field => field.id === id)
  if (index > -1) {
    dynamicForm.fields.splice(index, 1)
  }
}

function addOption(fieldIndex) {
  if (!dynamicForm.fields[fieldIndex].options) {
    dynamicForm.fields[fieldIndex].options = []
  }
  dynamicForm.fields[fieldIndex].options.push('新选项')
}

function removeOption(fieldIndex, optionIndex) {
  dynamicForm.fields[fieldIndex].options.splice(optionIndex, 1)
}

// 修饰符演示方法
function updateNormalCount() {
  lazyDemo.normalCount++
}

function updateLazyCount() {
  lazyDemo.lazyCount++
}

// 监听器设置
watch(() => lazyDemo.normalInput, updateNormalCount)
watch(() => lazyDemo.lazyInput, updateLazyCount)

// 预设数据
function loadSampleData() {
  Object.assign(registrationForm, {
    firstName: '张',
    lastName: '三',
    email: 'zhangsan@example.com',
    username: 'zhangsan123',
    birthDate: '1995-05-15',
    gender: 'male',
    phone: '13800138000',
    country: 'cn',
    state: '北京市',
    city: '北京',
    address: '朝阳区某某街道123号',
    zipCode: '100000',
    interests: ['reading', 'music', 'travel'],
    newsletter: true,
    agreeTerms: true,
    agreePrivacy: true
  })
}
</script>

<template>
  <div class="form-binding">
    <h2>📝 表单双向绑定详解</h2>
    
    <!-- 基础表单控件 -->
    <div class="section">
      <h3>1. 基础表单控件</h3>
      <div class="card">
        <div class="demo-group">
          <h4>文本输入框</h4>
          
          <div class="input-grid">
            <div class="input-demo">
              <label>基础文本:</label>
              <input v-model="textInputs.basicText" type="text" placeholder="输入文本">
              <span class="value-display">值: "{{ textInputs.basicText }}"</span>
            </div>
            
            <div class="input-demo">
              <label>邮箱:</label>
              <input v-model="textInputs.email" type="email" placeholder="输入邮箱">
              <span class="value-display">值: "{{ textInputs.email }}"</span>
            </div>
            
            <div class="input-demo">
              <label>密码:</label>
              <input v-model="textInputs.password" type="password" placeholder="输入密码">
              <span class="value-display">长度: {{ textInputs.password.length }}</span>
            </div>
            
            <div class="input-demo">
              <label>数字:</label>
              <input v-model.number="textInputs.number" type="number" placeholder="输入数字">
              <span class="value-display">值: {{ textInputs.number }} (类型: {{ typeof textInputs.number }})</span>
            </div>
            
            <div class="input-demo">
              <label>搜索:</label>
              <input v-model="textInputs.search" type="search" placeholder="搜索内容">
              <span class="value-display">值: "{{ textInputs.search }}"</span>
            </div>
            
            <div class="input-demo">
              <label>URL:</label>
              <input v-model="textInputs.url" type="url" placeholder="输入网址">
              <span class="value-display">值: "{{ textInputs.url }}"</span>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 多行文本和复选框 -->
    <div class="section">
      <h3>2. 多行文本和复选框</h3>
      <div class="card">
        <div class="demo-group">
          <div class="textarea-checkbox-grid">
            <!-- 多行文本 -->
            <div class="textarea-demo">
              <h5>多行文本</h5>
              
              <div class="textarea-example">
                <label>消息:</label>
                <textarea v-model="textareaData.message" placeholder="输入多行消息"></textarea>
                <span class="value-display">字符数: {{ textareaData.message.length }}</span>
              </div>
              
              <div class="textarea-example">
                <label>描述:</label>
                <textarea v-model="textareaData.description" placeholder="输入描述" rows="3"></textarea>
                <span class="value-display">行数: {{ textareaData.description.split('\n').length }}</span>
              </div>
            </div>
            
            <!-- 复选框 -->
            <div class="checkbox-demo">
              <h5>复选框</h5>
              
              <div class="single-checkboxes">
                <h6>单个复选框:</h6>
                <label class="checkbox-label">
                  <input type="checkbox" v-model="checkboxData.agreeTerms">
                  同意服务条款 ({{ checkboxData.agreeTerms }})
                </label>
                
                <label class="checkbox-label">
                  <input type="checkbox" v-model="checkboxData.newsletter">
                  订阅新闻通讯 ({{ checkboxData.newsletter }})
                </label>
                
                <label class="checkbox-label">
                  <input type="checkbox" v-model="checkboxData.notifications">
                  接收通知 ({{ checkboxData.notifications }})
                </label>
              </div>
              
              <div class="multiple-checkboxes">
                <h6>多选复选框 - 爱好:</h6>
                <div class="checkbox-group">
                  <label v-for="hobby in options.hobbies" :key="hobby.value" class="checkbox-label">
                    <input type="checkbox" :value="hobby.value" v-model="checkboxData.hobbies">
                    {{ hobby.label }}
                  </label>
                </div>
                <span class="value-display">选中: {{ checkboxData.hobbies.join(', ') }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 单选按钮和下拉选择 -->
    <div class="section">
      <h3>3. 单选按钮和下拉选择</h3>
      <div class="card">
        <div class="demo-group">
          <div class="radio-select-grid">
            <!-- 单选按钮 -->
            <div class="radio-demo">
              <h5>单选按钮</h5>
              
              <div class="radio-group">
                <h6>性别:</h6>
                <label class="radio-label">
                  <input type="radio" value="male" v-model="radioData.gender">
                  男性
                </label>
                <label class="radio-label">
                  <input type="radio" value="female" v-model="radioData.gender">
                  女性
                </label>
                <label class="radio-label">
                  <input type="radio" value="other" v-model="radioData.gender">
                  其他
                </label>
                <span class="value-display">选择: {{ radioData.gender }}</span>
              </div>
              
              <div class="radio-group">
                <h6>工作经验:</h6>
                <label class="radio-label">
                  <input type="radio" value="0-1" v-model="radioData.experience">
                  0-1年
                </label>
                <label class="radio-label">
                  <input type="radio" value="1-3" v-model="radioData.experience">
                  1-3年
                </label>
                <label class="radio-label">
                  <input type="radio" value="3-5" v-model="radioData.experience">
                  3-5年
                </label>
                <label class="radio-label">
                  <input type="radio" value="5+" v-model="radioData.experience">
                  5年以上
                </label>
                <span class="value-display">选择: {{ radioData.experience }}</span>
              </div>
            </div>
            
            <!-- 下拉选择 -->
            <div class="select-demo">
              <h5>下拉选择</h5>
              
              <div class="select-example">
                <label>国家:</label>
                <select v-model="selectData.country">
                  <option value="">请选择国家</option>
                  <option v-for="country in options.countries" :key="country.value" :value="country.value">
                    {{ country.label }}
                  </option>
                </select>
                <span class="value-display">选择: {{ selectData.country }}</span>
              </div>
              
              <div class="select-example">
                <label>优先级:</label>
                <select v-model="selectData.priority">
                  <option value="">选择优先级</option>
                  <option value="low">低</option>
                  <option value="medium">中</option>
                  <option value="high">高</option>
                  <option value="urgent">紧急</option>
                </select>
                <span class="value-display">选择: {{ selectData.priority }}</span>
              </div>
              
              <div class="select-example">
                <label>标签 (多选):</label>
                <select v-model="selectData.tags" multiple>
                  <option v-for="tag in options.tags" :key="tag.value" :value="tag.value">
                    {{ tag.label }}
                  </option>
                </select>
                <span class="value-display">选择: {{ selectData.tags.join(', ') }}</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 文件上传和范围输入 -->
    <div class="section">
      <h3>4. 文件上传和范围输入</h3>
      <div class="card">
        <div class="demo-group">
          <div class="file-range-grid">
            <!-- 文件上传 -->
            <div class="file-demo">
              <h5>文件上传</h5>
              
              <div class="file-example">
                <label>头像 (单文件):</label>
                <input type="file" @change="handleFileUpload($event, 'avatar')" accept="image/*">
                <div v-if="fileData.avatar" class="file-info">
                  <span>{{ fileData.avatar.name }}</span>
                  <button @click="removeFile('avatar')" class="remove-btn">删除</button>
                </div>
              </div>
              
              <div class="file-example">
                <label>文档 (多文件):</label>
                <input type="file" @change="handleFileUpload($event, 'documents')" multiple>
                <div v-if="fileData.documents.length > 0" class="file-list">
                  <div v-for="(file, index) in fileData.documents" :key="index" class="file-info">
                    <span>{{ file.name }}</span>
                    <button @click="removeFile('documents', index)" class="remove-btn">删除</button>
                  </div>
                </div>
              </div>
            </div>
            
            <!-- 范围输入 -->
            <div class="range-demo">
              <h5>范围输入</h5>
              
              <div class="range-example">
                <label>音量: {{ rangeData.volume }}%</label>
                <input type="range" v-model="rangeData.volume" min="0" max="100">
                <div class="range-visual">
                  <div class="range-fill" :style="{ width: rangeData.volume + '%' }"></div>
                </div>
              </div>
              
              <div class="range-example">
                <label>亮度: {{ rangeData.brightness }}%</label>
                <input type="range" v-model="rangeData.brightness" min="0" max="100" step="5">
                <div class="range-visual brightness">
                  <div class="range-fill" :style="{ width: rangeData.brightness + '%' }"></div>
                </div>
              </div>
              
              <div class="range-example">
                <label>价格: ¥{{ rangeData.price }}</label>
                <input type="range" v-model.number="rangeData.price" min="0" max="1000" step="10">
                <div class="price-indicator">
                  <span>¥0</span>
                  <span>¥{{ rangeData.price }}</span>
                  <span>¥1000</span>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 日期时间输入 -->
    <div class="section">
      <h3>5. 日期时间输入</h3>
      <div class="card">
        <div class="demo-group">
          <h4>各种日期时间控件</h4>
          
          <div class="datetime-grid">
            <div class="datetime-example">
              <label>生日:</label>
              <input type="date" v-model="dateTimeData.birthday">
              <span class="value-display">{{ dateTimeData.birthday }}</span>
            </div>
            
            <div class="datetime-example">
              <label>预约日期:</label>
              <input type="date" v-model="dateTimeData.appointmentDate">
              <span class="value-display">{{ dateTimeData.appointmentDate }}</span>
            </div>
            
            <div class="datetime-example">
              <label>预约时间:</label>
              <input type="time" v-model="dateTimeData.appointmentTime">
              <span class="value-display">{{ dateTimeData.appointmentTime }}</span>
            </div>
            
            <div class="datetime-example">
              <label>事件日期时间:</label>
              <input type="datetime-local" v-model="dateTimeData.eventDateTime">
              <span class="value-display">{{ dateTimeData.eventDateTime }}</span>
            </div>
            
            <div class="datetime-example">
              <label>起始周:</label>
              <input type="week" v-model="dateTimeData.startWeek">
              <span class="value-display">{{ dateTimeData.startWeek }}</span>
            </div>
            
            <div class="datetime-example">
              <label>起始月份:</label>
              <input type="month" v-model="dateTimeData.startMonth">
              <span class="value-display">{{ dateTimeData.startMonth }}</span>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 修饰符演示 -->
    <div class="section">
      <h3>6. v-model 修饰符</h3>
      <div class="card">
        <div class="demo-group">
          <h4>修饰符对比演示</h4>
          
          <div class="modifiers-grid">
            <!-- .lazy 修饰符 -->
            <div class="modifier-demo">
              <h5>.lazy 修饰符</h5>
              <p class="modifier-desc">在change事件而非input事件中更新</p>
              
              <div class="modifier-example">
                <label>普通输入 (实时更新):</label>
                <input v-model="lazyDemo.normalInput" placeholder="每个字符都触发更新">
                <span class="update-count">更新次数: {{ lazyDemo.normalCount }}</span>
              </div>
              
              <div class="modifier-example">
                <label>lazy输入 (失焦更新):</label>
                <input v-model.lazy="lazyDemo.lazyInput" placeholder="失焦时才更新">
                <span class="update-count">更新次数: {{ lazyDemo.lazyCount }}</span>
              </div>
            </div>
            
            <!-- .number 修饰符 -->
            <div class="modifier-demo">
              <h5>.number 修饰符</h5>
              <p class="modifier-desc">自动将用户输入转换为数字</p>
              
              <div class="modifier-example">
                <label>字符串输入:</label>
                <input v-model="numberDemo.stringInput" placeholder="输入数字">
                <span class="type-display">类型: {{ typeof numberDemo.stringInput }}, 值: {{ numberDemo.stringInput }}</span>
              </div>
              
              <div class="modifier-example">
                <label>数字输入 (.number):</label>
                <input v-model.number="numberDemo.numberInput" placeholder="输入数字">
                <span class="type-display">类型: {{ typeof numberDemo.numberInput }}, 值: {{ numberDemo.numberInput }}</span>
              </div>
              
              <div class="modifier-example">
                <label>价格输入 (type="number" + .number):</label>
                <input type="number" v-model.number="numberDemo.priceInput" placeholder="输入价格" step="0.01">
                <span class="type-display">类型: {{ typeof numberDemo.priceInput }}, 值: {{ numberDemo.priceInput }}</span>
              </div>
            </div>
            
            <!-- .trim 修饰符 -->
            <div class="modifier-demo">
              <h5>.trim 修饰符</h5>
              <p class="modifier-desc">自动去除首尾空格</p>
              
              <div class="modifier-example">
                <label>普通输入:</label>
                <input v-model="trimDemo.normalInput" placeholder="   输入带空格的文本   ">
                <span class="trim-display">长度: {{ trimDemo.normalInput.length }}, 值: "{{ trimDemo.normalInput }}"</span>
              </div>
              
              <div class="modifier-example">
                <label>trim输入 (.trim):</label>
                <input v-model.trim="trimDemo.trimmedInput" placeholder="   自动去除首尾空格   ">
                <span class="trim-display">长度: {{ trimDemo.trimmedInput.length }}, 值: "{{ trimDemo.trimmedInput }}"</span>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 完整注册表单 -->
    <div class="section">
      <h3>7. 完整注册表单示例</h3>
      <div class="card">
        <div class="demo-group">
          <div class="form-header">
            <h4>用户注册表单</h4>
            <div class="form-progress">
              <div class="progress-bar">
                <div class="progress-fill" :style="{ width: formCompletion + '%' }"></div>
              </div>
              <span class="progress-text">完成度: {{ formCompletion }}%</span>
            </div>
            <div class="form-actions">
              <button @click="loadSampleData" class="sample-btn">加载示例数据</button>
              <button @click="resetForm" class="reset-btn">重置表单</button>
            </div>
          </div>
          
          <form @submit.prevent="submitForm" class="registration-form">
            <!-- 个人信息 -->
            <fieldset class="form-section">
              <legend>个人信息</legend>
              
              <div class="form-row">
                <div class="form-field">
                  <label>姓:</label>
                  <input v-model="registrationForm.firstName" type="text" placeholder="姓">
                </div>
                <div class="form-field">
                  <label>名:</label>
                  <input v-model="registrationForm.lastName" type="text" placeholder="名">
                </div>
              </div>
              
              <div class="form-row">
                <div class="form-field">
                  <label>邮箱 *:</label>
                  <input 
                    v-model="registrationForm.email" 
                    type="email" 
                    placeholder="邮箱地址"
                    @blur="validateField('email', registrationForm.email)"
                    :class="{ error: validation.errors.email && validation.touched.email }"
                  >
                  <span v-if="validation.errors.email && validation.touched.email" class="error-message">
                    {{ validation.errors.email }}
                  </span>
                </div>
                <div class="form-field">
                  <label>用户名 *:</label>
                  <input 
                    v-model="registrationForm.username" 
                    type="text" 
                    placeholder="用户名"
                    @blur="validateField('username', registrationForm.username)"
                    :class="{ error: validation.errors.username && validation.touched.username }"
                  >
                  <span v-if="validation.errors.username && validation.touched.username" class="error-message">
                    {{ validation.errors.username }}
                  </span>
                </div>
              </div>
              
              <div class="form-row">
                <div class="form-field">
                  <label>密码 *:</label>
                  <input 
                    v-model="registrationForm.password" 
                    type="password" 
                    placeholder="密码"
                    @blur="validateField('password', registrationForm.password)"
                    :class="{ error: validation.errors.password && validation.touched.password }"
                  >
                  <span v-if="validation.errors.password && validation.touched.password" class="error-message">
                    {{ validation.errors.password }}
                  </span>
                </div>
                <div class="form-field">
                  <label>确认密码 *:</label>
                  <input 
                    v-model="registrationForm.confirmPassword" 
                    type="password" 
                    placeholder="确认密码"
                    @blur="validateField('confirmPassword', registrationForm.confirmPassword)"
                    :class="{ error: validation.errors.confirmPassword && validation.touched.confirmPassword }"
                  >
                  <span v-if="validation.errors.confirmPassword && validation.touched.confirmPassword" class="error-message">
                    {{ validation.errors.confirmPassword }}
                  </span>
                </div>
              </div>
              
              <div class="form-row">
                <div class="form-field">
                  <label>生日:</label>
                  <input v-model="registrationForm.birthDate" type="date">
                </div>
                <div class="form-field">
                  <label>性别:</label>
                  <div class="radio-group horizontal">
                    <label class="radio-label">
                      <input type="radio" value="male" v-model="registrationForm.gender">
                      男性
                    </label>
                    <label class="radio-label">
                      <input type="radio" value="female" v-model="registrationForm.gender">
                      女性
                    </label>
                    <label class="radio-label">
                      <input type="radio" value="other" v-model="registrationForm.gender">
                      其他
                    </label>
                  </div>
                </div>
              </div>
              
              <div class="form-row">
                <div class="form-field">
                  <label>手机号:</label>
                  <input 
                    v-model="registrationForm.phone" 
                    type="tel" 
                    placeholder="手机号"
                    @blur="validateField('phone', registrationForm.phone)"
                    :class="{ error: validation.errors.phone && validation.touched.phone }"
                  >
                  <span v-if="validation.errors.phone && validation.touched.phone" class="error-message">
                    {{ validation.errors.phone }}
                  </span>
                </div>
              </div>
            </fieldset>
            
            <!-- 地址信息 -->
            <fieldset class="form-section">
              <legend>地址信息</legend>
              
              <div class="form-row">
                <div class="form-field">
                  <label>国家:</label>
                  <select v-model="registrationForm.country">
                    <option value="">选择国家</option>
                    <option v-for="country in options.countries" :key="country.value" :value="country.value">
                      {{ country.label }}
                    </option>
                  </select>
                </div>
                <div class="form-field">
                  <label>省/州:</label>
                  <input v-model="registrationForm.state" type="text" placeholder="省/州">
                </div>
              </div>
              
              <div class="form-row">
                <div class="form-field">
                  <label>城市:</label>
                  <input v-model="registrationForm.city" type="text" placeholder="城市">
                </div>
                <div class="form-field">
                  <label>邮政编码:</label>
                  <input 
                    v-model="registrationForm.zipCode" 
                    type="text" 
                    placeholder="邮政编码"
                    @blur="validateField('zipCode', registrationForm.zipCode)"
                    :class="{ error: validation.errors.zipCode && validation.touched.zipCode }"
                  >
                  <span v-if="validation.errors.zipCode && validation.touched.zipCode" class="error-message">
                    {{ validation.errors.zipCode }}
                  </span>
                </div>
              </div>
              
              <div class="form-row">
                <div class="form-field full-width">
                  <label>详细地址:</label>
                  <input v-model="registrationForm.address" type="text" placeholder="详细地址">
                </div>
              </div>
            </fieldset>
            
            <!-- 兴趣爱好 -->
            <fieldset class="form-section">
              <legend>兴趣爱好</legend>
              
              <div class="form-field">
                <label>兴趣爱好:</label>
                <div class="checkbox-grid">
                  <label v-for="interest in options.hobbies" :key="interest.value" class="checkbox-label">
                    <input type="checkbox" :value="interest.value" v-model="registrationForm.interests">
                    {{ interest.label }}
                  </label>
                </div>
              </div>
            </fieldset>
            
            <!-- 通知设置 -->
            <fieldset class="form-section">
              <legend>通知设置</legend>
              
              <div class="form-row">
                <div class="form-field">
                  <label class="checkbox-label">
                    <input type="checkbox" v-model="registrationForm.newsletter">
                    订阅新闻通讯
                  </label>
                </div>
                <div class="form-field">
                  <label class="checkbox-label">
                    <input type="checkbox" v-model="registrationForm.emailNotifications">
                    邮件通知
                  </label>
                </div>
                <div class="form-field">
                  <label class="checkbox-label">
                    <input type="checkbox" v-model="registrationForm.smsNotifications">
                    短信通知
                  </label>
                </div>
              </div>
            </fieldset>
            
            <!-- 安全设置 -->
            <fieldset class="form-section">
              <legend>安全设置</legend>
              
              <div class="form-row">
                <div class="form-field">
                  <label>安全问题:</label>
                  <select v-model="registrationForm.securityQuestion">
                    <option value="">选择安全问题</option>
                    <option value="pet">您第一只宠物的名字？</option>
                    <option value="school">您的小学名称？</option>
                    <option value="mother">您母亲的姓名？</option>
                    <option value="city">您出生的城市？</option>
                  </select>
                </div>
                <div class="form-field">
                  <label>安全答案:</label>
                  <input v-model="registrationForm.securityAnswer" type="text" placeholder="安全答案">
                </div>
              </div>
            </fieldset>
            
            <!-- 协议同意 -->
            <fieldset class="form-section">
              <legend>协议同意</legend>
              
              <div class="form-row">
                <div class="form-field">
                  <label class="checkbox-label required">
                    <input type="checkbox" v-model="registrationForm.agreeTerms" required>
                    我同意 <a href="#" @click.prevent>服务条款</a> *
                  </label>
                </div>
                <div class="form-field">
                  <label class="checkbox-label required">
                    <input type="checkbox" v-model="registrationForm.agreePrivacy" required>
                    我同意 <a href="#" @click.prevent>隐私政策</a> *
                  </label>
                </div>
              </div>
            </fieldset>
            
            <!-- 提交按钮 -->
            <div class="form-submit">
              <button 
                type="submit" 
                :disabled="!isFormValid || validation.isSubmitting"
                class="submit-button"
              >
                <span v-if="validation.isSubmitting">提交中...</span>
                <span v-else>{{ isFormValid ? '提交注册' : '请完善表单' }}</span>
              </button>
              
              <div class="form-status">
                <span v-if="isFormValid" class="status-valid">✅ 表单验证通过</span>
                <span v-else class="status-invalid">❌ 请检查必填项和错误信息</span>
              </div>
            </div>
          </form>
        </div>
      </div>
    </div>
    
    <!-- 动态表单生成器 -->
    <div class="section">
      <h3>8. 动态表单生成器</h3>
      <div class="card">
        <div class="demo-group">
          <h4>自定义表单构建器</h4>
          
          <div class="dynamic-form-builder">
            <div class="builder-controls">
              <button @click="addField" class="add-field-btn">添加字段</button>
              <span class="field-count">字段数量: {{ dynamicForm.fields.length }}</span>
            </div>
            
            <div class="dynamic-fields">
              <div v-for="(field, index) in dynamicForm.fields" :key="field.id" class="dynamic-field">
                <div class="field-config">
                  <div class="config-row">
                    <input v-model="field.label" placeholder="字段标签" class="label-input">
                    <select v-model="field.type" class="type-select">
                      <option v-for="type in fieldTypeOptions" :key="type.value" :value="type.value">
                        {{ type.label }}
                      </option>
                    </select>
                    <label class="required-checkbox">
                      <input type="checkbox" v-model="field.required">
                      必填
                    </label>
                    <button @click="removeField(field.id)" class="remove-field-btn">删除</button>
                  </div>
                  
                  <!-- 选项设置（适用于select、radio、checkbox） -->
                  <div v-if="['select', 'radio', 'checkbox'].includes(field.type)" class="options-config">
                    <div class="options-header">
                      <span>选项设置:</span>
                      <button @click="addOption(index)" class="add-option-btn">添加选项</button>
                    </div>
                    <div v-if="field.options" class="options-list">
                      <div v-for="(option, optionIndex) in field.options" :key="optionIndex" class="option-item">
                        <input v-model="field.options[optionIndex]" placeholder="选项文本" class="option-input">
                        <button @click="removeOption(index, optionIndex)" class="remove-option-btn">删除</button>
                      </div>
                    </div>
                  </div>
                </div>
                
                <!-- 字段预览 -->
                <div class="field-preview">
                  <label>{{ field.label }}{{ field.required ? ' *' : '' }}:</label>
                  
                  <!-- 根据字段类型渲染不同的输入控件 -->
                  <input 
                    v-if="['text', 'email', 'password', 'number', 'date'].includes(field.type)"
                    v-model="field.value"
                    :type="field.type"
                    :placeholder="field.label"
                    :required="field.required"
                  >
                  
                  <textarea 
                    v-else-if="field.type === 'textarea'"
                    v-model="field.value"
                    :placeholder="field.label"
                    :required="field.required"
                  ></textarea>
                  
                  <select 
                    v-else-if="field.type === 'select'"
                    v-model="field.value"
                    :required="field.required"
                  >
                    <option value="">请选择</option>
                    <option v-for="(option, optionIndex) in field.options" :key="optionIndex" :value="option">
                      {{ option }}
                    </option>
                  </select>
                  
                  <div v-else-if="field.type === 'radio'" class="radio-group">
                    <label v-for="(option, optionIndex) in field.options" :key="optionIndex" class="radio-label">
                      <input type="radio" :value="option" v-model="field.value" :required="field.required">
                      {{ option }}
                    </label>
                  </div>
                  
                  <div v-else-if="field.type === 'checkbox'" class="checkbox-group">
                    <label class="checkbox-label">
                      <input type="checkbox" v-model="field.value" :required="field.required">
                      {{ field.label }}
                    </label>
                  </div>
                  
                  <input 
                    v-else-if="field.type === 'file'"
                    type="file"
                    :required="field.required"
                  >
                  
                  <span class="field-value">当前值: {{ field.value }}</span>
                </div>
              </div>
            </div>
            
            <!-- 生成的表单数据预览 -->
            <div class="form-data-preview">
              <h5>表单数据预览:</h5>
              <pre class="data-json">{{ JSON.stringify(dynamicForm.fields.map(f => ({ label: f.label, type: f.type, value: f.value, required: f.required })), null, 2) }}</pre>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 总结 -->
    <div class="section summary">
      <h3>📝 表单绑定总结</h3>
      <div class="card">
        <div class="summary-grid">
          <div class="summary-item">
            <h4>🎯 基础语法</h4>
            <ul>
              <li><code>v-model="value"</code> - 双向绑定</li>
              <li><code>v-model.lazy</code> - 延迟更新</li>
              <li><code>v-model.number</code> - 数字转换</li>
              <li><code>v-model.trim</code> - 去除空格</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>📝 表单控件</h4>
            <ul>
              <li><strong>文本</strong>: input, textarea</li>
              <li><strong>选择</strong>: checkbox, radio, select</li>
              <li><strong>文件</strong>: file (需要事件处理)</li>
              <li><strong>范围</strong>: range, date, time</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>✅ 表单验证</h4>
            <ul>
              <li>实时验证和错误提示</li>
              <li>必填字段标记</li>
              <li>格式验证（邮箱、手机号等）</li>
              <li>自定义验证规则</li>
            </ul>
          </div>
          
          <div class="summary-item">
            <h4>🚀 最佳实践</h4>
            <ul>
              <li>合理使用修饰符</li>
              <li>及时表单验证</li>
              <li>良好的用户体验</li>
              <li>数据类型转换</li>
            </ul>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.form-binding {
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

/* 基础输入样式 */
.input-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(300px, 1fr));
  gap: 20px;
}

.input-demo {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 15px;
  background: #f9f9f9;
  border-radius: 8px;
  border: 2px solid #ddd;
}

.input-demo label {
  font-weight: bold;
  color: #333;
}

.input-demo input {
  padding: 8px 12px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  transition: border-color 0.3s;
}

.input-demo input:focus {
  outline: none;
  border-color: #42b883;
}

.value-display {
  font-size: 12px;
  color: #666;
  background: #f0f0f0;
  padding: 4px 8px;
  border-radius: 4px;
  font-family: monospace;
}

/* 多行文本和复选框 */
.textarea-checkbox-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 30px;
}

.textarea-demo,
.checkbox-demo {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.textarea-demo h5,
.checkbox-demo h5 {
  margin: 0;
  color: #333;
}

.textarea-example {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.textarea-example label {
  font-weight: bold;
  color: #333;
}

.textarea-example textarea {
  padding: 8px 12px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  min-height: 80px;
  resize: vertical;
  font-family: inherit;
}

.textarea-example textarea:focus {
  outline: none;
  border-color: #42b883;
}

.single-checkboxes,
.multiple-checkboxes {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.single-checkboxes h6,
.multiple-checkboxes h6 {
  margin: 0;
  color: #333;
  font-size: 14px;
}

.checkbox-label {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  font-size: 14px;
}

.checkbox-label input[type="checkbox"] {
  width: 16px;
  height: 16px;
}

.checkbox-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

/* 单选按钮和下拉选择 */
.radio-select-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 30px;
}

.radio-demo,
.select-demo {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.radio-demo h5,
.select-demo h5 {
  margin: 0;
  color: #333;
}

.radio-group {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.radio-group h6 {
  margin: 0;
  color: #333;
  font-size: 14px;
}

.radio-group.horizontal {
  flex-direction: row;
  flex-wrap: wrap;
  gap: 15px;
}

.radio-label {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  font-size: 14px;
}

.radio-label input[type="radio"] {
  width: 16px;
  height: 16px;
}

.select-example {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.select-example label {
  font-weight: bold;
  color: #333;
}

.select-example select {
  padding: 8px 12px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
  background: white;
}

.select-example select[multiple] {
  min-height: 100px;
}

.select-example select:focus {
  outline: none;
  border-color: #42b883;
}

/* 文件上传和范围输入 */
.file-range-grid {
  display: grid;
  grid-template-columns: 1fr 1fr;
  gap: 30px;
}

.file-demo,
.range-demo {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.file-demo h5,
.range-demo h5 {
  margin: 0;
  color: #333;
}

.file-example {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.file-example label {
  font-weight: bold;
  color: #333;
}

.file-example input[type="file"] {
  padding: 8px;
  border: 2px solid #ddd;
  border-radius: 4px;
}

.file-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 8px;
  background: #f0f0f0;
  border-radius: 4px;
  margin-top: 5px;
}

.file-list {
  max-height: 150px;
  overflow-y: auto;
}

.remove-btn {
  background: #f44336;
  color: white;
  border: none;
  padding: 4px 8px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
}

.remove-btn:hover {
  background: #d32f2f;
}

.range-example {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.range-example label {
  font-weight: bold;
  color: #333;
}

.range-example input[type="range"] {
  width: 100%;
}

.range-visual {
  height: 8px;
  background: #ddd;
  border-radius: 4px;
  overflow: hidden;
  position: relative;
}

.range-fill {
  height: 100%;
  background: #42b883;
  transition: width 0.3s;
}

.range-visual.brightness .range-fill {
  background: linear-gradient(to right, #333, #fff);
}

.price-indicator {
  display: flex;
  justify-content: space-between;
  font-size: 12px;
  color: #666;
}

/* 日期时间输入 */
.datetime-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 15px;
}

.datetime-example {
  display: flex;
  flex-direction: column;
  gap: 8px;
  padding: 15px;
  background: #f9f9f9;
  border-radius: 8px;
}

.datetime-example label {
  font-weight: bold;
  color: #333;
}

.datetime-example input {
  padding: 8px 12px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.datetime-example input:focus {
  outline: none;
  border-color: #42b883;
}

/* 修饰符演示 */
.modifiers-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(350px, 1fr));
  gap: 20px;
}

.modifier-demo {
  padding: 15px;
  background: #f9f9f9;
  border-radius: 8px;
  border: 2px solid #ddd;
}

.modifier-demo h5 {
  margin: 0 0 10px 0;
  color: #333;
}

.modifier-desc {
  margin: 0 0 15px 0;
  color: #666;
  font-size: 14px;
  font-style: italic;
}

.modifier-example {
  display: flex;
  flex-direction: column;
  gap: 8px;
  margin-bottom: 15px;
}

.modifier-example label {
  font-weight: bold;
  color: #333;
}

.modifier-example input {
  padding: 8px 12px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.modifier-example input:focus {
  outline: none;
  border-color: #42b883;
}

.update-count,
.type-display,
.trim-display {
  font-size: 12px;
  color: #666;
  background: #f0f0f0;
  padding: 4px 8px;
  border-radius: 4px;
  font-family: monospace;
}

/* 注册表单 */
.registration-form {
  max-width: 800px;
}

.form-header {
  display: flex;
  flex-direction: column;
  gap: 15px;
  margin-bottom: 30px;
}

.form-header h4 {
  margin: 0;
  color: #333;
}

.form-progress {
  display: flex;
  align-items: center;
  gap: 10px;
}

.progress-bar {
  flex: 1;
  height: 8px;
  background: #ddd;
  border-radius: 4px;
  overflow: hidden;
}

.progress-fill {
  height: 100%;
  background: #42b883;
  transition: width 0.3s;
}

.progress-text {
  font-size: 14px;
  color: #666;
}

.form-actions {
  display: flex;
  gap: 10px;
}

.sample-btn,
.reset-btn {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.sample-btn {
  background: #2196f3;
  color: white;
}

.reset-btn {
  background: #ff9800;
  color: white;
}

.form-section {
  border: 2px solid #ddd;
  border-radius: 8px;
  padding: 20px;
  margin-bottom: 20px;
}

.form-section legend {
  font-weight: bold;
  color: #42b883;
  padding: 0 10px;
}

.form-row {
  display: flex;
  gap: 15px;
  margin-bottom: 15px;
}

.form-field {
  flex: 1;
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.form-field.full-width {
  flex: none;
  width: 100%;
}

.form-field label {
  font-weight: bold;
  color: #333;
}

.form-field input,
.form-field select,
.form-field textarea {
  padding: 8px 12px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.form-field input:focus,
.form-field select:focus,
.form-field textarea:focus {
  outline: none;
  border-color: #42b883;
}

.form-field input.error {
  border-color: #f44336;
}

.error-message {
  color: #f44336;
  font-size: 12px;
}

.checkbox-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 10px;
}

.checkbox-label.required {
  color: #333;
}

.checkbox-label.required a {
  color: #42b883;
  text-decoration: none;
}

.checkbox-label.required a:hover {
  text-decoration: underline;
}

.form-submit {
  display: flex;
  flex-direction: column;
  gap: 15px;
  align-items: center;
  margin-top: 30px;
}

.submit-button {
  padding: 12px 48px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 16px;
  cursor: pointer;
  transition: all 0.3s;
}

.submit-button:hover:not(:disabled) {
  background: #35a372;
  transform: translateY(-1px);
}

.submit-button:disabled {
  background: #ccc;
  cursor: not-allowed;
  transform: none;
}

.form-status {
  text-align: center;
}

.status-valid {
  color: #4caf50;
  font-weight: bold;
}

.status-invalid {
  color: #f44336;
  font-weight: bold;
}

/* 动态表单生成器 */
.dynamic-form-builder {
  max-width: 1000px;
}

.builder-controls {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding: 15px;
  background: #f5f5f5;
  border-radius: 8px;
}

.add-field-btn {
  background: #4caf50;
  color: white;
  border: none;
  padding: 10px 20px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

.field-count {
  color: #666;
  font-size: 14px;
}

.dynamic-fields {
  display: flex;
  flex-direction: column;
  gap: 20px;
  margin-bottom: 30px;
}

.dynamic-field {
  border: 2px solid #ddd;
  border-radius: 8px;
  padding: 20px;
  background: #fafafa;
}

.field-config {
  margin-bottom: 15px;
}

.config-row {
  display: flex;
  gap: 10px;
  align-items: center;
  margin-bottom: 10px;
  flex-wrap: wrap;
}

.label-input {
  flex: 1;
  padding: 6px 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  min-width: 150px;
}

.type-select {
  padding: 6px 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background: white;
}

.required-checkbox {
  display: flex;
  align-items: center;
  gap: 5px;
  cursor: pointer;
  font-size: 14px;
}

.remove-field-btn {
  background: #f44336;
  color: white;
  border: none;
  padding: 6px 12px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
}

.options-config {
  margin-top: 10px;
  padding: 10px;
  background: #f0f0f0;
  border-radius: 4px;
}

.options-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.add-option-btn {
  background: #2196f3;
  color: white;
  border: none;
  padding: 4px 8px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
}

.options-list {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.option-item {
  display: flex;
  gap: 10px;
  align-items: center;
}

.option-input {
  flex: 1;
  padding: 4px 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.remove-option-btn {
  background: #ff9800;
  color: white;
  border: none;
  padding: 2px 6px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 11px;
}

.field-preview {
  padding: 15px;
  background: white;
  border-radius: 4px;
  border: 1px solid #ddd;
}

.field-preview label {
  font-weight: bold;
  color: #333;
  margin-bottom: 5px;
  display: block;
}

.field-preview input,
.field-preview textarea,
.field-preview select {
  width: 100%;
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
  margin-bottom: 10px;
}

.field-value {
  font-size: 12px;
  color: #666;
  font-family: monospace;
}

.form-data-preview {
  margin-top: 20px;
  padding: 15px;
  background: #f5f5f5;
  border-radius: 8px;
}

.form-data-preview h5 {
  margin: 0 0 10px 0;
  color: #333;
}

.data-json {
  background: #333;
  color: #0f0;
  padding: 15px;
  border-radius: 4px;
  overflow-x: auto;
  font-size: 12px;
  margin: 0;
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
  .input-grid {
    grid-template-columns: 1fr;
  }
  
  .textarea-checkbox-grid,
  .radio-select-grid,
  .file-range-grid {
    grid-template-columns: 1fr;
  }
  
  .modifiers-grid {
    grid-template-columns: 1fr;
  }
  
  .form-row {
    flex-direction: column;
  }
  
  .datetime-grid {
    grid-template-columns: 1fr;
  }
  
  .checkbox-grid {
    grid-template-columns: repeat(auto-fit, minmax(120px, 1fr));
  }
  
  .config-row {
    flex-direction: column;
    align-items: stretch;
  }
  
  .label-input {
    min-width: auto;
  }
  
  .summary-grid {
    grid-template-columns: 1fr;
  }
  
  .builder-controls {
    flex-direction: column;
    gap: 10px;
  }
}
</style>