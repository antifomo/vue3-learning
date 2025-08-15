<script setup>
// Day 3 练习 - 动态表单生成器
import { ref, reactive, computed, watch } from 'vue'

// ==================== 状态管理 ====================

// 表单构建器状态
const formBuilder = reactive({
  title: '自定义表单',
  description: '',
  fields: [],
  nextFieldId: 1,
  settings: {
    showRequiredIndicator: true,
    allowMultipleSubmissions: false,
    showProgressBar: true,
    theme: 'default'
  }
})

// 表单数据存储
const formResponses = reactive({
  data: {},
  submissions: [],
  isSubmitting: false,
  validationErrors: {}
})

// 可用字段类型
const fieldTypes = [
  { 
    value: 'text', 
    label: '单行文本', 
    icon: '📝',
    description: '短文本输入',
    defaultProps: { placeholder: '请输入文本', maxLength: 100 }
  },
  { 
    value: 'textarea', 
    label: '多行文本', 
    icon: '📄',
    description: '长文本输入',
    defaultProps: { placeholder: '请输入详细内容', rows: 4 }
  },
  { 
    value: 'email', 
    label: '邮箱', 
    icon: '📧',
    description: '邮箱地址输入',
    defaultProps: { placeholder: 'example@email.com' }
  },
  { 
    value: 'number', 
    label: '数字', 
    icon: '🔢',
    description: '数值输入',
    defaultProps: { min: 0, max: 999999, step: 1 }
  },
  { 
    value: 'phone', 
    label: '手机号', 
    icon: '📱',
    description: '手机号码',
    defaultProps: { placeholder: '13800138000' }
  },
  { 
    value: 'date', 
    label: '日期', 
    icon: '📅',
    description: '日期选择',
    defaultProps: {}
  },
  { 
    value: 'select', 
    label: '下拉选择', 
    icon: '📋',
    description: '单选下拉框',
    defaultProps: { options: ['选项1', '选项2', '选项3'] }
  },
  { 
    value: 'radio', 
    label: '单选按钮', 
    icon: '🔘',
    description: '单选按钮组',
    defaultProps: { options: ['选项1', '选项2', '选项3'] }
  },
  { 
    value: 'checkbox', 
    label: '复选框', 
    icon: '☑️',
    description: '多选复选框',
    defaultProps: { options: ['选项1', '选项2', '选项3'] }
  },
  { 
    value: 'range', 
    label: '滑块', 
    icon: '🎚️',
    description: '数值范围选择',
    defaultProps: { min: 0, max: 100, step: 1 }
  },
  { 
    value: 'file', 
    label: '文件上传', 
    icon: '📎',
    description: '文件选择上传',
    defaultProps: { accept: '*', multiple: false }
  },
  { 
    value: 'rating', 
    label: '评分', 
    icon: '⭐',
    description: '星级评分',
    defaultProps: { max: 5 }
  }
]

// 编辑状态
const editingField = ref(null)
const draggedField = ref(null)
const showFieldLibrary = ref(false)

// ==================== 计算属性 ====================

// 表单完成进度
const formProgress = computed(() => {
  const totalFields = formBuilder.fields.filter(f => f.required).length
  if (totalFields === 0) return 100
  
  const completedFields = formBuilder.fields
    .filter(f => f.required)
    .filter(f => {
      const value = formResponses.data[f.id]
      if (Array.isArray(value)) return value.length > 0
      return value !== undefined && value !== null && value !== ''
    }).length
  
  return Math.round((completedFields / totalFields) * 100)
})

// 表单验证状态
const isFormValid = computed(() => {
  // 检查所有必填字段
  const requiredFields = formBuilder.fields.filter(f => f.required)
  
  return requiredFields.every(field => {
    const value = formResponses.data[field.id]
    
    // 检查值是否存在
    if (value === undefined || value === null || value === '') {
      return false
    }
    
    // 检查数组类型字段
    if (Array.isArray(value)) {
      return value.length > 0
    }
    
    // 特殊字段验证
    if (field.type === 'email') {
      return /^[^\s@]+@[^\s@]+\.[^\s@]+$/.test(value)
    }
    
    if (field.type === 'phone') {
      return /^1[3-9]\d{9}$/.test(value)
    }
    
    return true
  })
})

// 表单JSON预览
const formJSON = computed(() => {
  return {
    title: formBuilder.title,
    description: formBuilder.description,
    fields: formBuilder.fields.map(field => ({
      id: field.id,
      type: field.type,
      label: field.label,
      required: field.required,
      props: field.props
    })),
    settings: formBuilder.settings
  }
})

// ==================== 方法 ====================

// 添加字段
function addField(fieldType) {
  const typeConfig = fieldTypes.find(t => t.value === fieldType.value)
  
  const newField = {
    id: formBuilder.nextFieldId++,
    type: fieldType.value,
    label: fieldType.label,
    required: false,
    props: { ...typeConfig.defaultProps },
    validation: {
      rules: [],
      message: ''
    }
  }
  
  formBuilder.fields.push(newField)
  
  // 初始化表单数据
  initializeFieldData(newField)
  
  // 自动开始编辑新字段
  editingField.value = newField.id
  showFieldLibrary.value = false
}

