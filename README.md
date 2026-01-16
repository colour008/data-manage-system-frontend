# 动态数据管理系统 - 前端

基于 Vue 3 + Element Plus 开发的动态数据管理页面，支持字段动态配置、数据CRUD及Excel导出功能。

## 核心功能（对应代码实现）
1. **业务表标识管理**：输入/切换业务表标识，加载对应字段配置
2. **字段配置**：
   - 动态添加不同类型字段（字符串/整数/日期/布尔值等）
   - 查看已配置字段列表，支持删除操作
3. **数据管理**：
   - 根据配置的字段动态渲染数据录入表单
   - 数据新增、编辑、删除、刷新
   - 数据列表可视化展示
4. **Excel导出**：将当前业务表的数据列表导出为Excel文件（表头为字段名称，自动生成带时间戳的文件名）

## 技术栈
- 框架：Vue 3（Setup 语法糖）
- UI组件：Element Plus
- 工具库：XLSX + file-saver（Excel导出）
- 网络请求：Axios（全局挂载，通过 `proxy.$axios` 调用）

## 快速使用
### 1. 安装依赖
```bash
npm install
# 安装Excel导出依赖
npm install xlsx file-saver
```

### 2. 启动开发环境

```bash
npm run dev
```

### 3. 核心配置说明

- Axios 已全局挂载，接口请求前缀需与后端保持一致（当前代码中接口为 `/field/*`、`/data/*`）
- 默认业务表标识：`user_data`（可自行修改或输入）
- 字段类型支持：VARCHAR/INT/BIGINT/DECIMAL/DATE/DATETIME/TIMESTAMP/BOOLEAN/TEXT

## 代码核心逻辑

### 关键数据变量

- `tableCode`：当前选中的业务表标识
- `fieldList`：当前业务表的字段配置列表
- `dataList`：当前业务表的数据列表
- `editId`：当前编辑数据的 ID（区分新增 / 编辑状态）

### 核心方法

|     方法名      |           功能说明           |
| :-------------: | :--------------------------: |
| `getFieldList`  |  根据业务表标识加载字段配置  |
|   `addField`    |    添加新字段到当前业务表    |
|  `deleteField`  |  删除指定字段（含二次确认）  |
|    `addData`    |   新增数据（校验必填字段）   |
|   `editData`    | 回显数据到表单，进入编辑状态 |
|  `updateData`   |       保存编辑后的数据       |
|  `deleteData`   |  删除指定数据（含二次确认）  |
| `exportToExcel` |  导出数据列表为 Excel 文件   |

## 接口交互说明

- 所有接口请求均包含加载状态（v-loading）
- 操作结果有统一的消息提示（ElMessage）
- 删除操作需二次确认（ElMessageBox）
- 统一处理接口请求异常，反馈错误信息
