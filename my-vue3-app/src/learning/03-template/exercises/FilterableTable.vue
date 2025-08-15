<script setup>
// Day 3 练习 - 可过滤排序的数据表格
import { ref, reactive, computed, watch, onMounted } from 'vue'

// ==================== 数据管理 ====================

// 表格数据
const tableData = ref([])

// 表格配置
const tableConfig = reactive({
  // 分页配置
  pagination: {
    currentPage: 1,
    pageSize: 10,
    showSizeChanger: true,
    pageSizes: [5, 10, 20, 50, 100],
    showQuickJumper: true,
    showTotal: true
  },
  
  // 列配置
  columns: [
    {
      key: 'id',
      title: 'ID',
      sortable: true,
      filterable: false,
      width: 80,
      type: 'number',
      fixed: 'left'
    },
    {
      key: 'name',
      title: '姓名',
      sortable: true,
      filterable: true,
      width: 120,
      type: 'text',
      searchable: true
    },
    {
      key: 'email',
      title: '邮箱',
      sortable: true,
      filterable: true,
      width: 200,
      type: 'text',
      searchable: true
    },
    {
      key: 'age',
      title: '年龄',
      sortable: true,
      filterable: true,
      width: 80,
      type: 'number',
      range: { min: 18, max: 65 }
    },
    {
      key: 'department',
      title: '部门',
      sortable: true,
      filterable: true,
      width: 120,
      type: 'select',
      options: ['技术部', '销售部', '市场部', '人事部', '财务部']
    },
    {
      key: 'position',
      title: '职位',
      sortable: true,
      filterable: true,
      width: 120,
      type: 'text'
    },
    {
      key: 'salary',
      title: '薪资',
      sortable: true,
      filterable: true,
      width: 100,
      type: 'currency',
      range: { min: 5000, max: 50000 }
    },
    {
      key: 'hireDate',
      title: '入职日期',
      sortable: true,
      filterable: true,
      width: 120,
      type: 'date'
    },
    {
      key: 'status',
      title: '状态',
      sortable: true,
      filterable: true,
      width: 100,
      type: 'select',
      options: ['在职', '离职', '休假', '试用期']
    },
    {
      key: 'performance',
      title: '绩效评分',
      sortable: true,
      filterable: true,
      width: 120,
      type: 'rating',
      range: { min: 1, max: 5 }
    },
    {
      key: 'skills',
      title: '技能',
      sortable: false,
      filterable: true,
      width: 150,
      type: 'tags'
    },
    {
      key: 'actions',
      title: '操作',
      sortable: false,
      filterable: false,
      width: 120,
      type: 'actions',
      fixed: 'right'
    }
  ],
  
  // 排序配置
  sorting: {
    column: 'id',
    direction: 'asc' // asc, desc
  },
  
  // 过滤配置
  filters: {
    global: '', // 全局搜索
    columns: {} // 列过滤
  },
  
  // 选择配置
  selection: {
    enabled: true,
    multiple: true,
    selectedRows: [],
    selectAll: false
  },
  
  // 显示配置
  display: {
    showHeader: true,
    showFooter: true,
    striped: true,
    bordered: true,
    hover: true,
    compact: false,
    loading: false
  }
})

// 高级搜索配置
const advancedSearch = reactive({
  visible: false,
  conditions: [
    {
      id: 1,
      column: 'name',
      operator: 'contains',
      value: '',
      logic: 'and'
    }
  ],
  nextId: 2
})

// 导出配置
const exportConfig = reactive({
  visible: false,
  format: 'csv', // csv, excel, json
  includeHeaders: true,
  selectedOnly: false,
  columns: []
})

// 统计信息
const statistics = reactive({
  visible: false,
  data: {}
})

// ==================== 计算属性 ====================

// 过滤后的数据
const filteredData = computed(() => {
  let result = [...tableData.value]
  
  // 全局搜索
  if (tableConfig.filters.global) {
    const query = tableConfig.filters.global.toLowerCase()
    result = result.filter(row => {
      return tableConfig.columns
        .filter(col => col.searchable)
        .some(col => {
          const value = row[col.key]
          return String(value).toLowerCase().includes(query)
        })
    })
  }
  
  // 列过滤
  Object.entries(tableConfig.filters.columns).forEach(([column, filter]) => {
    if (filter && filter.value !== undefined && filter.value !== '') {
      const columnConfig = tableConfig.columns.find(col => col.key === column)
      
      switch (columnConfig.type) {
        case 'text':
          if (filter.operator === 'contains') {
            result = result.filter(row => 
              String(row[column]).toLowerCase().includes(String(filter.value).toLowerCase())
            )
          } else if (filter.operator === 'equals') {
            result = result.filter(row => row[column] === filter.value)
          } else if (filter.operator === 'startsWith') {
            result = result.filter(row => 
              String(row[column]).toLowerCase().startsWith(String(filter.value).toLowerCase())
            )
          }
          break
          
        case 'number':
        case 'currency':
          if (filter.operator === 'equals') {
            result = result.filter(row => row[column] === Number(filter.value))
          } else if (filter.operator === 'greater') {
            result = result.filter(row => row[column] > Number(filter.value))
          } else if (filter.operator === 'less') {
            result = result.filter(row => row[column] < Number(filter.value))
          } else if (filter.operator === 'between') {
            result = result.filter(row => 
              row[column] >= Number(filter.min) && row[column] <= Number(filter.max)
            )
          }
          break
          
        case 'select':
          if (Array.isArray(filter.value)) {
            result = result.filter(row => filter.value.includes(row[column]))
          } else {
            result = result.filter(row => row[column] === filter.value)
          }
          break
          
        case 'date':
          const filterDate = new Date(filter.value)
          if (filter.operator === 'equals') {
            result = result.filter(row => {
              const rowDate = new Date(row[column])
              return rowDate.toDateString() === filterDate.toDateString()
            })
          } else if (filter.operator === 'after') {
            result = result.filter(row => new Date(row[column]) > filterDate)
          } else if (filter.operator === 'before') {
            result = result.filter(row => new Date(row[column]) < filterDate)
          }
          break
          
        case 'rating':
          if (filter.operator === 'equals') {
            result = result.filter(row => row[column] === Number(filter.value))
          } else if (filter.operator === 'greater') {
            result = result.filter(row => row[column] >= Number(filter.value))
          }
          break
          
        case 'tags':
          if (filter.value) {
            result = result.filter(row => 
              row[column].some(tag => 
                tag.toLowerCase().includes(filter.value.toLowerCase())
              )
            )
          }
          break
      }
    }
  })
  
  // 高级搜索
  if (advancedSearch.visible && advancedSearch.conditions.length > 0) {
    result = result.filter(row => {
      return evaluateAdvancedConditions(row, advancedSearch.conditions)
    })
  }
  
  return result
})