// 初始化字段数据
function initializeFieldData(field) {
  if (field.type === 'checkbox') {
    formResponses.data[field.id] = []
  } else if (field.type === 'rating') {
    formResponses.data[field.id] = 0
  } else {
    formResponses.data[field.id] = ''
  }
}

// 删除字段
function removeField(fieldId) {
  const index = formBuilder.fields.findIndex(f => f.id === fieldId)
  if (index > -1) {
    formBuilder.fields.splice(index, 1)
    delete formResponses.data[fieldId]
    
    if (editingField.value === fieldId) {
      editingField.value = null
    }
  }
}

// 复制字段
function duplicateField(fieldId) {
  const field = formBuilder.fields.find(f => f.id === fieldId)
  if (field) {
    const newField = {
      ...field,
      id: formBuilder.nextFieldId++,
      label: field.label + ' (副本)'
    }
    
    const index = formBuilder.fields.findIndex(f => f.id === fieldId)
    formBuilder.fields.splice(index + 1, 0, newField)
    
    initializeFieldData(newField)
  }
}

// 移动字段
function moveField(fieldId, direction) {
  const index = formBuilder.fields.findIndex(f => f.id === fieldId)
  if (index === -1) return
  
  const newIndex = direction === 'up' ? index - 1 : index + 1
  if (newIndex < 0 || newIndex >= formBuilder.fields.length) return
  
  const field = formBuilder.fields.splice(index, 1)[0]
  formBuilder.fields.splice(newIndex, 0, field)
}

// 编辑字段
function startEditField(fieldId) {
  editingField.value = editingField.value === fieldId ? null : fieldId
}

// 添加选项（适用于select、radio、checkbox）
function addOption(field) {
  if (!field.props.options) {
    field.props.options = []
  }
  field.props.options.push(`新选项${field.props.options.length + 1}`)
}

// 删除选项
function removeOption(field, index) {
  if (field.props.options && field.props.options.length > 1) {
    field.props.options.splice(index, 1)
  }
}

// 表单提交
function submitForm() {
  if (!isFormValid.value) {
    alert('请填写所有必填字段')
    return
  }
  
  formResponses.isSubmitting = true
  
  // 模拟提交延迟
  setTimeout(() => {
    const submission = {
      id: Date.now(),
      timestamp: new Date().toISOString(),
      data: { ...formResponses.data }
    }
    
    formResponses.submissions.push(submission)
    formResponses.isSubmitting = false
    
    alert('表单提交成功！')
    
    // 如果不允许多次提交，清空表单
    if (!formBuilder.settings.allowMultipleSubmissions) {
      resetForm()
    }
  }, 2000)
}

// 重置表单
function resetForm() {
  formBuilder.fields.forEach(field => {
    initializeFieldData(field)
  })
  formResponses.validationErrors = {}
}

// 预览模式切换
const isPreviewMode = ref(false)

function togglePreviewMode() {
  isPreviewMode.value = !isPreviewMode.value
  editingField.value = null
}

// 导入/导出功能
function exportForm() {
  const dataStr = JSON.stringify(formJSON.value, null, 2)
  const blob = new Blob([dataStr], { type: 'application/json' })
  const url = URL.createObjectURL(blob)
  const a = document.createElement('a')
  a.href = url
  a.download = `${formBuilder.title.replace(/\s+/g, '_')}_form.json`
  a.click()
  URL.revokeObjectURL(url)
}

function importForm(event) {
  const file = event.target.files[0]
  if (!file) return
  
  const reader = new FileReader()
  reader.onload = (e) => {
    try {
      const formData = JSON.parse(e.target.result)
      
      formBuilder.title = formData.title || '导入的表单'
      formBuilder.description = formData.description || ''
      formBuilder.fields = formData.fields || []
      formBuilder.settings = { ...formBuilder.settings, ...formData.settings }
      
      // 重新初始化表单数据
      formResponses.data = {}
      formBuilder.fields.forEach(field => {
        initializeFieldData(field)
      })
      
      // 更新nextFieldId
      if (formBuilder.fields.length > 0) {
        formBuilder.nextFieldId = Math.max(...formBuilder.fields.map(f => f.id)) + 1
      }
      
      alert('表单导入成功！')
    } catch (error) {
      alert('导入失败，请检查文件格式')
      console.error('Import error:', error)
    }
  }
  reader.readAsText(file)
  
  // 清空input
  event.target.value = ''
}

