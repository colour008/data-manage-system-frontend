<template>
	<div class="app-container">
		<!-- 页面头部 -->
		<div class="header-container">
			<div class="header-content">
				<h1 class="page-title">
					<i class="icon-title el-icon-s-data"></i>
					动态数据管理系统
				</h1>
			</div>
		</div>

		<!-- 业务表标识输入区 -->
		<el-card class="card-wrapper table-code-card" shadow="hover">
			<div class="card-header">
				<h3 class="card-title">
					<i class="icon-card el-icon-s-platform"></i>
					业务表管理
				</h3>
			</div>
			<el-form-item
				label="业务表标识"
				label-width="110px"
				class="form-item">
				<div class="table-code-wrapper">
					<el-input
						v-model="tableCode"
						placeholder="请输入业务表标识（如：user_data）"
						class="table-code-input"
						size="large">
						<template #prefix>
							<i class="el-icon-folder-opened input-icon"></i>
						</template>
					</el-input>
					<el-button
						type="primary"
						@click="getFieldList"
						class="btn-refresh"
						size="large">
						<i class="el-icon-refresh"></i>
						加载字段
					</el-button>
				</div>
			</el-form-item>
		</el-card>

		<!-- 字段配置区域 -->
		<el-card class="card-wrapper" shadow="hover">
			<div class="card-header">
				<h3 class="card-title">
					<i class="icon-card el-icon-setting"></i>
					字段配置
					<span class="field-count" v-if="fieldList.length > 0">
						【已配置{{ fieldList.length }}个字段】
					</span>
				</h3>
			</div>

			<div class="field-add-section">
				<h4 class="section-title">
					<i class="el-icon-circle-plus"></i>
					添加新字段
				</h4>
				<el-form :model="fieldForm" inline class="field-form">
					<div class="form-row">
						<el-form-item
							label="字段名称"
							class="form-item enhanced">
							<el-input
								v-model="fieldForm.name"
								placeholder="如：用户名、年龄"
								class="form-input-enhanced"
								size="large">
								<template #prefix>
									<i class="el-icon-menu input-icon"></i>
								</template>
							</el-input>
						</el-form-item>

						<el-form-item
							label="字段编码"
							class="form-item enhanced">
							<el-input
								v-model="fieldForm.code"
								placeholder="如：user_name、age"
								class="form-input-enhanced"
								size="large">
								<template #prefix>
									<i class="el-icon-key input-icon"></i>
								</template>
							</el-input>
						</el-form-item>

						<el-form-item
							label="字段类型"
							class="form-item enhanced">
							<el-select
								v-model="fieldForm.type"
								placeholder="选择字段类型"
								class="form-select-enhanced"
								size="large">
								<template #prefix>
									<i class="el-icon-s-tools input-icon"></i>
								</template>
								<el-option
									label="字符串(VARCHAR)"
									value="VARCHAR" />
								<el-option label="整数(INT)" value="INT" />
								<el-option
									label="长整型(BIGINT)"
									value="BIGINT" />
								<el-option
									label="小数(DECIMAL)"
									value="DECIMAL" />
								<el-option label="日期(DATE)" value="DATE" />
								<el-option
									label="日期时间(DATETIME)"
									value="DATETIME" />
								<el-option
									label="时间戳(TIMESTAMP)"
									value="TIMESTAMP" />
								<el-option
									label="布尔值(BOOLEAN)"
									value="BOOLEAN" />
								<el-option label="文本(TEXT)" value="TEXT" />
							</el-select>
						</el-form-item>

						<el-form-item class="form-item enhanced">
							<el-button
								type="primary"
								@click="addField"
								class="btn-add-enhanced"
								size="large">
								<i class="el-icon-plus"></i>
								添加字段
							</el-button>
						</el-form-item>
					</div>
				</el-form>
			</div>

			<div class="list-container" v-if="fieldList.length > 0">
				<div class="list-header">
					<h4 class="list-title">
						<i class="el-icon-tickets"></i>
						已配置字段列表
					</h4>
				</div>
				<el-table
					:data="fieldList"
					style="width: 100%"
					v-loading="fieldLoading"
					class="data-table-enhanced"
					:header-cell-style="{
						background: '#667eea',
						color: 'white',
						fontWeight: '500',
						fontSize: '14px',
					}">
					<el-table-column
						prop="fieldName"
						label="字段名称"
						width="180">
						<template #header>
							<span class="column-header">
								<i class="el-icon-tickets"></i>
								字段名称
							</span>
						</template>
					</el-table-column>
					<el-table-column
						prop="fieldCode"
						label="字段编码"
						width="180">
						<template #header>
							<span class="column-header">
								<i class="el-icon-key"></i>
								字段编码
							</span>
						</template>
					</el-table-column>
					<el-table-column
						prop="fieldType"
						label="字段类型"
						width="150">
						<template #header>
							<span class="column-header">
								<i class="el-icon-s-tools"></i>
								字段类型
							</span>
						</template>
						<template #default="scope">
							<el-tag
								:type="getFieldTypeTag(scope.row.fieldType)">
								{{ scope.row.fieldType }}
							</el-tag>
						</template>
					</el-table-column>
					<el-table-column
						prop="createTime"
						label="创建时间"
						width="200">
						<template #header>
							<span class="column-header">
								<i class="el-icon-time"></i>
								创建时间
							</span>
						</template>
					</el-table-column>
					<el-table-column label="操作" width="120" fixed="right">
						<template #header>
							<span class="column-header">
								<i class="el-icon-setting"></i>
								操作
							</span>
						</template>
						<template #default="scope">
							<el-button
								type="danger"
								size="small"
								@click="deleteField(scope.row.id)"
								class="btn-sm-delete-enhanced">
								<i class="el-icon-delete"></i>
								删除
							</el-button>
						</template>
					</el-table-column>
				</el-table>
			</div>

			<div class="empty-state" v-else>
				<el-empty description="暂无字段配置" class="empty-container">
					<template #image>
						<i class="el-icon-document-copy empty-icon"></i>
					</template>
					<p class="empty-tip">请先添加字段或加载已有配置</p>
				</el-empty>
			</div>
		</el-card>

		<!-- 数据管理区域 -->
		<el-card class="card-wrapper" shadow="hover">
			<div class="card-header">
				<h3 class="card-title">
					<i class="icon-card el-icon-s-data"></i>
					数据管理
					<span class="field-count" v-if="dataList.length > 0">
						【已添加{{ dataList.length }}条记录】
					</span>
				</h3>
			</div>

			<div v-if="fieldList.length > 0">
				<div class="data-form-section">
					<h4 class="section-title">
						<i class="el-icon-edit"></i>
						{{ editId ? '编辑数据' : '添加新数据' }}
					</h4>
					<el-form
						:model="dataForm"
						inline
						class="data-form-enhanced">
						<div class="form-grid">
							<div
								class="form-grid-item"
								v-for="field in fieldList"
								:key="field.id">
								<div class="field-label">
									<span class="label-text">{{
										field.fieldName
									}}</span>
									<span
										v-if="field.isRequired === 1"
										class="required-mark"
										>*</span
									>
								</div>
								<div class="field-control">
									<template
										v-if="
											field.fieldType === 'DATE' ||
											field.fieldType === 'DATETIME' ||
											field.fieldType === 'TIMESTAMP'
										">
										<el-date-picker
											v-model="dataForm[field.fieldCode]"
											:type="
												field.fieldType === 'DATE'
													? 'date'
													: 'datetime'
											"
											placeholder="选择日期/时间"
											class="form-control-enhanced"
											:clearable="true" />
									</template>
									<template
										v-else-if="
											field.fieldType === 'INT' ||
											field.fieldType === 'BIGINT'
										">
										<el-input
											v-model="dataForm[field.fieldCode]"
											type="number"
											:placeholder="`请输入${field.fieldName}`"
											class="form-control-enhanced"
											:clearable="true" />
									</template>
									<template
										v-else-if="
											field.fieldType === 'DECIMAL'
										">
										<el-input
											v-model="dataForm[field.fieldCode]"
											type="number"
											step="0.01"
											:placeholder="`请输入${field.fieldName}`"
											class="form-control-enhanced"
											:clearable="true" />
									</template>
									<template
										v-else-if="
											field.fieldType === 'BOOLEAN'
										">
										<el-select
											v-model="dataForm[field.fieldCode]"
											placeholder="选择布尔值"
											class="form-control-enhanced"
											:clearable="true">
											<el-option
												label="是"
												value="true" />
											<el-option
												label="否"
												value="false" />
										</el-select>
									</template>
									<template v-else>
										<el-input
											v-model="dataForm[field.fieldCode]"
											:placeholder="`请输入${field.fieldName}`"
											class="form-control-enhanced"
											:clearable="true">
											<template #prefix>
												<i
													class="el-icon-edit input-icon"></i>
											</template>
										</el-input>
									</template>
								</div>
							</div>
						</div>

						<div class="form-actions">
							<el-button
								type="primary"
								@click="addData"
								class="btn-action-primary"
								:disabled="editId !== null">
								<i class="el-icon-circle-plus"></i>
								新增数据
							</el-button>
							<el-button
								type="warning"
								@click="updateData"
								class="btn-action-warning"
								:disabled="editId === null">
								<i class="el-icon-check"></i>
								保存修改
							</el-button>
							<el-button
								type="success"
								@click="getDataList"
								class="btn-action-success">
								<i class="el-icon-refresh"></i>
								刷新数据
							</el-button>
							<el-button
								type="info"
								@click="resetForm"
								class="btn-action-info"
								:disabled="editId === null">
								<i class="el-icon-refresh-left"></i>
								取消编辑
							</el-button>
						</div>
					</el-form>
				</div>

				<div class="list-container">
					<div class="list-header">
						<div class="list-header-left">
							<h4 class="list-title">
								<i class="el-icon-s-order"></i>
								数据列表
							</h4>
						</div>
						<div class="list-header-right">
							<el-button
								type="primary"
								@click="exportToExcel"
								class="btn-export-enhanced">
								<i class="el-icon-download"></i>
								导出Excel
							</el-button>
						</div>
					</div>
					<el-table
						:data="dataList"
						style="width: 100%"
						v-loading="dataLoading"
						class="data-table-enhanced"
						:row-class-name="tableRowClassName"
						:header-cell-style="{
							background: '#667eea',
							color: 'white',
							fontWeight: '500',
							fontSize: '14px',
						}">
						<el-table-column
							v-for="field in fieldList"
							:key="field.id"
							:prop="field.fieldCode"
							:label="field.fieldName"
							min-width="150">
							<template #default="scope">
								<span class="cell-content">
									{{
										formatCellValue(
											scope.row[field.fieldCode],
											field.fieldType,
										)
									}}
								</span>
							</template>
						</el-table-column>
						<el-table-column label="操作" width="180" fixed="right">
							<template #header>
								<span class="column-header">
									<i class="el-icon-setting"></i>
									操作
								</span>
							</template>
							<template #default="scope">
								<el-button
									type="warning"
									@click="editData(scope.row)"
									class="btn-action-edit"
									size="small">
									<i class="el-icon-edit"></i>
									编辑
								</el-button>
								<el-button
									type="danger"
									@click="deleteData(scope.row.id)"
									class="btn-action-delete"
									size="small">
									<i class="el-icon-delete"></i>
									删除
								</el-button>
							</template>
						</el-table-column>
					</el-table>

					<div class="table-footer" v-if="dataList.length > 0">
						<div class="summary-info">
							共
							<span class="highlight">{{ dataList.length }}</span>
							条记录
						</div>
					</div>
				</div>
			</div>

			<div class="empty-state" v-else>
				<el-empty description="请先配置字段" class="empty-container">
					<template #image>
						<i class="el-icon-s-data empty-icon"></i>
					</template>
					<p class="empty-tip">配置字段后即可管理数据</p>
				</el-empty>
			</div>
		</el-card>

		<div class="footer-tips">
			<i class="el-icon-info"></i>
			提示：所有操作都会实时同步到数据库，请谨慎操作
		</div>
	</div>