// 排序后的数据
const sortedData = computed(() => {
  if (!tableConfig.sorting.column) return filteredData.value
  
  const { column, direction } = tableConfig.sorting
  const columnConfig = tableConfig.columns.find(col => col.key === column)
  
  return [...filteredData.value].sort((a, b) => {
    let aVal = a[column]
    let bVal = b[column]
    
    // 根据类型处理排序
    switch (columnConfig.type) {
      case 'number':
      case 'currency':
      case 'rating':
        aVal = Number(aVal) || 0
        bVal = Number(bVal) || 0
        break
        
      case 'date':
        aVal = new Date(aVal)
        bVal = new Date(bVal)
        break
        
      case 'text':
      default:
        aVal = String(aVal).toLowerCase()
        bVal = String(bVal).toLowerCase()
    }
    
    let comparison = 0
    if (aVal > bVal) comparison = 1
    if (aVal < bVal) comparison = -1
    
    return direction === 'desc' ? -comparison : comparison
  })
})

// 分页后的数据
const paginatedData = computed(() => {
  const { currentPage, pageSize } = tableConfig.pagination
  const start = (currentPage - 1) * pageSize
  const end = start + pageSize
  
  return sortedData.value.slice(start, end)
})

// 分页信息
const paginationInfo = computed(() => {
  const { currentPage, pageSize } = tableConfig.pagination
  const total = sortedData.value.length
  const start = (currentPage - 1) * pageSize + 1
  const end = Math.min(currentPage * pageSize, total)
  const totalPages = Math.ceil(total / pageSize)
  
  return {
    total,
    start,
    end,
    totalPages,
    currentPage,
    pageSize
  }
})

// 选择状态
const selectionState = computed(() => {
  const { selectedRows } = tableConfig.selection
  const currentPageIds = paginatedData.value.map(row => row.id)
  
  const selectedInCurrentPage = selectedRows.filter(id => 
    currentPageIds.includes(id)
  ).length
  
  return {
    selectedCount: selectedRows.length,
    allSelected: selectedInCurrentPage === currentPageIds.length && currentPageIds.length > 0,
    partialSelected: selectedInCurrentPage > 0 && selectedInCurrentPage < currentPageIds.length
  }
})

// ==================== 方法 ====================

// 生成模拟数据
function generateMockData() {
  const departments = ['技术部', '销售部', '市场部', '人事部', '财务部']
  const positions = ['经理', '主管', '专员', '助理', '总监', '总经理']
  const statuses = ['在职', '离职', '休假', '试用期']
  const skills = ['JavaScript', 'Vue.js', 'React', 'Node.js', 'Python', 'Java', 'UI设计', '项目管理', '数据分析', '市场营销']
  
  const names = [
    '张三', '李四', '王五', '赵六', '钱七', '孙八', '周九', '吴十',
    '刘一', '陈二', '杨三', '黄四', '朱五', '徐六', '何七', '吕八',
    '施九', '张十', '孔十一', '曹十二', '严十三', '华十四', '金十五', '魏十六'
  ]
  
  const data = []
  
  for (let i = 1; i <= 100; i++) {
    const name = names[Math.floor(Math.random() * names.length)]
    const department = departments[Math.floor(Math.random() * departments.length)]
    const position = positions[Math.floor(Math.random() * positions.length)]
    const status = statuses[Math.floor(Math.random() * statuses.length)]
    
    const age = Math.floor(Math.random() * 40) + 22
    const salary = Math.floor(Math.random() * 30000) + 8000
    const performance = Math.floor(Math.random() * 5) + 1
    
    const hireDate = new Date(
      2020 + Math.floor(Math.random() * 4),
      Math.floor(Math.random() * 12),
      Math.floor(Math.random() * 28) + 1
    ).toISOString().split('T')[0]
    
    const userSkills = []
    const skillCount = Math.floor(Math.random() * 4) + 1
    for (let j = 0; j < skillCount; j++) {
      const skill = skills[Math.floor(Math.random() * skills.length)]
      if (!userSkills.includes(skill)) {
        userSkills.push(skill)
      }
    }
    
    data.push({
      id: i,
      name: `${name}${i}`,
      email: `${name.toLowerCase()}${i}@company.com`,
      age,
      department,
      position,
      salary,
      hireDate,
      status,
      performance,
      skills: userSkills
    })
  }
  
  tableData.value = data
  updateStatistics()
}

// 排序处理
function handleSort(column) {
  if (tableConfig.sorting.column === column) {
    // 切换排序方向
    tableConfig.sorting.direction = tableConfig.sorting.direction === 'asc' ? 'desc' : 'asc'
  } else {
    // 新列排序
    tableConfig.sorting.column = column
    tableConfig.sorting.direction = 'asc'
  }
  
  // 重置到第一页
  tableConfig.pagination.currentPage = 1
}

// 过滤处理
function handleFilter(column, filter) {
  if (!tableConfig.filters.columns[column]) {
    tableConfig.filters.columns[column] = {}
  }
  
  Object.assign(tableConfig.filters.columns[column], filter)
  
  // 重置到第一页
  tableConfig.pagination.currentPage = 1
}

// 清除过滤
function clearFilter(column) {
  if (column) {
    delete tableConfig.filters.columns[column]
  } else {
    tableConfig.filters.columns = {}
    tableConfig.filters.global = ''
  }
  
  tableConfig.pagination.currentPage = 1
}

// 全局搜索
function handleGlobalSearch(query) {
  tableConfig.filters.global = query
  tableConfig.pagination.currentPage = 1
}

// 分页处理
function handlePageChange(page) {
  tableConfig.pagination.currentPage = page
}

function handlePageSizeChange(size) {
  tableConfig.pagination.pageSize = size
  tableConfig.pagination.currentPage = 1
}

// 选择处理
function handleSelectRow(row, selected) {
  const { selectedRows } = tableConfig.selection
  const index = selectedRows.indexOf(row.id)
  
  if (selected && index === -1) {
    selectedRows.push(row.id)
  } else if (!selected && index > -1) {
    selectedRows.splice(index, 1)
  }
}

function handleSelectAll(selected) {
  const currentPageIds = paginatedData.value.map(row => row.id)
  
  if (selected) {
    currentPageIds.forEach(id => {
      if (!tableConfig.selection.selectedRows.includes(id)) {
        tableConfig.selection.selectedRows.push(id)
      }
    })
  } else {
    tableConfig.selection.selectedRows = tableConfig.selection.selectedRows.filter(id =>
      !currentPageIds.includes(id)
    )
  }
}

function clearSelection() {
  tableConfig.selection.selectedRows = []
}

// 高级搜索
function addAdvancedCondition() {
  advancedSearch.conditions.push({
    id: advancedSearch.nextId++,
    column: 'name',
    operator: 'contains',
    value: '',
    logic: 'and'
  })
}