// 预设模板
const templates = [
  {
    name: '用户反馈表',
    description: '收集用户反馈和建议',
    data: {
      title: '用户反馈表',
      description: '请告诉我们您的使用体验和建议',
      fields: [
        { id: 1, type: 'text', label: '姓名', required: true, props: { placeholder: '请输入您的姓名' } },
        { id: 2, type: 'email', label: '邮箱', required: true, props: { placeholder: '请输入邮箱地址' } },
        { id: 3, type: 'rating', label: '产品满意度', required: true, props: { max: 5 } },
        { id: 4, type: 'radio', label: '推荐指数', required: true, props: { options: ['非常推荐', '推荐', '一般', '不推荐'] } },
        { id: 5, type: 'textarea', label: '详细反馈', required: false, props: { placeholder: '请详细描述您的使用体验', rows: 4 } }
      ]
    }
  },
  {
    name: '活动报名表',
    description: '活动参与者信息收集',
    data: {
      title: '活动报名表',
      description: '欢迎参加我们的活动，请填写以下信息',
      fields: [
        { id: 1, type: 'text', label: '姓名', required: true, props: { placeholder: '请输入真实姓名' } },
        { id: 2, type: 'phone', label: '手机号', required: true, props: { placeholder: '请输入手机号' } },
        { id: 3, type: 'select', label: '职业', required: true, props: { options: ['学生', '在职人员', '自由职业', '退休', '其他'] } },
        { id: 4, type: 'checkbox', label: '兴趣方向', required: true, props: { options: ['技术分享', '产品体验', '行业交流', '职业发展'] } },
        { id: 5, type: 'date', label: '期望参与时间', required: true, props: {} }
      ]
    }
  },
  {
    name: '求职申请表',
    description: '求职者基本信息表单',
    data: {
      title: '求职申请表',
      description: '请认真填写个人信息，我们会尽快与您联系',
      fields: [
        { id: 1, type: 'text', label: '姓名', required: true, props: { placeholder: '请输入姓名' } },
        { id: 2, type: 'email', label: '邮箱', required: true, props: { placeholder: '请输入常用邮箱' } },
        { id: 3, type: 'phone', label: '联系电话', required: true, props: { placeholder: '请输入手机号' } },
        { id: 4, type: 'select', label: '工作经验', required: true, props: { options: ['应届毕业生', '1-3年', '3-5年', '5-10年', '10年以上'] } },
        { id: 5, type: 'text', label: '期望薪资', required: false, props: { placeholder: '如：10K-15K' } },
        { id: 6, type: 'file', label: '简历附件', required: true, props: { accept: '.pdf,.doc,.docx', multiple: false } },
        { id: 7, type: 'textarea', label: '自我介绍', required: true, props: { placeholder: '请简要介绍您的工作经历和技能', rows: 5 } }
      ]
    }
  }
]

function loadTemplate(template) {
  const templateData = template.data
  
  formBuilder.title = templateData.title
  formBuilder.description = templateData.description
  formBuilder.fields = [...templateData.fields]
  
  // 重新初始化表单数据
  formResponses.data = {}
  formBuilder.fields.forEach(field => {
    initializeFieldData(field)
  })
  
  // 更新nextFieldId
  formBuilder.nextFieldId = Math.max(...formBuilder.fields.map(f => f.id)) + 1
  
  alert(`模板"${template.name}"加载成功！`)
}

// 拖拽排序相关
function onDragStart(event, field) {
  draggedField.value = field
  event.dataTransfer.effectAllowed = 'move'
}

function onDragOver(event) {
  event.preventDefault()
  event.dataTransfer.dropEffect = 'move'
}

function onDrop(event, targetField) {
  event.preventDefault()
  
  if (!draggedField.value || draggedField.value.id === targetField.id) {
    return
  }
  
  const dragIndex = formBuilder.fields.findIndex(f => f.id === draggedField.value.id)
  const dropIndex = formBuilder.fields.findIndex(f => f.id === targetField.id)
  
  if (dragIndex > -1 && dropIndex > -1) {
    const draggedItem = formBuilder.fields.splice(dragIndex, 1)[0]
    formBuilder.fields.splice(dropIndex, 0, draggedItem)
  }
  
  draggedField.value = null
}

// 监听表单变化
watch(() => formBuilder.fields, () => {
  // 移除已删除字段的数据
  const fieldIds = new Set(formBuilder.fields.map(f => f.id))
  Object.keys(formResponses.data).forEach(key => {
    if (!fieldIds.has(parseInt(key))) {
      delete formResponses.data[key]
    }
  })
}, { deep: true })
</script>

