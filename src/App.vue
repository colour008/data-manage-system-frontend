<template>
  <div class="app-container">
    <!-- 页面头部 -->
    <div class="header-container">
      <el-page-header content="动态数据管理系统" class="page-header" />
    </div>

    <!-- 业务表标识输入区 -->
    <el-card class="card-wrapper table-code-card">
      <el-form-item label="业务表标识" label-width="100px" class="form-item">
        <el-input
            v-model="tableCode"
            placeholder="请输入业务表标识（如：user_data）"
            class="table-code-input"
            prefix-icon="el-icon-folder-opened"
            clearable
        />
        <el-button type="primary" @click="getFieldList" class="btn-refresh" icon="el-icon-refresh">
          加载字段
        </el-button>
      </el-form-item>
    </el-card>

    <!-- 字段配置区域 -->
    <el-card title="🔧 字段配置" class="card-wrapper">
      <el-form :model="fieldForm" inline class="field-form">
        <el-form-item label="字段名称" label-width="80px" class="form-item">
          <el-input
              v-model="fieldForm.name"
              placeholder="如：用户名、年龄"
              prefix-icon="el-icon-menu"
              style="width: 180px"
              clearable
              class="form-input"
          />
        </el-form-item>
        <el-form-item label="字段编码" label-width="80px" class="form-item">
          <el-input
              v-model="fieldForm.code"
              placeholder="如：user_name、age"
              prefix-icon="el-icon-key"
              style="width: 180px"
              clearable
              class="form-input"
          />
        </el-form-item>
        <el-form-item label="字段类型" label-width="80px" class="form-item">
          <el-select
              v-model="fieldForm.type"
              placeholder="选择字段类型"
              style="width: 180px"
              prefix-icon="el-icon-s-tools"
              class="form-select"
          >
            <el-option label="字符串(VARCHAR)" value="VARCHAR" />
            <el-option label="整数(INT)" value="INT" />
            <el-option label="长整型(BIGINT)" value="BIGINT" />
            <el-option label="小数(DECIMAL)" value="DECIMAL" />
            <el-option label="日期(DATE)" value="DATE" />
            <el-option label="日期时间(DATETIME)" value="DATETIME" />
            <el-option label="时间戳(TIMESTAMP)" value="TIMESTAMP" />
            <el-option label="布尔值(BOOLEAN)" value="BOOLEAN" />
            <el-option label="文本(TEXT)" value="TEXT" />
          </el-select>
        </el-form-item>
        <el-form-item>
          <el-button
              type="primary"
              @click="addField"
              icon="el-icon-plus"
              class="btn-add"
          >
            添加字段
          </el-button>
        </el-form-item>
      </el-form>

      <div class="list-container">
        <h4 class="list-title">已配置字段</h4>
        <el-table
            :data="fieldList"
            border
            stripe
            style="width: 100%"
            v-loading="fieldLoading"
            class="data-table"
        >
          <el-table-column prop="fieldName" label="字段名称" width="150" />
          <el-table-column prop="fieldCode" label="字段编码" width="150" />
          <el-table-column prop="fieldType" label="字段类型" width="150" />
          <el-table-column prop="createTime" label="创建时间" width="200" />
          <el-table-column label="操作" width="120" fixed="right">
            <template #default="scope">
              <el-button
                  type="danger"
                  size="small"
                  icon="el-icon-delete"
                  @click="deleteField(scope.row.id)"
                  class="btn-sm-delete"
              >
                删除
              </el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </el-card>

    <!-- 数据管理区域 -->
    <el-card title="📊 数据管理" class="card-wrapper">
      <el-form :model="dataForm" inline class="data-form" v-if="fieldList.length > 0">
        <el-form-item
            v-for="field in fieldList"
            :key="field.id"
            :label="field.fieldName"
            label-width="80px"
            class="form-item"
        >
          <template v-if="field.fieldType === 'DATE' || field.fieldType === 'DATETIME' || field.fieldType === 'TIMESTAMP'">
            <el-date-picker
                v-model="dataForm[field.fieldCode]"
                :type="field.fieldType === 'DATE' ? 'date' : 'datetime'"
                placeholder="选择日期/时间"
                style="width: 200px"
                clearable
                class="form-input"
            />
          </template>
          <template v-else-if="field.fieldType === 'INT' || field.fieldType === 'BIGINT'">
            <el-input
                v-model="dataForm[field.fieldCode]"
                type="number"
                :placeholder="`请输入${field.fieldName}`"
                style="width: 200px"
                clearable
                class="form-input"
            />
          </template>
          <template v-else-if="field.fieldType === 'DECIMAL'">
            <el-input
                v-model="dataForm[field.fieldCode]"
                type="number"
                step="0.01"
                :placeholder="`请输入${field.fieldName}`"
                style="width: 200px"
                clearable
                class="form-input"
            />
          </template>
          <template v-else-if="field.fieldType === 'BOOLEAN'">
            <el-select
                v-model="dataForm[field.fieldCode]"
                placeholder="选择布尔值"
                style="width: 200px"
                clearable
                class="form-select"
            >
              <el-option label="是" value="true" />
              <el-option label="否" value="false" />
            </el-select>
          </template>
          <template v-else>
            <el-input
                v-model="dataForm[field.fieldCode]"
                :placeholder="`请输入${field.fieldName}`"
                prefix-icon="el-icon-edit"
                style="width: 200px"
                clearable
                class="form-input"
            />
          </template>
        </el-form-item>
        <el-form-item>
          <el-button
              type="primary"
              @click="addData"
              icon="el-icon-circle-plus"
              class="btn-add"
              :disabled="editId !== null"
          >
            新增数据
          </el-button>
          <el-button
              type="warning"
              @click="updateData"
              icon="el-icon-edit"
              class="btn-update"
              style="margin-left: 10px"
              :disabled="editId === null"
          >
            保存修改
          </el-button>
          <el-button
              type="success"
              @click="getDataList"
              icon="el-icon-refresh"
              class="btn-refresh-data"
              style="margin-left: 10px"
          >
            刷新数据
          </el-button>
          <el-button
              type="info"
              @click="resetForm"
              icon="el-icon-refresh-left"
              class="btn-reset"
              style="margin-left: 10px"
              :disabled="editId === null"
          >
            取消编辑
          </el-button>
        </el-form-item>
      </el-form>
      <el-empty v-else description="请先加载字段配置" class="empty-container"></el-empty>

      <div class="list-container" v-if="fieldList.length > 0">
        <div class="list-header">
          <h4 class="list-title">数据列表</h4>
          <el-button
              type="primary"
              icon="el-icon-download"
              @click="exportToExcel"
              class="btn-export"
          >
            导出Excel
          </el-button>
        </div>
        <el-table
            :data="dataList"
            border
            stripe
            style="width: 100%"
            v-loading="dataLoading"
            class="data-table"
        >
          <el-table-column
              v-for="field in fieldList"
              :key="field.id"
              :prop="field.fieldCode"
              :label="field.fieldName"
              min-width="120"
          />
          <el-table-column label="操作" width="180" fixed="right">
            <template #default="scope">
              <el-button
                  type="text"
                  icon="el-icon-edit"
                  @click="editData(scope.row)"
                  class="btn-edit"
              >
                编辑
              </el-button>
              <el-button
                  type="text"
                  icon="el-icon-delete"
                  @click="deleteData(scope.row.id)"
                  class="btn-delete"
              >
                删除
              </el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </el-card>
  </div>