function removeAdvancedCondition(id) {
  const index = advancedSearch.conditions.findIndex(c => c.id === id)
  if (index > -1 && advancedSearch.conditions.length > 1) {
    advancedSearch.conditions.splice(index, 1)
  }
}

function clearAdvancedSearch() {
  advancedSearch.conditions = [
    {
      id: 1,
      column: 'name',
      operator: 'contains',
      value: '',
      logic: 'and'
    }
  ]
  advancedSearch.nextId = 2
}

function evaluateAdvancedConditions(row, conditions) {
  if (conditions.length === 0) return true
  
  let result = evaluateCondition(row, conditions[0])
  
  for (let i = 1; i < conditions.length; i++) {
    const condition = conditions[i]
    const conditionResult = evaluateCondition(row, condition)
    
    if (condition.logic === 'and') {
      result = result && conditionResult
    } else {
      result = result || conditionResult
    }
  }
  
  return result
}

function evaluateCondition(row, condition) {
  const { column, operator, value } = condition
  const columnConfig = tableConfig.columns.find(col => col.key === column)
  const rowValue = row[column]
  
  if (!value && value !== 0) return true
  
  switch (columnConfig.type) {
    case 'text':
      switch (operator) {
        case 'contains':
          return String(rowValue).toLowerCase().includes(String(value).toLowerCase())
        case 'equals':
          return rowValue === value
        case 'startsWith':
          return String(rowValue).toLowerCase().startsWith(String(value).toLowerCase())
        case 'endsWith':
          return String(rowValue).toLowerCase().endsWith(String(value).toLowerCase())
        default:
          return true
      }
      
    case 'number':
    case 'currency':
    case 'rating':
      const numValue = Number(value)
      const numRowValue = Number(rowValue)
      switch (operator) {
        case 'equals':
          return numRowValue === numValue
        case 'greater':
          return numRowValue > numValue
        case 'greaterEqual':
          return numRowValue >= numValue
        case 'less':
          return numRowValue < numValue
        case 'lessEqual':
          return numRowValue <= numValue
        default:
          return true
      }
      
    default:
      return true
  }
}

// 导出功能
function showExportDialog() {
  exportConfig.visible = true
  exportConfig.columns = tableConfig.columns
    .filter(col => col.type !== 'actions')
    .map(col => ({ key: col.key, title: col.title, selected: true }))
}

function exportData() {
  let dataToExport = exportConfig.selectedOnly 
    ? tableData.value.filter(row => tableConfig.selection.selectedRows.includes(row.id))
    : sortedData.value
  
  const selectedColumns = exportConfig.columns.filter(col => col.selected)
  
  if (exportConfig.format === 'csv') {
    exportToCSV(dataToExport, selectedColumns)
  } else if (exportConfig.format === 'json') {
    exportToJSON(dataToExport, selectedColumns)
  }
  
  exportConfig.visible = false
}

function exportToCSV(data, columns) {
  let csv = ''
  
  // 添加表头
  if (exportConfig.includeHeaders) {
    csv += columns.map(col => col.title).join(',') + '\n'
  }
  
  // 添加数据
  data.forEach(row => {
    const rowData = columns.map(col => {
      let value = row[col.key]
      if (Array.isArray(value)) {
        value = value.join(';')
      }
      return `"${value}"`
    })
    csv += rowData.join(',') + '\n'
  })
  
  // 下载文件
  const blob = new Blob([csv], { type: 'text/csv;charset=utf-8;' })
  const link = document.createElement('a')
  link.href = URL.createObjectURL(blob)
  link.download = `table_data_${new Date().toISOString().split('T')[0]}.csv`
  link.click()
}

function exportToJSON(data, columns) {
  const exportData = data.map(row => {
    const filteredRow = {}
    columns.forEach(col => {
      filteredRow[col.key] = row[col.key]
    })
    return filteredRow
  })
  
  const json = JSON.stringify(exportData, null, 2)
  const blob = new Blob([json], { type: 'application/json' })
  const link = document.createElement('a')
  link.href = URL.createObjectURL(blob)
  link.download = `table_data_${new Date().toISOString().split('T')[0]}.json`
  link.click()
}

// 统计分析
function updateStatistics() {
  const data = tableData.value
  
  statistics.data = {
    total: data.length,
    departments: countByField(data, 'department'),
    positions: countByField(data, 'position'),
    statuses: countByField(data, 'status'),
    ageDistribution: getAgeDistribution(data),
    salaryStats: getSalaryStats(data),
    performanceStats: getPerformanceStats(data)
  }
}

function countByField(data, field) {
  const counts = {}
  data.forEach(item => {
    const value = item[field]
    counts[value] = (counts[value] || 0) + 1
  })
  return counts
}

function getAgeDistribution(data) {
  const ranges = {
    '20-29': 0,
    '30-39': 0,
    '40-49': 0,
    '50-59': 0,
    '60+': 0
  }
  
  data.forEach(item => {
    const age = item.age
    if (age < 30) ranges['20-29']++
    else if (age < 40) ranges['30-39']++
    else if (age < 50) ranges['40-49']++
    else if (age < 60) ranges['50-59']++
    else ranges['60+']++
  })
  
  return ranges
}

function getSalaryStats(data) {
  const salaries = data.map(item => item.salary).sort((a, b) => a - b)
  const total = salaries.reduce((sum, salary) => sum + salary, 0)
  
  return {
    min: Math.min(...salaries),
    max: Math.max(...salaries),
    average: Math.round(total / salaries.length),
    median: salaries[Math.floor(salaries.length / 2)]
  }
}

function getPerformanceStats(data) {
  const performances = data.map(item => item.performance)
  const counts = [0, 0, 0, 0, 0]
  
  performances.forEach(p => {
    counts[p - 1]++
  })
  
  const average = performances.reduce((sum, p) => sum + p, 0) / performances.length
  
  return {
    average: average.toFixed(1),
    distribution: counts.map((count, index) => ({
      rating: index + 1,
      count,
      percentage: Math.round((count / data.length) * 100)
    }))
  }
}

// 行操作
function editRow(row) {
  alert(`编辑 ${row.name} 的信息`)
}

function deleteRow(row) {
  if (confirm(`确定要删除 ${row.name} 吗？`)) {
    const index = tableData.value.findIndex(item => item.id === row.id)
    if (index > -1) {
      tableData.value.splice(index, 1)
      
      // 从选择列表中移除
      const selectedIndex = tableConfig.selection.selectedRows.indexOf(row.id)
      if (selectedIndex > -1) {
        tableConfig.selection.selectedRows.splice(selectedIndex, 1)
      }
      
      updateStatistics()
    }
  }
}

function viewRow(row) {
  alert(`查看 ${row.name} 的详细信息:\n${JSON.stringify(row, null, 2)}`)
}