</template>

<script setup>
import { ref, reactive } from 'vue';
import { getCurrentInstance } from 'vue';
import { ElMessage, ElMessageBox } from 'element-plus';
// 导入Excel导出相关库
import * as XLSX from 'xlsx';
import { saveAs } from 'file-saver';

// 获取全局axios
const { proxy } = getCurrentInstance();

// 核心变量
const tableCode = ref('user_data'); // 默认填充user_data，减少输入
const fieldList = ref([]);
const dataList = ref([]);
const fieldLoading = ref(false);
const dataLoading = ref(false);
const editId = ref(null); // 当前编辑的数据ID

// 字段表单数据
const fieldForm = reactive({
	name: '',
	code: '',
	type: 'VARCHAR', // 默认值：字符串
});

// 数据表单数据（动态绑定字段值）
const dataForm = reactive({});

// 1. 添加字段
const addField = async () => {
	if (!tableCode.value.trim()) {
		ElMessage.warning('请输入业务表标识');
		return;
	}
	if (!fieldForm.name || !fieldForm.code) {
		ElMessage.warning('请输入字段名称和编码');
		return;
	}
	fieldLoading.value = true;
	try {
		const res = await proxy.$axios.post('/field/add', {
			tableCode: tableCode.value.trim(),
			fieldName: fieldForm.name,
			fieldCode: fieldForm.code,
			fieldType: fieldForm.type,
			isRequired: '1',
			sortNum: fieldList.value.length + 1,
		});
		if (res.data.success) {
			ElMessage.success('字段添加成功 ✨');
			getFieldList(); // 刷新字段
			// 清空输入
			fieldForm.name = '';
			fieldForm.code = '';
			fieldForm.type = 'VARCHAR'; // 重置为默认类型
		} else {
			ElMessage.error(`添加失败：${res.data.msg}`);
		}
	} catch (err) {
		ElMessage.error(`添加失败：${err.response?.data?.msg || err.message}`);
	} finally {
		fieldLoading.value = false;
	}
};