</template>

<script setup>
import { ref, reactive } from 'vue'
import { getCurrentInstance } from 'vue'
import { ElMessage, ElMessageBox } from 'element-plus'
// 导入Excel导出相关库
import * as XLSX from 'xlsx'
import { saveAs } from 'file-saver'

// 获取全局axios
const { proxy } = getCurrentInstance()

// 核心变量
const tableCode = ref('user_data') // 默认填充user_data，减少输入
const fieldList = ref([])
const dataList = ref([])
const fieldLoading = ref(false)
const dataLoading = ref(false)
const editId = ref(null) // 当前编辑的数据ID

// 字段表单数据
const fieldForm = reactive({
  name: '',
  code: '',
  type: 'VARCHAR' // 默认值：字符串
})

// 数据表单数据（动态绑定字段值）
const dataForm = reactive({})

// 1. 添加字段
const addField = async () => {
  if (!tableCode.value.trim()) {
    ElMessage.warning('请输入业务表标识')
    return
  }
  if (!fieldForm.name || !fieldForm.code) {
    ElMessage.warning('请输入字段名称和编码')
    return
  }
  fieldLoading.value = true
  try {
    const res = await proxy.$axios.post('/field/add', {
      tableCode: tableCode.value.trim(),
      fieldName: fieldForm.name,
      fieldCode: fieldForm.code,
      fieldType: fieldForm.type,
      isRequired: '1',
      sortNum: fieldList.value.length + 1
    })
    if (res.data.success) {
      ElMessage.success('字段添加成功 ✨')
      getFieldList() // 刷新字段
      // 清空输入
      fieldForm.name = ''
      fieldForm.code = ''
      fieldForm.type = 'VARCHAR' // 重置为默认类型
    } else {
      ElMessage.error(`添加失败：${res.data.msg}`)
    }
  } catch (err) {
    ElMessage.error(`添加失败：${err.response?.data?.msg || err.message}`)
  } finally {
    fieldLoading.value = false
  }
}