<template>
  <div class="dynamic-form">
    <h2>🎨 动态表单生成器</h2>
    
    <!-- 工具栏 -->
    <div class="toolbar">
      <div class="toolbar-section">
        <button @click="togglePreviewMode" :class="{ active: isPreviewMode }" class="mode-btn">
          {{ isPreviewMode ? '📝 编辑模式' : '👁️ 预览模式' }}
        </button>
        
        <button @click="showFieldLibrary = !showFieldLibrary" class="field-library-btn">
          ➕ 添加字段
        </button>
        
        <button @click="resetForm" class="reset-btn">
          🔄 重置表单
        </button>
      </div>
      
      <div class="toolbar-section">
        <button @click="exportForm" class="export-btn">
          📤 导出表单
        </button>
        
        <label class="import-btn">
          📥 导入表单
          <input type="file" @change="importForm" accept=".json" hidden>
        </label>
      </div>
    </div>
    
    <div class="form-container">
      <!-- 编辑模式 -->
      <div v-if="!isPreviewMode" class="edit-mode">
        <!-- 表单设置 -->
        <div class="form-settings">
          <h3>📋 表单设置</h3>
          
          <div class="setting-group">
            <label>表单标题:</label>
            <input v-model="formBuilder.title" type="text" placeholder="请输入表单标题">
          </div>
          
          <div class="setting-group">
            <label>表单描述:</label>
            <textarea v-model="formBuilder.description" placeholder="请输入表单描述"></textarea>
          </div>
          
          <div class="setting-options">
            <label class="checkbox-label">
              <input type="checkbox" v-model="formBuilder.settings.showRequiredIndicator">
              显示必填项标识
            </label>
            
            <label class="checkbox-label">
              <input type="checkbox" v-model="formBuilder.settings.allowMultipleSubmissions">
              允许多次提交
            </label>
            
            <label class="checkbox-label">
              <input type="checkbox" v-model="formBuilder.settings.showProgressBar">
              显示进度条
            </label>
          </div>
        </div>
        
        <!-- 模板选择 -->
        <div class="templates-section">
          <h3>📚 快速模板</h3>
          <div class="templates-grid">
            <div v-for="template in templates" :key="template.name" class="template-card">
              <h4>{{ template.name }}</h4>
              <p>{{ template.description }}</p>
              <button @click="loadTemplate(template)" class="load-template-btn">
                使用模板
              </button>
            </div>
          </div>
        </div>
        
        <!-- 字段库 -->
        <div v-if="showFieldLibrary" class="field-library">
          <h3>🧩 字段库</h3>
          <div class="field-types-grid">
            <div 
              v-for="fieldType in fieldTypes" 
              :key="fieldType.value"
              @click="addField(fieldType)"
              class="field-type-card"
            >
              <div class="field-type-icon">{{ fieldType.icon }}</div>
              <div class="field-type-info">
                <h4>{{ fieldType.label }}</h4>
                <p>{{ fieldType.description }}</p>
              </div>
            </div>
          </div>
        </div>
        
        <!-- 字段列表 -->
        <div class="fields-editor">
          <h3>📝 表单字段 ({{ formBuilder.fields.length }})</h3>
          
          <div v-if="formBuilder.fields.length === 0" class="empty-fields">
            <p>还没有添加任何字段</p>
            <button @click="showFieldLibrary = true" class="add-first-field-btn">
              添加第一个字段
            </button>
          </div>
          
          <div v-else class="fields-list">
            <div 
              v-for="(field, index) in formBuilder.fields" 
              :key="field.id"
              :draggable="true"
              @dragstart="onDragStart($event, field)"
              @dragover="onDragOver($event)"
              @drop="onDrop($event, field)"
              :class="{ 
                'field-item': true, 
                'editing': editingField === field.id,
                'dragging': draggedField && draggedField.id === field.id
              }"
            >
              <!-- 字段预览 -->
              <div class="field-preview">
                <div class="field-header">
                  <span class="field-icon">{{ fieldTypes.find(t => t.value === field.type)?.icon }}</span>
                  <span class="field-label">{{ field.label }}</span>
                  <span v-if="field.required" class="required-indicator">*</span>
                  <span class="field-type">({{ fieldTypes.find(t => t.value === field.type)?.label }})</span>
                </div>
                
                <div class="field-actions">
                  <button @click="startEditField(field.id)" class="edit-btn">
                    {{ editingField === field.id ? '收起' : '编辑' }}
                  </button>
                  <button @click="duplicateField(field.id)" class="duplicate-btn">复制</button>
                  <button @click="moveField(field.id, 'up')" :disabled="index === 0" class="move-btn">↑</button>
                  <button @click="moveField(field.id, 'down')" :disabled="index === formBuilder.fields.length - 1" class="move-btn">↓</button>
                  <button @click="removeField(field.id)" class="remove-btn">删除</button>
                </div>
              </div>
              
              <!-- 字段编辑 -->
              <div v-if="editingField === field.id" class="field-editor">
                <div class="editor-grid">
                  <div class="editor-field">
                    <label>字段标签:</label>
                    <input v-model="field.label" type="text" placeholder="字段标签">
                  </div>
                  
                  <div class="editor-field">
                    <label class="checkbox-label">
                      <input type="checkbox" v-model="field.required">
                      必填字段
                    </label>
                  </div>
                </div>
                
                <!-- 字段特定属性 -->
                <div class="field-props">
                  <!-- 文本字段属性 -->
                  <template v-if="['text', 'textarea', 'email'].includes(field.type)">
                    <div class="prop-field">
                      <label>占位符:</label>
                      <input v-model="field.props.placeholder" type="text" placeholder="占位符文本">
                    </div>
                    
                    <div v-if="field.type === 'text'" class="prop-field">
                      <label>最大长度:</label>
                      <input v-model.number="field.props.maxLength" type="number" min="1">
                    </div>
                    
                    <div v-if="field.type === 'textarea'" class="prop-field">
                      <label>行数:</label>
                      <input v-model.number="field.props.rows" type="number" min="1" max="10">
                    </div>
                  </template>
                  
                  <!-- 数字字段属性 -->
                  <template v-if="['number', 'range'].includes(field.type)">
                    <div class="prop-row">
                      <div class="prop-field">
                        <label>最小值:</label>
                        <input v-model.number="field.props.min" type="number">
                      </div>
                      <div class="prop-field">
                        <label>最大值:</label>
                        <input v-model.number="field.props.max" type="number">
                      </div>
                      <div class="prop-field">
                        <label>步长:</label>
                        <input v-model.number="field.props.step" type="number" min="0.01" step="0.01">
                      </div>
                    </div>
                  </template>
                  
                  <!-- 选项字段属性 -->
                  <template v-if="['select', 'radio', 'checkbox'].includes(field.type)">
                    <div class="options-editor">
                      <div class="options-header">
                        <label>选项设置:</label>
                        <button @click="addOption(field)" class="add-option-btn">添加选项</button>
                      </div>
                      
                      <div class="options-list">
                        <div v-for="(option, optionIndex) in field.props.options" :key="optionIndex" class="option-item">
                          <input v-model="field.props.options[optionIndex]" type="text" placeholder="选项文本">
                          <button @click="removeOption(field, optionIndex)" :disabled="field.props.options.length <= 1" class="remove-option-btn">
                            删除
                          </button>
                        </div>
                      </div>
                    </div>
                  </template>
                  
                  <!-- 文件字段属性 -->
                  <template v-if="field.type === 'file'">
                    <div class="prop-field">
                      <label>接受的文件类型:</label>
                      <input v-model="field.props.accept" type="text" placeholder="如: .pdf,.doc,.jpg">
                    </div>
                    
                    <div class="prop-field">
                      <label class="checkbox-label">
                        <input type="checkbox" v-model="field.props.multiple">
                        允许多文件上传
                      </label>
                    </div>
                  </template>
                  
                  <!-- 评分字段属性 -->
                  <template v-if="field.type === 'rating'">
                    <div class="prop-field">
                      <label>最高分:</label>
                      <input v-model.number="field.props.max" type="number" min="3" max="10">
                    </div>
                  </template>
                </div>
              </div>
            </div>
          </div>
        </div>
        
        <!-- JSON预览 -->
        <div class="json-preview">
          <h3>💻 表单配置 JSON</h3>
          <pre class="json-content">{{ JSON.stringify(formJSON, null, 2) }}</pre>
        </div>
      </div>
      
      <!-- 预览模式 -->
      <div v-else class="preview-mode">
        <div class="form-preview">
          <!-- 表单头部 -->
          <div class="form-header">
            <h1>{{ formBuilder.title || '未命名表单' }}</h1>
            <p v-if="formBuilder.description" class="form-description">{{ formBuilder.description }}</p>
            
            <!-- 进度条 -->
            <div v-if="formBuilder.settings.showProgressBar && formBuilder.fields.some(f => f.required)" class="progress-bar">
              <div class="progress-fill" :style="{ width: formProgress + '%' }"></div>
              <span class="progress-text">完成进度: {{ formProgress }}%</span>
            </div>
          </div>
          
          <!-- 表单字段 -->
          <form @submit.prevent="submitForm" class="preview-form">
            <div v-for="field in formBuilder.fields" :key="field.id" class="form-field">
              <label class="field-label">
                {{ field.label }}
                <span v-if="field.required && formBuilder.settings.showRequiredIndicator" class="required">*</span>
              </label>
              
              <!-- 文本输入 -->
              <input 
                v-if="field.type === 'text'"
                v-model="formResponses.data[field.id]"
                type="text"
                :placeholder="field.props.placeholder"
                :maxlength="field.props.maxLength"
                :required="field.required"
                class="form-input"
              >
              
              <!-- 多行文本 -->
              <textarea 
                v-else-if="field.type === 'textarea'"
                v-model="formResponses.data[field.id]"
                :placeholder="field.props.placeholder"
                :rows="field.props.rows || 4"
                :required="field.required"
                class="form-textarea"
              ></textarea>
              
              <!-- 邮箱 -->
              <input 
                v-else-if="field.type === 'email'"
                v-model="formResponses.data[field.id]"
                type="email"
                :placeholder="field.props.placeholder"
                :required="field.required"
                class="form-input"
              >
              
              <!-- 数字 -->
              <input 
                v-else-if="field.type === 'number'"
                v-model.number="formResponses.data[field.id]"
                type="number"
                :min="field.props.min"
                :max="field.props.max"
                :step="field.props.step"
                :required="field.required"
                class="form-input"
              >
              
              <!-- 手机号 -->
              <input 
                v-else-if="field.type === 'phone'"
                v-model="formResponses.data[field.id]"
                type="tel"
                :placeholder="field.props.placeholder"
                :required="field.required"
                class="form-input"
              >
              
              <!-- 日期 -->
              <input 
                v-else-if="field.type === 'date'"
                v-model="formResponses.data[field.id]"
                type="date"
                :required="field.required"
                class="form-input"
              >
              
              <!-- 下拉选择 -->
              <select 
                v-else-if="field.type === 'select'"
                v-model="formResponses.data[field.id]"
                :required="field.required"
                class="form-select"
              >
                <option value="">请选择</option>
                <option v-for="option in field.props.options" :key="option" :value="option">
                  {{ option }}
                </option>
              </select>
              
              <!-- 单选按钮 -->
              <div v-else-if="field.type === 'radio'" class="radio-group">
                <label v-for="option in field.props.options" :key="option" class="radio-label">
                  <input 
                    type="radio" 
                    :value="option" 
                    v-model="formResponses.data[field.id]"
                    :required="field.required"
                  >
                  {{ option }}
                </label>
              </div>
              
              <!-- 复选框 -->
              <div v-else-if="field.type === 'checkbox'" class="checkbox-group">
                <label v-for="option in field.props.options" :key="option" class="checkbox-label">
                  <input 
                    type="checkbox" 
                    :value="option" 
                    v-model="formResponses.data[field.id]"
                  >
                  {{ option }}
                </label>
              </div>
              
              <!-- 滑块 -->
              <div v-else-if="field.type === 'range'" class="range-field">
                <input 
                  type="range" 
                  v-model.number="formResponses.data[field.id]"
                  :min="field.props.min"
                  :max="field.props.max"
                  :step="field.props.step"
                  class="form-range"
                >
                <span class="range-value">{{ formResponses.data[field.id] }}</span>
              </div>
              
              <!-- 文件上传 -->
              <input 
                v-else-if="field.type === 'file'"
                type="file"
                :accept="field.props.accept"
                :multiple="field.props.multiple"
                :required="field.required"
                class="form-file"
              >
              
              <!-- 评分 -->
              <div v-else-if="field.type === 'rating'" class="rating-field">
                <div class="stars">
                  <span 
                    v-for="star in field.props.max" 
                    :key="star"
                    @click="formResponses.data[field.id] = star"
                    :class="{ 
                      'star': true, 
                      'active': star <= formResponses.data[field.id] 
                    }"
                  >
                    ⭐
                  </span>
                </div>
                <span class="rating-text">{{ formResponses.data[field.id] }}/{{ field.props.max }}</span>
              </div>
            </div>
            
            <!-- 提交按钮 -->
            <div class="form-submit">
              <button 
                type="submit" 
                :disabled="!isFormValid || formResponses.isSubmitting"
                class="submit-btn"
              >
                <span v-if="formResponses.isSubmitting">提交中...</span>
                <span v-else>{{ isFormValid ? '提交表单' : '请完善表单' }}</span>
              </button>
              
              <div class="form-status">
                <span v-if="isFormValid" class="status-valid">✅ 表单验证通过</span>
                <span v-else class="status-invalid">❌ 请填写必填项</span>
              </div>
            </div>
          </form>
          
          <!-- 提交记录 -->
          <div v-if="formResponses.submissions.length > 0" class="submissions">
            <h3>📊 提交记录 ({{ formResponses.submissions.length }})</h3>
            
            <div class="submissions-list">
              <div v-for="submission in formResponses.submissions" :key="submission.id" class="submission-item">
                <div class="submission-header">
                  <span class="submission-id">#{{ submission.id }}</span>
                  <span class="submission-time">{{ new Date(submission.timestamp).toLocaleString() }}</span>
                </div>
                
                <div class="submission-data">
                  <div v-for="field in formBuilder.fields" :key="field.id" class="data-item">
                    <span class="data-label">{{ field.label }}:</span>
                    <span class="data-value">
                      <template v-if="Array.isArray(submission.data[field.id])">
                        {{ submission.data[field.id].join(', ') }}
                      </template>
                      <template v-else>
                        {{ submission.data[field.id] || '未填写' }}
                      </template>
                    </span>
                  </div>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.dynamic-form {
  padding: 20px;
  max-width: 1400px;
  margin: 0 auto;
}