// 批量操作
function batchDelete() {
  if (tableConfig.selection.selectedRows.length === 0) {
    alert('请先选择要删除的行')
    return
  }
  
  if (confirm(`确定要删除选中的 ${tableConfig.selection.selectedRows.length} 条记录吗？`)) {
    tableData.value = tableData.value.filter(row => 
      !tableConfig.selection.selectedRows.includes(row.id)
    )
    
    clearSelection()
    updateStatistics()
  }
}

function batchExport() {
  if (tableConfig.selection.selectedRows.length === 0) {
    alert('请先选择要导出的行')
    return
  }
  
  exportConfig.selectedOnly = true
  showExportDialog()
}

// 工具函数
function formatCurrency(value) {
  return `¥${value.toLocaleString()}`
}

function formatDate(dateString) {
  return new Date(dateString).toLocaleDateString()
}

function getStars(rating) {
  return '★'.repeat(rating) + '☆'.repeat(5 - rating)
}

// 确保过滤器对象存在
function ensureColumnFilter(columnKey, defaultValue = {}) {
  if (!tableConfig.filters.columns[columnKey]) {
    tableConfig.filters.columns[columnKey] = { ...defaultValue }
  }
  return tableConfig.filters.columns[columnKey]
}

// 获取列过滤器的特定属性
function getColumnFilterProp(columnKey, prop, defaultValue = '') {
  const filter = ensureColumnFilter(columnKey)
  if (filter[prop] === undefined) {
    filter[prop] = defaultValue
  }
  return filter[prop]
}

// 设置列过滤器的特定属性
function setColumnFilterProp(columnKey, prop, value) {
  const filter = ensureColumnFilter(columnKey)
  filter[prop] = value
}

// 处理过滤选项变化
function handleFilterOptionChange(columnKey, option, checked) {
  // 确保过滤器对象存在
  const filter = ensureColumnFilter(columnKey, { value: [] })
  
  // 确保value是数组
  if (!Array.isArray(filter.value)) {
    filter.value = []
  }
  
  const values = filter.value
  const index = values.indexOf(option)
  
  if (checked && index === -1) {
    values.push(option)
  } else if (!checked && index > -1) {
    values.splice(index, 1)
  }
}

// 生成分页页码数组
function getPageNumbers() {
  const { currentPage, totalPages } = paginationInfo.value
  const pages = []
  
  if (totalPages <= 7) {
    // 如果总页数小于等于7，显示所有页码
    for (let i = 1; i <= totalPages; i++) {
      pages.push(i)
    }
  } else {
    // 总页数大于7时，智能显示页码
    if (currentPage <= 4) {
      // 当前页在前4页
      for (let i = 1; i <= 5; i++) {
        pages.push(i)
      }
      pages.push('...')
      pages.push(totalPages)
    } else if (currentPage >= totalPages - 3) {
      // 当前页在后4页
      pages.push(1)
      pages.push('...')
      for (let i = totalPages - 4; i <= totalPages; i++) {
        pages.push(i)
      }
    } else {
      // 当前页在中间
      pages.push(1)
      pages.push('...')
      for (let i = currentPage - 1; i <= currentPage + 1; i++) {
        pages.push(i)
      }
      pages.push('...')
      pages.push(totalPages)
    }
  }
  
  return pages
}

// 初始化
onMounted(() => {
  generateMockData()
})

// 监听过滤变化
watch(() => tableConfig.filters, () => {
  updateStatistics()
}, { deep: true })
</script>