// 2. 加载字段列表
const getFieldList = async () => {
  const currentTableCode = tableCode.value.trim()
  if (!currentTableCode) {
    ElMessage.warning('请输入业务表标识')
    return
  }
  fieldLoading.value = true
  try {
    const res = await proxy.$axios.get('/field/list', {
      params: { tableCode: currentTableCode }
    })
    if (res.data.success) {
      fieldList.value = res.data.data || []
      ElMessage.success(`字段加载成功 📝（共${fieldList.value.length}个字段）`)
      // 加载字段后自动加载数据
      getDataList()
    } else {
      ElMessage.error(`加载失败：${res.data.msg}`)
    }
  } catch (err) {
    ElMessage.error(`加载失败：${err.response?.data?.msg || err.message}`)
  } finally {
    fieldLoading.value = false
  }
}

// 3. 删除字段
const deleteField = async (fieldId) => {
  if (!fieldId) return
  try {
    // 二次确认
    await ElMessageBox.confirm(
        '确定删除该字段？删除后关联的数据也会被清除！',
        '删除字段确认',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
    )
    const res = await proxy.$axios.delete(`/field/delete/${fieldId}`)
    if (res.data.success) {
      ElMessage.success('字段删除成功 🗑️')
      getFieldList() // 刷新字段列表
    } else {
      ElMessage.error(`删除失败：${res.data.msg}`)
    }
  } catch (err) {
    if (err !== 'cancel') {
      ElMessage.error(`删除失败：${err.response?.data?.msg || err.message}`)
    } else {
      ElMessage.info('已取消删除')
    }
  }
}

// 4. 新增数据
const addData = async () => {
  const currentTableCode = tableCode.value.trim()
  if (!currentTableCode) {
    ElMessage.warning('请输入业务表标识')
    return
  }
  if (fieldList.value.length === 0) {
    ElMessage.warning('请先加载字段配置')
    return
  }
  // 校验必填字段
  let isEmpty = false
  fieldList.value.forEach(field => {
    if (field.isRequired === 1 && !dataForm[field.fieldCode]) {
      ElMessage.warning(`${field.fieldName}为必填项`)
      isEmpty = true
    }
  })
  if (isEmpty) return

  dataLoading.value = true
  try {
    const res = await proxy.$axios.post('/data/add', {
      tableCode: currentTableCode,
      fieldValues: { ...dataForm }
    })
    if (res.data.success) {
      ElMessage.success('数据添加成功 🎉')
      getDataList() // 刷新数据
      // 清空输入
      resetForm()
    } else {
      ElMessage.error(`新增失败：${res.data.msg}`)
    }
  } catch (err) {
    ElMessage.error(`新增失败：${err.response?.data?.msg || err.message}`)
  } finally {
    dataLoading.value = false
  }
}

// 5. 加载数据列表
const getDataList = async () => {
  const currentTableCode = tableCode.value.trim()
  if (!currentTableCode) {
    ElMessage.warning('请输入业务表标识')
    return
  }
  dataLoading.value = true
  try {
    const res = await proxy.$axios.get('/data/list', {
      params: { tableCode: currentTableCode }
    })
    if (res.data.success) {
      dataList.value = res.data.data || []
      ElMessage.success(`数据加载成功 📊（共${dataList.value.length}条数据）`)
    } else {
      ElMessage.error(`加载失败：${res.data.msg}`)
    }
  } catch (err) {
    ElMessage.error(`加载失败：${err.response?.data?.msg || err.message}`)
  } finally {
    dataLoading.value = false
  }
}