/* 工具栏 */
.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 30px;
  padding: 15px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  flex-wrap: wrap;
  gap: 15px;
}

.toolbar-section {
  display: flex;
  gap: 10px;
  align-items: center;
}

.mode-btn {
  padding: 10px 20px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.3s;
}

.mode-btn.active {
  background: #2196f3;
}

.field-library-btn,
.reset-btn,
.export-btn,
.import-btn {
  padding: 8px 16px;
  background: #f5f5f5;
  color: #333;
  border: 1px solid #ddd;
  border-radius: 6px;
  cursor: pointer;
  font-size: 14px;
  transition: all 0.3s;
}

.field-library-btn:hover,
.reset-btn:hover,
.export-btn:hover,
.import-btn:hover {
  background: #e0e0e0;
}

.import-btn {
  display: inline-block;
}

/* 表单容器 */
.form-container {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  overflow: hidden;
}

/* 编辑模式 */
.edit-mode {
  padding: 20px;
}

/* 表单设置 */
.form-settings {
  margin-bottom: 30px;
  padding: 20px;
  background: #f9f9f9;
  border-radius: 8px;
}

.form-settings h3 {
  margin: 0 0 15px 0;
  color: #333;
}

.setting-group {
  margin-bottom: 15px;
}

.setting-group label {
  display: block;
  margin-bottom: 5px;
  font-weight: bold;
  color: #333;
}