<template>
  <div class="filterable-table">
    <h2>📊 可过滤排序的数据表格</h2>
    
    <!-- 工具栏 -->
    <div class="toolbar">
      <div class="toolbar-left">
        <div class="global-search">
          <input 
            v-model="tableConfig.filters.global"
            @input="handleGlobalSearch($event.target.value)"
            placeholder="全局搜索..."
            class="search-input"
          >
          <span class="search-icon">🔍</span>
        </div>
        
        <button @click="advancedSearch.visible = !advancedSearch.visible" class="advanced-search-btn">
          {{ advancedSearch.visible ? '隐藏' : '高级' }}搜索
        </button>
        
        <button @click="clearFilter()" class="clear-filter-btn">
          清除过滤
        </button>
      </div>
      
      <div class="toolbar-right">
        <button @click="statistics.visible = !statistics.visible" class="stats-btn">
          📈 统计分析
        </button>
        
        <button @click="showExportDialog" class="export-btn">
          📤 导出数据
        </button>
        
        <button @click="generateMockData" class="refresh-btn">
          🔄 刷新数据
        </button>
      </div>
    </div>
    
    <!-- 高级搜索 -->
    <div v-if="advancedSearch.visible" class="advanced-search">
      <div class="advanced-search-header">
        <h3>高级搜索条件</h3>
        <div class="advanced-search-actions">
          <button @click="addAdvancedCondition" class="add-condition-btn">添加条件</button>
          <button @click="clearAdvancedSearch" class="clear-conditions-btn">清除条件</button>
        </div>
      </div>
      
      <div class="search-conditions">
        <div v-for="(condition, index) in advancedSearch.conditions" :key="condition.id" class="condition-row">
          <select v-if="index > 0" v-model="condition.logic" class="logic-select">
            <option value="and">且</option>
            <option value="or">或</option>
          </select>
          
          <select v-model="condition.column" class="column-select">
            <option v-for="col in tableConfig.columns.filter(c => c.filterable)" :key="col.key" :value="col.key">
              {{ col.title }}
            </option>
          </select>
          
          <select v-model="condition.operator" class="operator-select">
            <option value="contains">包含</option>
            <option value="equals">等于</option>
            <option value="startsWith">开始于</option>
            <option value="endsWith">结束于</option>
            <option value="greater">大于</option>
            <option value="less">小于</option>
          </select>
          
          <input v-model="condition.value" placeholder="搜索值" class="value-input">
          
          <button @click="removeAdvancedCondition(condition.id)" :disabled="advancedSearch.conditions.length <= 1" class="remove-condition-btn">
            删除
          </button>
        </div>
      </div>
    </div>
    
    <!-- 选择信息 -->
    <div v-if="tableConfig.selection.enabled && tableConfig.selection.selectedRows.length > 0" class="selection-info">
      <div class="selection-summary">
        已选择 {{ tableConfig.selection.selectedRows.length }} 项
      </div>
      <div class="selection-actions">
        <button @click="batchDelete" class="batch-delete-btn">批量删除</button>
        <button @click="batchExport" class="batch-export-btn">批量导出</button>
        <button @click="clearSelection" class="clear-selection-btn">清除选择</button>
      </div>
    </div>
    
    <!-- 统计面板 -->
    <div v-if="statistics.visible" class="statistics-panel">
      <div class="statistics-header">
        <h3>📊 数据统计分析</h3>
        <button @click="statistics.visible = false" class="close-stats-btn">×</button>
      </div>
      
      <div class="statistics-content">
        <div class="stat-section">
          <h4>基本信息</h4>
          <div class="stat-grid">
            <div class="stat-item">
              <span class="stat-label">总记录数:</span>
              <span class="stat-value">{{ statistics.data.total }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">当前显示:</span>
              <span class="stat-value">{{ filteredData.length }}</span>
            </div>
          </div>
        </div>
        
        <div class="stat-section">
          <h4>部门分布</h4>
          <div class="stat-list">
            <div v-for="(count, dept) in statistics.data.departments" :key="dept" class="stat-bar">
              <span class="stat-name">{{ dept }}</span>
              <div class="stat-progress">
                <div class="stat-fill" :style="{ width: (count / statistics.data.total * 100) + '%' }"></div>
              </div>
              <span class="stat-count">{{ count }}</span>
            </div>
          </div>
        </div>
        
        <div class="stat-section">
          <h4>薪资统计</h4>
          <div class="stat-grid">
            <div class="stat-item">
              <span class="stat-label">平均薪资:</span>
              <span class="stat-value">{{ formatCurrency(statistics.data.salaryStats?.average || 0) }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">最高薪资:</span>
              <span class="stat-value">{{ formatCurrency(statistics.data.salaryStats?.max || 0) }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">最低薪资:</span>
              <span class="stat-value">{{ formatCurrency(statistics.data.salaryStats?.min || 0) }}</span>
            </div>
            <div class="stat-item">
              <span class="stat-label">中位数:</span>
              <span class="stat-value">{{ formatCurrency(statistics.data.salaryStats?.median || 0) }}</span>
            </div>
          </div>
        </div>
        
        <div class="stat-section">
          <h4>绩效分布</h4>
          <div class="performance-chart">
            <div v-for="item in statistics.data.performanceStats?.distribution" :key="item.rating" class="performance-bar">
              <span class="performance-rating">{{ getStars(item.rating) }}</span>
              <div class="performance-progress">
                <div class="performance-fill" :style="{ width: item.percentage + '%' }"></div>
              </div>
              <span class="performance-percentage">{{ item.percentage }}%</span>
            </div>
          </div>
        </div>
      </div>
    </div>
    
    <!-- 表格容器 -->
    <div class="table-container">
      <div class="table-wrapper">
        <table class="data-table" :class="{
          striped: tableConfig.display.striped,
          bordered: tableConfig.display.bordered,
          hover: tableConfig.display.hover,
          compact: tableConfig.display.compact
        }">
          <!-- 表头 -->
          <thead v-if="tableConfig.display.showHeader">
            <tr class="header-row">
              <!-- 选择列 -->
              <th v-if="tableConfig.selection.enabled" class="selection-header">
                <input 
                  type="checkbox" 
                  :checked="selectionState.allSelected"
                  :indeterminate.prop="selectionState.partialSelected"
                  @change="handleSelectAll($event.target.checked)"
                >
              </th>
              
              <!-- 数据列 -->
              <th v-for="column in tableConfig.columns" :key="column.key" 
                  :class="['column-header', { sortable: column.sortable }]"
                  :style="{ width: column.width + 'px' }">
                <div class="header-content">
                  <span class="header-title" @click="column.sortable && handleSort(column.key)">
                    {{ column.title }}
                    <span v-if="column.sortable" class="sort-indicator">
                      <span :class="['sort-arrow', 'sort-asc', { 
                        active: tableConfig.sorting.column === column.key && tableConfig.sorting.direction === 'asc' 
                      }]">↑</span>
                      <span :class="['sort-arrow', 'sort-desc', { 
                        active: tableConfig.sorting.column === column.key && tableConfig.sorting.direction === 'desc' 
                      }]">↓</span>
                    </span>
                  </span>
                  
                  <!-- 列过滤 -->
                  <div v-if="column.filterable" class="column-filter">
                    <div class="filter-trigger" @click.stop>
                      🔽
                      <div class="filter-dropdown">
                        <!-- 文本过滤 -->
                        <div v-if="column.type === 'text'" class="text-filter">
                          <select :value="getColumnFilterProp(column.key, 'operator', 'contains')"
                                  @change="setColumnFilterProp(column.key, 'operator', $event.target.value)">
                            <option value="contains">包含</option>
                            <option value="equals">等于</option>
                            <option value="startsWith">开始于</option>
                          </select>
                          <input :value="getColumnFilterProp(column.key, 'value', '')"
                                 @input="setColumnFilterProp(column.key, 'value', $event.target.value)"
                                 placeholder="过滤值">
                        </div>
                        
                        <!-- 数字过滤 -->
                        <div v-else-if="['number', 'currency', 'rating'].includes(column.type)" class="number-filter">
                          <select :value="getColumnFilterProp(column.key, 'operator', 'equals')"
                                  @change="setColumnFilterProp(column.key, 'operator', $event.target.value)">
                            <option value="equals">等于</option>
                            <option value="greater">大于</option>
                            <option value="less">小于</option>
                            <option value="between">范围</option>
                          </select>
                          <input v-if="getColumnFilterProp(column.key, 'operator', 'equals') !== 'between'"
                                 :value="getColumnFilterProp(column.key, 'value', '')"
                                 @input="setColumnFilterProp(column.key, 'value', $event.target.value)"
                                 type="number" placeholder="数值">
                          <div v-else class="range-inputs">
                            <input :value="getColumnFilterProp(column.key, 'min', '')"
                                   @input="setColumnFilterProp(column.key, 'min', $event.target.value)"
                                   type="number" placeholder="最小值">
                            <input :value="getColumnFilterProp(column.key, 'max', '')"
                                   @input="setColumnFilterProp(column.key, 'max', $event.target.value)"
                                   type="number" placeholder="最大值">
                          </div>
                        </div>
                        
                        <!-- 选择过滤 -->
                        <div v-else-if="column.type === 'select'" class="select-filter">
                          <div v-for="option in column.options" :key="option" class="filter-option">
                            <label>
                              <input type="checkbox" 
                                     :value="option"
                                     :checked="getColumnFilterProp(column.key, 'value', []).includes(option)"
                                     @change="handleFilterOptionChange(column.key, option, $event.target.checked)">
                              {{ option }}
                            </label>
                          </div>
                        </div>
                        
                        <!-- 日期过滤 -->
                        <div v-else-if="column.type === 'date'" class="date-filter">
                          <select :value="getColumnFilterProp(column.key, 'operator', 'equals')"
                                  @change="setColumnFilterProp(column.key, 'operator', $event.target.value)">
                            <option value="equals">等于</option>
                            <option value="after">之后</option>
                            <option value="before">之前</option>
                          </select>
                          <input :value="getColumnFilterProp(column.key, 'value', '')"
                                 @input="setColumnFilterProp(column.key, 'value', $event.target.value)"
                                 type="date">
                        </div>
                        
                        <!-- 标签过滤 -->
                        <div v-else-if="column.type === 'tags'" class="tags-filter">
                          <input :value="getColumnFilterProp(column.key, 'value', '')"
                                 @input="setColumnFilterProp(column.key, 'value', $event.target.value)"
                                 placeholder="搜索标签">
                        </div>
                        
                        <div class="filter-actions">
                          <button @click="clearFilter(column.key)" class="clear-filter-btn">清除</button>
                        </div>
                      </div>
                    </div>
                  </div>
                </div>
              </th>
            </tr>
          </thead>
          
          <!-- 表体 -->
          <tbody>
            <tr v-if="tableConfig.display.loading" class="loading-row">
              <td :colspan="tableConfig.columns.length + (tableConfig.selection.enabled ? 1 : 0)">
                <div class="loading-indicator">
                  <span class="loading-spinner">⌛</span>
                  <span>加载中...</span>
                </div>
              </td>
            </tr>
            
            <tr v-else-if="paginatedData.length === 0" class="empty-row">
              <td :colspan="tableConfig.columns.length + (tableConfig.selection.enabled ? 1 : 0)">
                <div class="empty-message">
                  <span class="empty-icon">📭</span>
                  <span>暂无数据</span>
                </div>
              </td>
            </tr>
            
            <tr v-else v-for="row in paginatedData" :key="row.id" class="data-row">
              <!-- 选择列 -->
              <td v-if="tableConfig.selection.enabled" class="selection-cell">
                <input 
                  type="checkbox" 
                  :checked="tableConfig.selection.selectedRows.includes(row.id)"
                  @change="handleSelectRow(row, $event.target.checked)"
                >
              </td>
              
              <!-- 数据列 -->
              <td v-for="column in tableConfig.columns" :key="column.key" 
                  :class="['data-cell', column.type]">
                
                <!-- 文本类型 -->
                <template v-if="column.type === 'text'">
                  {{ row[column.key] }}
                </template>
                
                <!-- 数字类型 -->
                <template v-else-if="column.type === 'number'">
                  {{ row[column.key] }}
                </template>
                
                <!-- 货币类型 -->
                <template v-else-if="column.type === 'currency'">
                  {{ formatCurrency(row[column.key]) }}
                </template>
                
                <!-- 日期类型 -->
                <template v-else-if="column.type === 'date'">
                  {{ formatDate(row[column.key]) }}
                </template>
                
                <!-- 评分类型 -->
                <template v-else-if="column.type === 'rating'">
                  <span class="rating-stars">{{ getStars(row[column.key]) }}</span>
                  <span class="rating-number">({{ row[column.key] }})</span>
                </template>
                
                <!-- 标签类型 -->
                <template v-else-if="column.type === 'tags'">
                  <div class="tags-container">
                    <span v-for="tag in row[column.key]" :key="tag" class="tag">
                      {{ tag }}
                    </span>
                  </div>
                </template>
                
                <!-- 操作列 -->
                <template v-else-if="column.type === 'actions'">
                  <div class="actions-container">
                    <button @click="viewRow(row)" class="action-btn view-btn" title="查看">👁️</button>
                    <button @click="editRow(row)" class="action-btn edit-btn" title="编辑">✏️</button>
                    <button @click="deleteRow(row)" class="action-btn delete-btn" title="删除">🗑️</button>
                  </div>
                </template>
              </td>
            </tr>
          </tbody>
        </table>
      </div>
    </div>
    
    <!-- 分页器 -->
    <div v-if="tableConfig.display.showFooter" class="pagination">
      <div class="pagination-info">
        <span v-if="tableConfig.pagination.showTotal">
          显示第 {{ paginationInfo.start }} - {{ paginationInfo.end }} 条，
          共 {{ paginationInfo.total }} 条记录
        </span>
      </div>
      
      <div class="pagination-controls">
        <!-- 页面大小选择 -->
        <div v-if="tableConfig.pagination.showSizeChanger" class="page-size-selector">
          <span>每页显示</span>
          <select v-model="tableConfig.pagination.pageSize" 
                  @change="handlePageSizeChange($event.target.value)">
            <option v-for="size in tableConfig.pagination.pageSizes" :key="size" :value="size">
              {{ size }}
            </option>
          </select>
          <span>条</span>
        </div>
        
        <!-- 页码控制 -->
        <div class="page-controls">
          <button @click="handlePageChange(1)" 
                  :disabled="paginationInfo.currentPage === 1"
                  class="page-btn">
            首页
          </button>
          
          <button @click="handlePageChange(paginationInfo.currentPage - 1)" 
                  :disabled="paginationInfo.currentPage === 1"
                  class="page-btn">
            上一页
          </button>
          
          <div class="page-numbers">
            <button v-for="page in getPageNumbers()" :key="page"
                    @click="page !== '...' && handlePageChange(page)"
                    :class="['page-number', { 
                      active: page === paginationInfo.currentPage,
                      disabled: page === '...'
                    }]">
              {{ page }}
            </button>
          </div>
          
          <button @click="handlePageChange(paginationInfo.currentPage + 1)" 
                  :disabled="paginationInfo.currentPage === paginationInfo.totalPages"
                  class="page-btn">
            下一页
          </button>
          
          <button @click="handlePageChange(paginationInfo.totalPages)" 
                  :disabled="paginationInfo.currentPage === paginationInfo.totalPages"
                  class="page-btn">
            末页
          </button>
        </div>
        
        <!-- 快速跳转 -->
        <div v-if="tableConfig.pagination.showQuickJumper" class="quick-jumper">
          <span>跳至</span>
          <input type="number" 
                 :value="paginationInfo.currentPage"
                 @keyup.enter="handlePageChange(parseInt($event.target.value))"
                 min="1" 
                 :max="paginationInfo.totalPages">
          <span>页</span>
        </div>
      </div>
    </div>
    
    <!-- 导出对话框 -->
    <div v-if="exportConfig.visible" class="modal-overlay" @click="exportConfig.visible = false">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h3>导出数据</h3>
          <button @click="exportConfig.visible = false" class="modal-close">×</button>
        </div>
        
        <div class="modal-body">
          <div class="export-options">
            <div class="option-group">
              <label>导出格式:</label>
              <div class="radio-group">
                <label class="radio-label">
                  <input type="radio" v-model="exportConfig.format" value="csv">
                  CSV
                </label>
                <label class="radio-label">
                  <input type="radio" v-model="exportConfig.format" value="json">
                  JSON
                </label>
              </div>
            </div>
            
            <div class="option-group">
              <label class="checkbox-label">
                <input type="checkbox" v-model="exportConfig.includeHeaders">
                包含表头
              </label>
            </div>
            
            <div class="option-group">
              <label class="checkbox-label">
                <input type="checkbox" v-model="exportConfig.selectedOnly">
                仅导出选中项
              </label>
            </div>
            
            <div class="option-group">
              <label>选择列:</label>
              <div class="columns-selector">
                <label v-for="column in exportConfig.columns" :key="column.key" class="checkbox-label">
                  <input type="checkbox" v-model="column.selected">
                  {{ column.title }}
                </label>
              </div>
            </div>
          </div>
        </div>
        
        <div class="modal-footer">
          <button @click="exportData" class="confirm-btn">导出</button>
          <button @click="exportConfig.visible = false" class="cancel-btn">取消</button>
        </div>
      </div>
    </div>
  </div>