// 2. 加载字段列表
const getFieldList = async () => {
	const currentTableCode = tableCode.value.trim();
	if (!currentTableCode) {
		ElMessage.warning('请输入业务表标识');
		return;
	}
	fieldLoading.value = true;
	try {
		const res = await proxy.$axios.get('/field/list', {
			params: { tableCode: currentTableCode },
		});
		if (res.data.success) {
			fieldList.value = res.data.data || [];
			ElMessage.success(
				`字段加载成功 📝（共${fieldList.value.length}个字段）`,
			);
			// 加载字段后自动加载数据
			getDataList();
		} else {
			ElMessage.error(`加载失败：${res.data.msg}`);
		}
	} catch (err) {
		ElMessage.error(`加载失败：${err.response?.data?.msg || err.message}`);
	} finally {
		fieldLoading.value = false;
	}
};

// 3. 删除字段
const deleteField = async (fieldId) => {
	if (!fieldId) return;
	try {
		// 二次确认
		await ElMessageBox.confirm(
			'确定删除该字段？删除后关联的数据也会被清除！',
			'删除字段确认',
			{
				confirmButtonText: '确定',
				cancelButtonText: '取消',
				type: 'warning',
			},
		);
		const res = await proxy.$axios.delete(`/field/delete/${fieldId}`);
		if (res.data.success) {
			ElMessage.success('字段删除成功 🗑️');
			getFieldList(); // 刷新字段列表
		} else {
			ElMessage.error(`删除失败：${res.data.msg}`);
		}
	} catch (err) {
		if (err !== 'cancel') {
			ElMessage.error(
				`删除失败：${err.response?.data?.msg || err.message}`,
			);
		} else {
			ElMessage.info('已取消删除');
		}
	}
};