.setting-group input,
.setting-group textarea {
  width: 100%;
  padding: 8px 12px;
  border: 2px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.setting-group textarea {
  min-height: 80px;
  resize: vertical;
}

.setting-options {
  display: flex;
  gap: 20px;
  flex-wrap: wrap;
}

.checkbox-label {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  font-size: 14px;
}

/* 模板选择 */
.templates-section {
  margin-bottom: 30px;
}

.templates-section h3 {
  margin: 0 0 15px 0;
  color: #333;
}

.templates-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 15px;
}

.template-card {
  padding: 15px;
  background: #f5f5f5;
  border: 2px solid #ddd;
  border-radius: 8px;
  transition: all 0.3s;
}

.template-card:hover {
  border-color: #42b883;
  box-shadow: 0 4px 8px rgba(66, 184, 131, 0.2);
}

.template-card h4 {
  margin: 0 0 8px 0;
  color: #333;
}

.template-card p {
  margin: 0 0 10px 0;
  color: #666;
  font-size: 14px;
}

.load-template-btn {
  background: #42b883;
  color: white;
  border: none;
  padding: 8px 16px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 14px;
}

/* 字段库 */
.field-library {
  margin-bottom: 30px;
  padding: 20px;
  background: #f0f8ff;
  border-radius: 8px;
  border: 2px solid #2196f3;
}