</template>

<style scoped>
.filterable-table {
  padding: 20px;
  max-width: 100%;
  margin: 0 auto;
}

/* 工具栏样式 */
.toolbar {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 20px;
  padding: 15px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  flex-wrap: wrap;
  gap: 15px;
}

.toolbar-left,
.toolbar-right {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}

.global-search {
  position: relative;
  display: flex;
  align-items: center;
}

.search-input {
  padding: 8px 35px 8px 12px;
  border: 2px solid #ddd;
  border-radius: 20px;
  font-size: 14px;
  width: 250px;
  transition: border-color 0.3s;
}

.search-input:focus {
  outline: none;
  border-color: #42b883;
}

.search-icon {
  position: absolute;
  right: 12px;
  color: #666;
}

/* 按钮样式 */
button {
  padding: 8px 16px;
  border: none;
  border-radius: 4px;
  font-size: 14px;
  cursor: pointer;
  transition: all 0.3s;
}

.advanced-search-btn {
  background: #2196f3;
  color: white;
}

.clear-filter-btn {
  background: #ff9800;
  color: white;
}

.stats-btn {
  background: #9c27b0;
  color: white;
}

.export-btn {
  background: #4caf50;
  color: white;
}

.refresh-btn {
  background: #607d8b;
  color: white;
}