// 4. 新增数据
const addData = async () => {
	const currentTableCode = tableCode.value.trim();
	if (!currentTableCode) {
		ElMessage.warning('请输入业务表标识');
		return;
	}
	if (fieldList.value.length === 0) {
		ElMessage.warning('请先加载字段配置');
		return;
	}
	// 校验必填字段
	let isEmpty = false;
	fieldList.value.forEach((field) => {
		if (field.isRequired === 1 && !dataForm[field.fieldCode]) {
			ElMessage.warning(`${field.fieldName}为必填项`);
			isEmpty = true;
		}
	});
	if (isEmpty) return;

	dataLoading.value = true;
	try {
		const res = await proxy.$axios.post('/data/add', {
			tableCode: currentTableCode,
			fieldValues: { ...dataForm },
		});
		if (res.data.success) {
			ElMessage.success('数据添加成功 🎉');
			getDataList(); // 刷新数据
			// 清空输入
			resetForm();
		} else {
			ElMessage.error(`新增失败：${res.data.msg}`);
		}
	} catch (err) {
		ElMessage.error(`新增失败：${err.response?.data?.msg || err.message}`);
	} finally {
		dataLoading.value = false;
	}
};

// 5. 加载数据列表
const getDataList = async () => {
	const currentTableCode = tableCode.value.trim();
	if (!currentTableCode) {
		ElMessage.warning('请输入业务表标识');
		return;
	}
	dataLoading.value = true;
	try {
		const res = await proxy.$axios.get('/data/list', {
			params: { tableCode: currentTableCode },
		});
		if (res.data.success) {
			dataList.value = res.data.data || [];
			ElMessage.success(
				`数据加载成功 📊（共${dataList.value.length}条数据）`,
			);
		} else {
			ElMessage.error(`加载失败：${res.data.msg}`);
		}
	} catch (err) {
		ElMessage.error(`加载失败：${err.response?.data?.msg || err.message}`);
	} finally {
		dataLoading.value = false;
	}
};