// 6. 删除数据
const deleteData = async (id) => {
  if (!id) return
  try {
    await ElMessageBox.confirm(
        '确定删除该条数据？删除后不可恢复！',
        '删除确认',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
    )
    const res = await proxy.$axios.delete(`/data/delete/${id}`)
    if (res.data.success) {
      ElMessage.success('数据删除成功 🗑️')
      getDataList() // 刷新数据
    } else {
      ElMessage.error(`删除失败：${res.data.msg}`)
    }
  } catch (err) {
    if (err !== 'cancel') {
      ElMessage.error(`删除失败：${err.response?.data?.msg || err.message}`)
    } else {
      ElMessage.info('已取消删除')
    }
  }
}

// 7. 编辑数据（回显到表单）
const editData = (row) => {
  editId.value = row.id
  // 回显字段值
  fieldList.value.forEach(field => {
    dataForm[field.fieldCode] = row[field.fieldCode] || ''
  })
  ElMessage.info('请在上方表单修改数据，完成后点击「保存修改」')
}

// 8. 更新数据
const updateData = async () => {
  if (!editId.value) {
    ElMessage.warning('请先选择要编辑的数据')
    return
  }
  const currentTableCode = tableCode.value.trim()
  if (!currentTableCode) {
    ElMessage.warning('请输入业务表标识')
    return
  }
  // 校验必填字段
  let isEmpty = false
  fieldList.value.forEach(field => {
    if (field.isRequired === 1 && !dataForm[field.fieldCode]) {
      ElMessage.warning(`${field.fieldName}为必填项`)
      isEmpty = true
    }
  })
  if (isEmpty) return

  dataLoading.value = true
  try {
    const res = await proxy.$axios.put(`/data/update/${editId.value}`, {
      tableCode: currentTableCode,
      fieldValues: { ...dataForm }
    })
    if (res.data.success) {
      ElMessage.success('数据修改成功 ✏️')
      getDataList() // 刷新数据
      resetForm()
    } else {
      ElMessage.error(`修改失败：${res.data.msg}`)
    }
  } catch (err) {
    ElMessage.error(`修改失败：${err.response?.data?.msg || err.message}`)
  } finally {
    dataLoading.value = false
  }
}

// 9. 重置表单
const resetForm = () => {
  editId.value = null
  fieldList.value.forEach(field => {
    delete dataForm[field.fieldCode]
  })
}

// 10. 导出数据到Excel
const exportToExcel = () => {
  if (dataList.value.length === 0) {
    ElMessage.warning('暂无数据可导出')
    return
  }

  // 构建导出的表头和数据映射（字段编码 -> 字段名称）
  const headerMap = {}
  fieldList.value.forEach(field => {
    headerMap[field.fieldCode] = field.fieldName
  })

  // 转换数据格式，将字段编码替换为字段名称
  const exportData = dataList.value.map(row => {
    const newRow = {}
    Object.keys(row).forEach(key => {
      if (headerMap[key]) {
        newRow[headerMap[key]] = row[key]
      }
    })
    return newRow
  })

  // 创建工作簿和工作表
  const worksheet = XLSX.utils.json_to_sheet(exportData)
  const workbook = XLSX.utils.book_new()
  XLSX.utils.book_append_sheet(workbook, worksheet, '数据列表')

  // 生成Excel文件并下载
  const excelBuffer = XLSX.write(workbook, { bookType: 'xlsx', type: 'array' })
  const blob = new Blob([excelBuffer], { type: 'application/octet-stream' })
  // 生成带时间戳的文件名
  const fileName = `${tableCode.value}_数据_${new Date().toLocaleString().replace(/[/: ]/g, '-')}.xlsx`
  saveAs(blob, fileName)

  ElMessage.success('数据导出成功 🎉')
}

// 页面加载后自动加载字段
getFieldList()
</script>

<style scoped>
/* 全局容器 - 现代简约底色 */
.app-container {
  width: 95%;
  margin: 0 auto;
  padding: 24px 0;
  background-color: #f9fafb;
  min-height: 100vh;
  font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, sans-serif;
}

/* 头部容器 */
.header-container {
  margin-bottom: 24px;
}
.page-header {
  --el-page-header-text-color: #1f2937;
  --el-page-header-font-size: 22px;
  --el-page-header-content-font-weight: 600;
}

/* 卡片通用样式 - 轻量阴影+圆角 */
.card-wrapper {
  margin-bottom: 24px;
  border-radius: 12px;
  box-shadow: 0 1px 3px 0 rgba(0, 0, 0, 0.1), 0 1px 2px 0 rgba(0, 0, 0, 0.06);
  border: none;
  background-color: #ffffff;
  overflow: hidden;
}