button:hover {
  transform: translateY(-1px);
  box-shadow: 0 2px 4px rgba(0, 0, 0, 0.2);
}

/* 高级搜索样式 */
.advanced-search {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
  padding: 20px;
}

.advanced-search-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 15px;
}

.advanced-search-actions {
  display: flex;
  gap: 10px;
}

.add-condition-btn {
  background: #4caf50;
  color: white;
}

.clear-conditions-btn {
  background: #f44336;
  color: white;
}

.search-conditions {
  display: flex;
  flex-direction: column;
  gap: 10px;
}

.condition-row {
  display: flex;
  align-items: center;
  gap: 10px;
  flex-wrap: wrap;
}

.logic-select,
.column-select,
.operator-select {
  padding: 6px 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  background: white;
}

.value-input {
  padding: 6px 10px;
  border: 1px solid #ddd;
  border-radius: 4px;
  flex: 1;
  min-width: 150px;
}

.remove-condition-btn {
  background: #f44336;
  color: white;
  padding: 6px 10px;
}

.remove-condition-btn:disabled {
  background: #ccc;
  cursor: not-allowed;
}

/* 选择信息样式 */
.selection-info {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 10px 15px;
  background: #e3f2fd;
  border: 1px solid #2196f3;
  border-radius: 4px;
  margin-bottom: 15px;
}

.selection-actions {
  display: flex;
  gap: 10px;
}

.batch-delete-btn {
  background: #f44336;
  color: white;
}

.batch-export-btn {
  background: #4caf50;
  color: white;
}

.clear-selection-btn {
  background: #757575;
  color: white;
}

/* 统计面板样式 */
.statistics-panel {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  margin-bottom: 20px;
  max-height: 400px;
  overflow-y: auto;
}

.statistics-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 20px;
  border-bottom: 1px solid #ddd;
}

.close-stats-btn {
  background: none;
  border: none;
  font-size: 18px;
  cursor: pointer;
  padding: 5px;
}

.statistics-content {
  padding: 20px;
}

.stat-section {
  margin-bottom: 20px;
}

.stat-section h4 {
  margin: 0 0 10px 0;
  color: #333;
}

.stat-grid {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 10px;
}

.stat-item {
  display: flex;
  justify-content: space-between;
  padding: 8px;
  background: #f5f5f5;
  border-radius: 4px;
}

.stat-label {
  color: #666;
}

.stat-value {
  font-weight: bold;
  color: #333;
}