// 6. 删除数据
const deleteData = async (id) => {
	if (!id) return;
	try {
		await ElMessageBox.confirm(
			'确定删除该条数据？删除后不可恢复！',
			'删除确认',
			{
				confirmButtonText: '确定',
				cancelButtonText: '取消',
				type: 'warning',
			},
		);
		const res = await proxy.$axios.delete(`/data/delete/${id}`);
		if (res.data.success) {
			ElMessage.success('数据删除成功 🗑️');
			getDataList(); // 刷新数据
		} else {
			ElMessage.error(`删除失败：${res.data.msg}`);
		}
	} catch (err) {
		if (err !== 'cancel') {
			ElMessage.error(
				`删除失败：${err.response?.data?.msg || err.message}`,
			);
		} else {
			ElMessage.info('已取消删除');
		}
	}
};

// 7. 编辑数据（回显到表单）
const editData = (row) => {
	editId.value = row.id;
	// 回显字段值
	fieldList.value.forEach((field) => {
		dataForm[field.fieldCode] = row[field.fieldCode] || '';
	});
	ElMessage.info('请在上方表单修改数据，完成后点击「保存修改」');
};

// 8. 更新数据
const updateData = async () => {
	if (!editId.value) {
		ElMessage.warning('请先选择要编辑的数据');
		return;
	}
	const currentTableCode = tableCode.value.trim();
	if (!currentTableCode) {
		ElMessage.warning('请输入业务表标识');
		return;
	}
	// 校验必填字段
	let isEmpty = false;
	fieldList.value.forEach((field) => {
		if (field.isRequired === 1 && !dataForm[field.fieldCode]) {
			ElMessage.warning(`${field.fieldName}为必填项`);
			isEmpty = true;
		}
	});
	if (isEmpty) return;

	dataLoading.value = true;
	try {
		const res = await proxy.$axios.put(`/data/update/${editId.value}`, {
			tableCode: currentTableCode,
			fieldValues: { ...dataForm },
		});
		if (res.data.success) {
			ElMessage.success('数据修改成功 ✏️');
			getDataList(); // 刷新数据
			resetForm();
		} else {
			ElMessage.error(`修改失败：${res.data.msg}`);
		}
	} catch (err) {
		ElMessage.error(`修改失败：${err.response?.data?.msg || err.message}`);
	} finally {
		dataLoading.value = false;
	}
};

// 9. 重置表单
const resetForm = () => {
	editId.value = null;
	fieldList.value.forEach((field) => {
		delete dataForm[field.fieldCode];
	});
};

// 10. 导出数据到Excel
const exportToExcel = () => {
	if (dataList.value.length === 0) {
		ElMessage.warning('暂无数据可导出');
		return;
	}

	// 构建导出的表头和数据映射（字段编码 -> 字段名称）
	const headerMap = {};
	fieldList.value.forEach((field) => {
		headerMap[field.fieldCode] = field.fieldName;
	});

	// 转换数据格式，将字段编码替换为字段名称
	const exportData = dataList.value.map((row) => {
		const newRow = {};
		Object.keys(row).forEach((key) => {
			if (headerMap[key]) {
				newRow[headerMap[key]] = row[key];
			}
		});
		return newRow;
	});

	// 创建工作簿和工作表
	const worksheet = XLSX.utils.json_to_sheet(exportData);
	const workbook = XLSX.utils.book_new();
	XLSX.utils.book_append_sheet(workbook, worksheet, '数据列表');

	// 生成Excel文件并下载
	const excelBuffer = XLSX.write(workbook, {
		bookType: 'xlsx',
		type: 'array',
	});
	const blob = new Blob([excelBuffer], { type: 'application/octet-stream' });
	// 生成带时间戳的文件名
	const fileName = `${tableCode.value}_数据_${new Date()
		.toLocaleString()
		.replace(/[/: ]/g, '-')}.xlsx`;
	saveAs(blob, fileName);

	ElMessage.success('数据导出成功 🎉');
};