/* 业务表标识卡片 */
.table-code-card {
  padding: 20px 24px;
}
.table-code-input {
  width: 320px;
  margin-right: 16px;
  border-radius: 8px;
  border: 1px solid #e5e7eb;
  transition: all 0.2s ease;
}
.table-code-input:focus {
  border-color: #3b82f6;
  box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.1);
}
.btn-refresh {
  height: 40px;
  border-radius: 8px;
  font-weight: 500;
  background-color: #3b82f6;
  border-color: #3b82f6;
}
.btn-refresh:hover {
  background-color: #2563eb;
  border-color: #2563eb;
}

/* 表单通用样式 */
.form-item {
  margin-bottom: 16px;
}
.form-item .el-form-item__label {
  color: #374151;
  font-weight: 500;
  font-size: 14px;
}
.form-input, .form-select {
  border-radius: 8px;
  border: 1px solid #e5e7eb;
  transition: all 0.2s ease;
}
.form-input:focus, .form-select:focus {
  border-color: #3b82f6;
  box-shadow: 0 0 0 2px rgba(59, 130, 246, 0.1);
}

/* 字段表单 */
.field-form {
  padding: 8px 0 24px 0;
  border-bottom: 1px solid #f3f4f6;
  margin-bottom: 24px;
}

/* 列表容器 */
.list-container {
  margin-top: 20px;
  padding: 0 4px 20px 4px;
}
.list-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 16px;
}
.list-title {
  font-size: 16px;
  color: #1f2937;
  margin: 0 0 16px 0;
  font-weight: 600;
  padding-bottom: 8px;
  border-bottom: 1px solid #f3f4f6;
}

/* 表格样式 */
.data-table {
  --el-table-header-text-color: #1f2937;
  --el-table-row-hover-bg-color: #f9fafb;
  --el-table-header-text-color: #374151;
  border-radius: 8px;
  overflow: hidden;
}
.data-table .el-table__header {
  background-color: #f9fafb;
}
.data-table .el-table__cell {
  padding: 12px 0;
}

/* 按钮样式 - 现代扁平化 */
.btn-add {
  background-color: #10b981;
  border-color: #10b981;
  border-radius: 8px;
  font-weight: 500;
  transition: all 0.2s ease;
}
.btn-add:hover {
  background-color: #059669;
  border-color: #059669;
}

.btn-update {
  background-color: #f59e0b;
  border-color: #f59e0b;
  border-radius: 8px;
  font-weight: 500;
  transition: all 0.2s ease;
}
.btn-update:hover {
  background-color: #d97706;
  border-color: #d97706;
}

.btn-refresh-data {
  background-color: #6366f1;
  border-color: #6366f1;
  border-radius: 8px;
  font-weight: 500;
  transition: all 0.2s ease;
}
.btn-refresh-data:hover {
  background-color: #4f46e5;
  border-color: #4f46e5;
}

.btn-reset {
  background-color: #6b7280;
  border-color: #6b7280;
  border-radius: 8px;
  font-weight: 500;
  transition: all 0.2s ease;
}
.btn-reset:hover {
  background-color: #4b5563;
  border-color: #4b5563;
}

.btn-export {
  background-color: #3b82f6;
  border-color: #3b82f6;
  border-radius: 8px;
  font-weight: 500;
}
.btn-export:hover {
  background-color: #2563eb;
  border-color: #2563eb;
}

.btn-sm-delete {
  border-radius: 6px;
  font-size: 12px;
}

.btn-edit {
  color: #f59e0b;
  font-weight: 500;
  transition: all 0.2s ease;
}
.btn-edit:hover {
  background-color: #fffbeb;
  color: #d97706;
  border-radius: 6px;
}

.btn-delete {
  color: #ef4444;
  font-weight: 500;
  transition: all 0.2s ease;
}
.btn-delete:hover {
  background-color: #fef2f2;
  color: #dc2626;
  border-radius: 6px;
}

/* 数据表单 */
.data-form {
  padding: 8px 0 24px 0;
  border-bottom: 1px solid #f3f4f6;
  margin-bottom: 24px;
}

/* 空状态容器 */
.empty-container {
  padding: 40px 0;
}

/* 适配小屏幕 */
@media (max-width: 1200px) {
  .table-code-input {
    width: 220px;
  }
  .form-item {
    margin-bottom: 12px;
  }
}
</style>