.field-library h3 {
  margin: 0 0 15px 0;
  color: #1976d2;
}

.field-types-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(200px, 1fr));
  gap: 10px;
}

.field-type-card {
  display: flex;
  align-items: center;
  gap: 10px;
  padding: 12px;
  background: white;
  border: 2px solid #e3f2fd;
  border-radius: 8px;
  cursor: pointer;
  transition: all 0.3s;
}

.field-type-card:hover {
  border-color: #2196f3;
  box-shadow: 0 2px 8px rgba(33, 150, 243, 0.2);
}

.field-type-icon {
  font-size: 24px;
}

.field-type-info h4 {
  margin: 0;
  color: #333;
  font-size: 14px;
}

.field-type-info p {
  margin: 2px 0 0 0;
  color: #666;
  font-size: 12px;
}

/* 字段编辑器 */
.fields-editor {
  margin-bottom: 30px;
}

.fields-editor h3 {
  margin: 0 0 15px 0;
  color: #333;
}

.empty-fields {
  text-align: center;
  padding: 40px;
  color: #999;
}

.add-first-field-btn {
  background: #42b883;
  color: white;
  border: none;
  padding: 12px 24px;
  border-radius: 6px;
  cursor: pointer;
  font-size: 16px;
  margin-top: 15px;
}

.fields-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.field-item {
  border: 2px solid #ddd;
  border-radius: 8px;
  background: #fafafa;
  transition: all 0.3s;
  cursor: move;
}

.field-item:hover {
  border-color: #42b883;
}

.field-item.editing {
  border-color: #2196f3;
  box-shadow: 0 4px 12px rgba(33, 150, 243, 0.2);
}

.field-item.dragging {
  opacity: 0.5;
  transform: scale(0.95);
}

.field-preview {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px;
}

.field-header {
  display: flex;
  align-items: center;
  gap: 10px;
}

.field-icon {
  font-size: 18px;
}

.field-label {
  font-weight: bold;
  color: #333;
}

.required-indicator {
  color: #f44336;
  font-weight: bold;
}

.field-type {
  color: #666;
  font-size: 12px;
}

.field-actions {
  display: flex;
  gap: 8px;
}

.field-actions button {
  padding: 4px 8px;
  border: none;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
  transition: all 0.3s;
}

.edit-btn {
  background: #2196f3;
  color: white;
}

.duplicate-btn {
  background: #ff9800;
  color: white;
}

.move-btn {
  background: #9c27b0;
  color: white;
}

.move-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

.remove-btn {
  background: #f44336;
  color: white;
}

/* 字段编辑器 */
.field-editor {
  padding: 15px;
  border-top: 1px solid #ddd;
  background: white;
}

.editor-grid {
  display: grid;
  grid-template-columns: 1fr auto;
  gap: 15px;
  margin-bottom: 15px;
  align-items: end;
}