// 页面加载后自动加载字段
getFieldList();

// 在原有的JavaScript代码后添加以下辅助方法
const getFieldTypeTag = (type) => {
	const tagMap = {
		VARCHAR: 'primary',
		INT: 'success',
		BIGINT: 'warning',
		DECIMAL: 'info',
		DATE: 'success',
		DATETIME: 'warning',
		TIMESTAMP: 'danger',
		BOOLEAN: 'primary',
		TEXT: 'info',
	};
	return tagMap[type] || 'info';
};

const formatCellValue = (value, type) => {
	if (value === null || value === undefined) return '-';

	switch (type) {
		case 'BOOLEAN':
			return value === 'true' ? '是' : '否';
		case 'DATE':
		case 'DATETIME':
		case 'TIMESTAMP':
			return new Date(value).toLocaleString();
		default:
			return value;
	}
};

const tableRowClassName = ({ rowIndex }) => {
	return rowIndex % 2 === 1 ? 'even-row' : '';
};
</script>

<style scoped>
/* 整体布局 */
.app-container {
	width: 98%;
	margin: 0 auto;
	padding: 10px 0; /* 缩小整体内边距，更紧凑 */
	/* background: linear-gradient(135deg, #f5f7fa 0%, #e4e8f0 100%); */
	min-height: 100vh;
	font-family:
		'Inter',
		-apple-system,
		BlinkMacSystemFont,
		'Segoe UI',
		Roboto,
		'Helvetica Neue',
		Arial,
		sans-serif;
}

/* 头部样式 */
.header-container {
	margin-bottom: 10px; /* 缩小底部间距 */
	text-align: center;
}

.header-content {
	background: #667eea;
	padding: 3px; /* 缩小内边距 */
	color: white;
	box-shadow: 0 10px 30px rgba(102, 126, 234, 0.2);
}

.page-title {
	font-size: 24px;
	font-weight: 500;
	display: flex;
	align-items: center;
	justify-content: center;
}

.icon-title {
	font-size: 28px;
}

.page-subtitle {
	font-size: 16px;
	opacity: 0.9;
	font-weight: 400;
}

/* 卡片通用样式 */
.card-wrapper {
	margin-bottom: 10px; /* 缩小卡片间距 */
	border: none;
	background: white;
	overflow: hidden;
	transition:
		transform 0.3s ease,
		box-shadow 0.3s ease;
	box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1);
}

.card-wrapper:hover {
	transform: translateY(-2px);
	box-shadow: 0 15px 35px rgba(0, 0, 0, 0.1);
}

.card-header {
	padding: 0px 10px; /* 缩小内边距 */
	margin-bottom: 5px; /* 缩小底部间距 */
	border: 1px solid #e2e8f0;
}

.card-title {
	font-size: 18px;
	font-weight: 500;
	color: #2d3748;
	display: flex;
}

.icon-card {
	color: #667eea;
	font-size: 22px;
}

.field-count {
	color: rgb(202, 52, 52);
	font-size: 17px;
	margin-left: 6px;
}

/* 业务表标识卡片 */

.table-code-wrapper {
	display: flex;
	gap: 10px;
	align-items: center;
}

.table-code-input {
	flex: 1;
	transition: all 0.3s ease;
}