.stat-list {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.stat-bar {
  display: flex;
  align-items: center;
  gap: 10px;
}

.stat-name {
  min-width: 60px;
  font-size: 14px;
}

.stat-progress {
  flex: 1;
  height: 20px;
  background: #f0f0f0;
  border-radius: 10px;
  overflow: hidden;
}

.stat-fill {
  height: 100%;
  background: linear-gradient(45deg, #42b883, #66d9a2);
  transition: width 0.3s;
}

.stat-count {
  min-width: 30px;
  text-align: right;
  font-weight: bold;
}

.performance-chart {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.performance-bar {
  display: flex;
  align-items: center;
  gap: 10px;
}

.performance-rating {
  min-width: 80px;
  color: #ff9800;
}

.performance-progress {
  flex: 1;
  height: 20px;
  background: #f0f0f0;
  border-radius: 10px;
  overflow: hidden;
}

.performance-fill {
  height: 100%;
  background: linear-gradient(45deg, #ff9800, #ffb74d);
  transition: width 0.3s;
}

.performance-percentage {
  min-width: 40px;
  text-align: right;
  font-weight: bold;
}

/* 表格样式 */
.table-container {
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  overflow: hidden;
  margin-bottom: 20px;
}

.table-wrapper {
  overflow-x: auto;
}

.data-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 14px;
}

.data-table.striped tbody tr:nth-child(even) {
  background-color: #f9f9f9;
}

.data-table.bordered {
  border: 1px solid #ddd;
}

.data-table.bordered th,
.data-table.bordered td {
  border: 1px solid #ddd;
}

.data-table.hover tbody tr:hover {
  background-color: #f5f5f5;
}

.data-table.compact th,
.data-table.compact td {
  padding: 6px 8px;
}

/* 表头样式 */
.header-row {
  background: #f8f9fa;
}

.column-header {
  padding: 12px 8px;
  text-align: left;
  font-weight: bold;
  color: #333;
  position: relative;
  user-select: none;
}

.column-header.sortable {
  cursor: pointer;
}

.header-content {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 8px;
}

.header-title {
  display: flex;
  align-items: center;
  gap: 5px;
}

.sort-indicator {
  display: flex;
  flex-direction: column;
  font-size: 10px;
  line-height: 1;
}

.sort-arrow {
  color: #ccc;
  transition: color 0.3s;
}

.sort-arrow.active {
  color: #42b883;
}

/* 列过滤样式 */
.column-filter {
  position: relative;
}

.filter-trigger {
  cursor: pointer;
  color: #666;
  font-size: 12px;
  position: relative;
}

.filter-dropdown {
  position: absolute;
  top: 100%;
  right: 0;
  background: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  box-shadow: 0 4px 12px rgba(0, 0, 0, 0.15);
  padding: 10px;
  min-width: 200px;
  z-index: 1000;
  display: none;
}

.filter-trigger:hover .filter-dropdown {
  display: block;
}

.text-filter,
.number-filter,
.date-filter,
.tags-filter {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.range-inputs {
  display: flex;
  gap: 5px;
}

.range-inputs input {
  flex: 1;
}

.select-filter {
  max-height: 150px;
  overflow-y: auto;
}

.filter-option {
  padding: 4px 0;
}

.filter-option label {
  display: flex;
  align-items: center;
  gap: 5px;
  cursor: pointer;
}

.filter-actions {
  border-top: 1px solid #eee;
  padding-top: 8px;
  margin-top: 8px;
}

.clear-filter-btn {
  background: #f44336;
  color: white;
  padding: 4px 8px;
  font-size: 12px;
}

/* 选择列样式 */
.selection-header,
.selection-cell {
  width: 40px;
  text-align: center;
  padding: 8px;
}

/* 数据行样式 */
.data-row {
  transition: background-color 0.3s;
}

.data-cell {
  padding: 8px;
  vertical-align: middle;
}

.data-cell.currency {
  text-align: right;
  font-weight: bold;
  color: #4caf50;
}

.data-cell.rating {
  display: flex;
  align-items: center;
  gap: 5px;
}

.rating-stars {
  color: #ff9800;
}

.rating-number {
  color: #666;
  font-size: 12px;
}

.tags-container {
  display: flex;
  flex-wrap: wrap;
  gap: 4px;
}

.tag {
  background: #42b883;
  color: white;
  padding: 2px 6px;
  border-radius: 10px;
  font-size: 12px;
}

.actions-container {
  display: flex;
  gap: 4px;
}

.action-btn {
  padding: 4px 6px;
  border: none;
  border-radius: 3px;
  cursor: pointer;
  font-size: 12px;
  transition: all 0.3s;
}

.view-btn {
  background: #2196f3;
}

.edit-btn {
  background: #ff9800;
}

.delete-btn {
  background: #f44336;
}

/* 加载和空数据样式 */
.loading-row,
.empty-row {
  text-align: center;
}

.loading-indicator,
.empty-message {
  display: flex;
  align-items: center;
  justify-content: center;
  gap: 10px;
  padding: 40px;
  color: #666;
}

.loading-spinner {
  animation: spin 1s linear infinite;
}

@keyframes spin {
  from { transform: rotate(0deg); }
  to { transform: rotate(360deg); }
}

.empty-icon {
  font-size: 24px;
}

/* 分页样式 */
.pagination {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 20px;
  background: white;
  border-radius: 8px;
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.1);
  flex-wrap: wrap;
  gap: 15px;
}

.pagination-info {
  color: #666;
  font-size: 14px;
}

.pagination-controls {
  display: flex;
  align-items: center;
  gap: 15px;
  flex-wrap: wrap;
}

.page-size-selector {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 14px;
}

.page-size-selector select {
  padding: 4px 8px;
  border: 1px solid #ddd;
  border-radius: 4px;
}

.page-controls {
  display: flex;
  align-items: center;
  gap: 8px;
}

.page-btn {
  padding: 6px 12px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s;
}

.page-btn:hover:not(:disabled) {
  background: #f5f5f5;
  border-color: #42b883;
}

.page-btn:disabled {
  opacity: 0.5;
  cursor: not-allowed;
}

.page-numbers {
  display: flex;
  gap: 4px;
}

.page-number {
  padding: 6px 10px;
  background: white;
  border: 1px solid #ddd;
  border-radius: 4px;
  cursor: pointer;
  transition: all 0.3s;
  min-width: 36px;
  text-align: center;
}

.page-number:hover:not(.disabled) {
  background: #f5f5f5;
  border-color: #42b883;
}

.page-number.active {
  background: #42b883;
  color: white;
  border-color: #42b883;
}

.page-number.disabled {
  cursor: default;
  background: transparent;
  border: none;
}

.quick-jumper {
  display: flex;
  align-items: center;
  gap: 8px;
  font-size: 14px;
}

.quick-jumper input {
  width: 50px;
  padding: 4px 6px;
  border: 1px solid #ddd;
  border-radius: 4px;
  text-align: center;
}

/* 模态框样式 */
.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0, 0, 0, 0.5);
  display: flex;
  align-items: center;
  justify-content: center;
  z-index: 2000;
}

.modal-content {
  background: white;
  border-radius: 8px;
  box-shadow: 0 4px 16px rgba(0, 0, 0, 0.3);
  max-width: 500px;
  width: 90%;
  max-height: 80vh;
  overflow-y: auto;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 15px 20px;
  border-bottom: 1px solid #ddd;
}

.modal-header h3 {
  margin: 0;
}

.modal-close {
  background: none;
  border: none;
  font-size: 18px;
  cursor: pointer;
  padding: 5px;
}

.modal-body {
  padding: 20px;
}

.export-options {
  display: flex;
  flex-direction: column;
  gap: 15px;
}

.option-group {
  display: flex;
  flex-direction: column;
  gap: 8px;
}

.option-group label {
  font-weight: bold;
  color: #333;
}

.radio-group {
  display: flex;
  gap: 15px;
}

.radio-label,
.checkbox-label {
  display: flex;
  align-items: center;
  gap: 5px;
  cursor: pointer;
}

.columns-selector {
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(150px, 1fr));
  gap: 8px;
  max-height: 150px;
  overflow-y: auto;
  border: 1px solid #ddd;
  padding: 10px;
  border-radius: 4px;
}

.modal-footer {
  display: flex;
  justify-content: flex-end;
  gap: 10px;
  padding: 15px 20px;
  border-top: 1px solid #ddd;
}

.confirm-btn {
  background: #42b883;
  color: white;
}

.cancel-btn {
  background: #757575;
  color: white;
}

/* 响应式设计 */
@media (max-width: 768px) {
  .toolbar {
    flex-direction: column;
    align-items: stretch;
  }
  
  .toolbar-left,
  .toolbar-right {
    justify-content: center;
  }
  
  .search-input {
    width: 100%;
  }
  
  .condition-row {
    flex-direction: column;
    align-items: stretch;
  }
  
  .pagination {
    flex-direction: column;
    align-items: stretch;
  }
  
  .pagination-controls {
    justify-content: center;
  }
  
  .page-controls {
    flex-wrap: wrap;
    justify-content: center;
  }
  
  .table-wrapper {
    overflow-x: scroll;
  }
  
  .modal-content {
    width: 95%;
    margin: 10px;
  }
}
</style>