.editor-field {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.editor-field label {
  font-weight: bold;
  color: #333;
}

.editor-field input {
  padding: 8px 12px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.field-props {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.prop-field {
  display: flex;
  flex-direction: column;
  gap: 5px;
}

.prop-field label {
  font-weight: bold;
  color: #333;
  font-size: 14px;
}

.prop-field input {
  padding: 6px 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  font-size: 14px;
}

.prop-row {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 10px;
}

/* 选项编辑器 */
.options-editor {
  border: 1px solid #ddd;
  border-radius: 4px;
  padding: 10px;
  background: #f9f9f9;
}

.options-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
}

.add-option-btn {
  background: #4caf50;
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
  gap: 8px;
}

.option-item {
  display: flex;
  gap: 10px;
  align-items: center;
}

.option-item input {
  flex: 1;
  padding: 6px 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.remove-option-btn {
  background: #f44336;
  color: white;
  border: none;
  padding: 4px 8px;
  border-radius: 4px;
  cursor: pointer;
  font-size: 12px;
}

.remove-option-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

/* JSON预览 */
.json-preview {
  margin-top: 30px;
}

.json-preview h3 {
  margin: 0 0 10px 0;
  color: #333;
}

.json-content {
  background: #2d2d2d;
  color: #f8f8f2;
  padding: 15px;
  border-radius: 8px;
  overflow-x: auto;
  font-size: 12px;
  max-height: 400px;
  overflow-y: auto;
}

/* 预览模式 */
.preview-mode {
  min-height: 600px;
}

.form-preview {
  max-width: 800px;
  margin: 0 auto;
  padding: 30px;
}

.form-header {
  text-align: center;
  margin-bottom: 30px;
}

.form-header h1 {
  margin: 0 0 10px 0;
  color: #333;
}

.form-description {
  margin: 0 0 20px 0;
  color: #666;
  font-size: 16px;
}

.progress-bar {
  position: relative;
  height: 8px;
  background: #e0e0e0;
  border-radius: 4px;
  overflow: hidden;
  margin-bottom: 10px;
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

/* 表单字段样式 */
.preview-form {
  display: flex;
  flex-direction: column;
  gap: 20px;
}

.form-field {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.field-label {
  font-weight: bold;
  color: #333;
  font-size: 16px;
}

.required {
  color: #f44336;
}

.form-input,
.form-textarea,
.form-select,
.form-file {
  padding: 12px;
  border: 2px solid #ddd;
  border-radius: 6px;
  font-size: 16px;
  transition: border-color 0.3s;
}

.form-input:focus,
.form-textarea:focus,
.form-select:focus {
  outline: none;
  border-color: #42b883;
}

.form-textarea {
  resize: vertical;
  font-family: inherit;
}

.radio-group,
.checkbox-group {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.radio-label,
.checkbox-label {
  display: flex;
  align-items: center;
  gap: 8px;
  cursor: pointer;
  font-size: 16px;
}

.radio-label input,
.checkbox-label input {
  width: 18px;
  height: 18px;
}

.range-field {
  display: flex;
  align-items: center;
  gap: 15px;
}

.form-range {
  flex: 1;
  height: 8px;
}

.range-value {
  font-weight: bold;
  color: #42b883;
  min-width: 40px;
  text-align: center;
}

.rating-field {
  display: flex;
  align-items: center;
  gap: 15px;
}

.stars {
  display: flex;
  gap: 5px;
}

.star {
  font-size: 24px;
  cursor: pointer;
  opacity: 0.3;
  transition: all 0.3s;
}

.star.active {
  opacity: 1;
  transform: scale(1.1);
}

.rating-text {
  font-weight: bold;
  color: #42b883;
}

/* 提交区域 */
.form-submit {
  display: flex;
  flex-direction: column;
  gap: 15px;
  align-items: center;
  margin-top: 30px;
  padding-top: 20px;
  border-top: 2px solid #f0f0f0;
}

.submit-btn {
  padding: 15px 40px;
  background: #42b883;
  color: white;
  border: none;
  border-radius: 8px;
  font-size: 18px;
  cursor: pointer;
  transition: all 0.3s;
}

.submit-btn:hover:not(:disabled) {
  background: #35a372;
  transform: translateY(-2px);
}

.submit-btn:disabled {
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

/* 提交记录 */
.submissions {
  margin-top: 40px;
  padding-top: 20px;
  border-top: 2px solid #f0f0f0;
}

.submissions h3 {
  margin: 0 0 20px 0;
  color: #333;
}

.submissions-list {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.submission-item {
  padding: 15px;
  background: #f9f9f9;
  border-radius: 8px;
  border: 1px solid #ddd;
}

.submission-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 10px;
  font-weight: bold;
}

.submission-id {
  color: #42b883;
}

.submission-time {
  color: #666;
  font-size: 14px;
}

.submission-data {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
  gap: 10px;
}

.data-item {
  display: flex;
  flex-direction: column;
  gap: 2px;
}

.data-label {
  font-weight: bold;
  color: #333;
  font-size: 14px;
}

.data-value {
  color: #666;
  font-size: 14px;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .toolbar {
    flex-direction: column;
    align-items: stretch;
  }
  
  .toolbar-section {
    justify-content: center;
  }
  
  .templates-grid {
    grid-template-columns: 1fr;
  }
  
  .field-types-grid {
    grid-template-columns: 1fr;
  }
  
  .editor-grid {
    grid-template-columns: 1fr;
  }
  
  .prop-row {
    grid-template-columns: 1fr;
  }
  
  .submission-data {
    grid-template-columns: 1fr;
  }
  
  .field-preview {
    flex-direction: column;
    gap: 10px;
    align-items: flex-start;
  }
  
  .field-actions {
    width: 100%;
    justify-content: space-between;
  }
}
</style>