.table-code-input:hover,
.table-code-input:focus-within {
	border-color: #667eea;
	box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.btn-refresh {
	background: #667eea;
	border: none;
	padding: 0 24px;
	font-weight: 300;
	transition: all 0.3s ease;
}

.btn-refresh:hover {
	transform: translateY(-2px);
	box-shadow: 0 5px 15px rgba(102, 126, 234, 0.3);
}

/* 字段添加区域 */
.field-add-section {
	background: #f8fafc;
	padding: 5px 15px; /* 缩小内边距 */
	margin-bottom: 16px; /* 缩小底部间距 */
	border: 1px solid #e2e8f0;
}

.section-title {
	font-size: 16px;
	font-weight: 500;
	color: #2d3748;
	margin-bottom: 15px; /* 缩小底部间距 */
	display: flex;
	align-items: center;
}

.form-row {
	display: grid;
	grid-template-columns: repeat(auto-fit, minmax(250px, 1fr));
	gap: 15px; /* 缩小间距 */
	align-items: end;
}

.form-item.enhanced {
	margin-bottom: 0;
}

.form-item.enhanced :deep(.el-form-item__label) {
	font-weight: 500;
	color: #4a5568;
	font-size: 14px;
}

.form-input-enhanced,
.form-select-enhanced {
	width: 100%;
	transition: all 0.3s ease;
}

.form-input-enhanced:hover,
.form-select-enhanced:hover,
.form-input-enhanced:focus-within,
.form-select-enhanced:focus-within {
	border-color: #667eea;
	box-shadow: 0 0 0 3px rgba(102, 126, 234, 0.1);
}

.input-icon {
	color: #a0aec0;
}

.btn-add-enhanced {
	background: linear-gradient(135deg, #10b981 0%, #059669 100%);
	border: none;
	padding: 0 28px;
	height: 40px;
	font-weight: 300;
	transition: all 0.3s ease;
}

.btn-add-enhanced:hover {
	transform: translateY(-2px);
	box-shadow: 0 5px 15px rgba(16, 185, 129, 0.3);
}

/* 列表区域 */
.list-container {
	margin-top: 15px; /* 缩小顶部间距 */
}

.list-header {
	display: flex;
	justify-content: space-between;
	align-items: center;
	margin-bottom: 15px; /* 缩小底部间距 */
	padding: 0 4px;
}

.list-header-left {
	display: flex;
	flex-direction: column;
	gap: 4px;
}

.list-title {
	font-size: 18px;
	font-weight: 500;
	color: #2d3748;
	margin: 0;
	display: flex;
}

/* 表格样式 */
.data-table-enhanced {
	overflow: hidden;
}

.data-table-enhanced :deep(.el-table__row) {
	transition: background-color 0.3s ease;
}

.data-table-enhanced :deep(.el-table__row:hover) {
	background-color: #f7fafc;
}

.data-table-enhanced :deep(.el-table__cell) {
	padding: 10px 0; /* 缩小单元格内边距 */
	border-color: #e2e8f0;
}

.column-header {
	display: flex;
	align-items: center;
}

.cell-content {
	color: #4a5568;
	font-size: 14px;
}

/* 操作按钮 */
.btn-sm-delete-enhanced {
	background: linear-gradient(135deg, #f56565 0%, #e53e3e 100%);
	border: none;
	color: white;
	transition: all 0.3s ease;
	padding: 0 10px; /* 调整内边距 */
}

.btn-sm-delete-enhanced:hover {
	transform: scale(1.05); /* 微调 hover 缩放效果 */
	box-shadow: 0 3px 10px rgba(245, 101, 101, 0.3);
}

/* 数据表单区域 */
.data-form-section {
	background: #f8fafc;
	padding: 10px; /* 缩小内边距 */
	margin-bottom: 20px; /* 缩小底部间距 */
	border: 1px solid #e2e8f0;
}

.form-grid {
	display: grid;
	grid-template-columns: repeat(auto-fill, minmax(300px, 1fr));
	gap: 15px; /* 缩小间距 */
	margin-bottom: 20px; /* 缩小底部间距 */
}

.form-grid-item {
	background: white;
	padding: 14px; /* 缩小内边距 */
	border: 1px solid #e2e8f0;
	transition: all 0.3s ease;
}

.form-grid-item:hover {
	border-color: #667eea;
	box-shadow: 0 5px 15px rgba(0, 0, 0, 0.05);
}

.field-label {
	display: flex;
	align-items: center;
	gap: 4px;
	margin-bottom: 6px; /* 缩小底部间距 */
}

.label-text {
	font-weight: 500;
	color: #4a5568;
	font-size: 14px;
}

.required-mark {
	color: #f56565;
}

.form-control-enhanced {
	width: 100%;
}

.form-actions {
	display: flex;
	gap: 10px; /* 缩小按钮间距 */
	flex-wrap: wrap;
	padding-top: 15px; /* 缩小顶部内边距 */
	border-top: 1px solid #e2e8f0;
}

.btn-action-primary,
.btn-action-warning,
.btn-action-success,
.btn-action-info {
	padding: 0 20px; /* 缩小按钮内边距 */
	font-weight: 300;
	transition: all 0.3s ease;
	border: none;
	color: #ffffff;
}

.btn-action-primary {
	background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
}

.btn-action-warning {
	background: linear-gradient(135deg, #f59e0b 0%, #d97706 100%);
}

.btn-action-success {
	background: linear-gradient(135deg, #10b981 0%, #059669 100%);
}

.btn-action-info {
	background: linear-gradient(135deg, #718096 0%, #404e67 100%);
}

.btn-action-primary:hover,
.btn-action-warning:hover,
.btn-action-success:hover,
.btn-action-info:hover {
	transform: translateY(-2px);
	opacity: 0.95;
}

/* 数据操作按钮 */
.btn-action-edit {
	background: linear-gradient(135deg, #f59e0b 0%, #d97706 100%);
	border: none;
	color: white;
	transition: all 0.3s ease;
	margin-right: 5px; /* 缩小按钮间距 */
}

.btn-action-delete {
	background: linear-gradient(135deg, #f56565 0%, #e53e3e 100%);
	border: none;
	color: white;
	transition: all 0.3s ease;
}

.btn-action-edit:hover,
.btn-action-delete:hover {
	transform: translateY(-2px);
	box-shadow: 0 3px 10px rgba(0, 0, 0, 0.1);
}

/* 导出按钮 */
.btn-export-enhanced {
	background: linear-gradient(135deg, #4299e1 0%, #3182ce 100%);
	border: none;
	color: white;
	font-weight: 500;
	transition: all 0.3s ease;
}

.btn-export-enhanced:hover {
	transform: translateY(-2px);
	box-shadow: 0 5px 15px rgba(66, 153, 225, 0.3);
}

/* 空状态 */
.empty-state {
	padding: 30px 0; /* 缩小内边距 */
}

.empty-icon {
	font-size: 80px;
	color: #a0aec0;
}

.empty-tip {
	color: #718096;
	font-size: 14px;
	margin-top: 10px;
}

/* 表格底部 */
.table-footer {
	margin-top: 15px; /* 缩小顶部间距 */
	padding: 14px; /* 缩小内边距 */
	background: #f8fafc;
	display: flex;
	justify-content: flex-end;
}

.summary-info {
	color: #718096;
	font-size: 14px;
}

.highlight {
	color: #667eea;
	font-weight: 500;
}

/* 页脚提示 */
.footer-tips {
	padding: 10px; /* 缩小内边距 */
	color: #cb2d2d;
	font-size: 14px;
}

/* 针对数据管理区的输入框 */
:deep(.form-control-enhanced .el-input__inner::placeholder) {
	color: #9b9b9b !important;
	font-size: 13px !important;
	opacity: 1 !important;
	font-weight: 300 !important;
	font-style: italic;
}

/* 针对字段配置区的输入框 */
:deep(.form-input-enhanced .el-input__inner::placeholder) {
	color: #9b9b9b !important;
	font-size: 13px !important;
	opacity: 1 !important;
	font-weight: 300 !important;
	font-style: italic;
}

:deep(.el-form-item__content .el-select--large .el-select__wrapper) {
	font-size: 12px !important;
}

/* 响应式设计 */
@media (max-width: 1200px) {
	.form-grid {
		grid-template-columns: repeat(auto-fill, minmax(280px, 1fr));
	}
}

@media (max-width: 992px) {
	.form-row {
		grid-template-columns: 1fr;
	}

	.form-grid {
		grid-template-columns: 1fr;
	}

	.list-header {
		flex-direction: column;
		gap: 10px; /* 缩小间距 */
		align-items: stretch;
	}

	.form-actions {
		justify-content: center;
	}
}

@media (max-width: 768px) {
	.app-container {
		width: 100%;
		padding: 8px; /* 缩小内边距 */
	}

	.header-content {
		padding: 20px 15px; /* 缩小内边距 */
	}

	.page-title {
		font-size: 24px;
	}

	.card-header {
		padding: 10px 15px 0; /* 缩小内边距 */
	}

	.table-code-wrapper {
		flex-direction: column;
	}

	.table-code-input {
		width: 100%;
	}

	.btn-refresh {
		width: 100%;
	}
}
</style>
