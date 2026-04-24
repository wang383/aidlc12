# 页面模板参考文档

> 本文档包含 4 种标准页面类型的完整 HTML 模板。Skill 在生成 DEMO 时按需加载对应模板。

## 目录
1. 列表页模板
2. 表单页模板
3. 详情页模板
4. 仪表盘模板

## 1. 列表页模板

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>用户管理 - 列表页</title>
  <link rel="stylesheet" href="https:https:https:https://unpkg.com/element-plus@2.9.1/dist/index.css" />
  <style>
    body {
      margin: 0;
      padding: 20px;
      background: #f5f7fa;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
    }
    .page-container {
      max-width: 1200px;
      margin: 0 auto;
      background: #fff;
      border-radius: 8px;
      padding: 24px;
      box-shadow: 0 2px 12px rgba(0, 0, 0, 0.06);
    }
    .page-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
    }
    .page-header h2 {
      margin: 0;
      font-size: 20px;
      color: #303133;
    }
    .search-area {
      margin-bottom: 20px;
      padding: 16px;
      background: #fafafa;
      border-radius: 6px;
    }
    .table-area {
      margin-bottom: 20px;
    }
    .pagination-area {
      display: flex;
      justify-content: flex-end;
      padding-top: 16px;
    }
  </style>
</head>
<body>
  <div id="app">
    <div class="page-container">
      <!-- 页面标题 -->
      <div class="page-header">
        <h2>用户管理</h2>
        <div>
          <el-button type="primary" @click="handleAdd">
            <el-icon><Plus></Plus></el-icon>
            新增
          </el-button>
          <el-button @click="handleExport">
            <el-icon><Download></Download></el-icon>
            导出
          </el-button>
        </div>
      </div>

      <!-- 搜索区域 -->
      <div class="search-area">
        <el-form :model="searchForm" :inline="true">
          <el-form-item label="姓名">
            <el-input v-model="searchForm.name" placeholder="请输入姓名" clearable></el-input>
          </el-form-item>
          <el-form-item label="角色">
            <el-select v-model="searchForm.role" placeholder="请选择角色" clearable>
              <el-option label="管理员" value="管理员"></el-option>
              <el-option label="普通用户" value="普通用户"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="状态">
            <el-select v-model="searchForm.status" placeholder="请选择状态" clearable>
              <el-option label="启用" value="启用"></el-option>
              <el-option label="禁用" value="禁用"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="handleSearch">
              <el-icon><Search></Search></el-icon>
              搜索
            </el-button>
            <el-button @click="handleReset">
              <el-icon><Refresh></Refresh></el-icon>
              重置
            </el-button>
          </el-form-item>
        </el-form>
      </div>

      <!-- 数据表格 -->
      <div class="table-area">
        <el-table :data="pagedData" border stripe style="width: 100%">
          <el-table-column prop="id" label="ID" width="60" align="center"></el-table-column>
          <el-table-column prop="name" label="姓名" width="100"></el-table-column>
          <el-table-column prop="email" label="邮箱" min-width="180"></el-table-column>
          <el-table-column prop="role" label="角色" width="100" align="center">
            <template #default="{ row }">
              <el-tag :type="row.role === '管理员' ? 'danger' : 'info'">{{ row.role }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="status" label="状态" width="80" align="center">
            <template #default="{ row }">
              <el-tag :type="row.status === '启用' ? 'success' : 'danger'">{{ row.status }}</el-tag>
            </template>
          </el-table-column>
          <el-table-column prop="createdAt" label="创建时间" width="170" align="center"></el-table-column>
          <el-table-column label="操作" width="160" align="center" fixed="right">
            <template #default="{ row }">
              <el-button type="primary" link size="small" @click="handleEdit(row)">
                <el-icon><Edit></Edit></el-icon>
                编辑
              </el-button>
              <el-button type="danger" link size="small" @click="handleDelete(row)">
                <el-icon><Delete></Delete></el-icon>
                删除
              </el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>

      <!-- 分页 -->
      <div class="pagination-area">
        <el-pagination
          v-model:current-page="currentPage"
          v-model:page-size="pageSize"
          :page-sizes="[5, 10, 20, 50]"
          :total="filteredData.length"
          layout="total, sizes, prev, pager, next, jumper"
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
        ></el-pagination>
      </div>

      <!-- 新增/编辑对话框 -->
      <el-dialog
        v-model="dialogVisible"
        :title="dialogTitle"
        width="500px"
        destroy-on-close
      >
        <el-form
          ref="dialogFormRef"
          :model="dialogForm"
          :rules="dialogRules"
          label-width="80px"
        >
          <el-form-item label="姓名" prop="name">
            <el-input v-model="dialogForm.name" placeholder="请输入姓名"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="dialogForm.email" placeholder="请输入邮箱"></el-input>
          </el-form-item>
          <el-form-item label="角色" prop="role">
            <el-select v-model="dialogForm.role" placeholder="请选择角色" style="width: 100%">
              <el-option label="管理员" value="管理员"></el-option>
              <el-option label="普通用户" value="普通用户"></el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="状态" prop="status">
            <el-select v-model="dialogForm.status" placeholder="请选择状态" style="width: 100%">
              <el-option label="启用" value="启用"></el-option>
              <el-option label="禁用" value="禁用"></el-option>
            </el-select>
          </el-form-item>
        </el-form>
        <template #footer>
          <el-button @click="dialogVisible = false">取消</el-button>
          <el-button type="primary" @click="handleSubmit">确定</el-button>
        </template>
      </el-dialog>
    </div>
  </div>

  <script src="https:https:https:https://unpkg.com/vue@3.4.38/dist/vue.global.prod.js"></script>
  <script src="https:https:https:https://unpkg.com/element-plus@2.9.1"></script>
  <script src="https:https:https:https://unpkg.com/@element-plus/icons-vue@2.3.1"></script>
  <script>
    const { createApp, ref, reactive, computed } = Vue

    const app = createApp({
      setup() {
        // 搜索表单
        const searchForm = reactive({
          name: '',
          role: '',
          status: ''
        })

        // 表格数据
        const tableData = ref([
          { id: 1, name: '张伟', email: 'zhangwei@example.com', role: '管理员', status: '启用', createdAt: '2024-01-05 09:30:00' },
          { id: 2, name: '李娜', email: 'lina@example.com', role: '普通用户', status: '启用', createdAt: '2024-01-08 14:20:00' },
          { id: 3, name: '王强', email: 'wangqiang@example.com', role: '普通用户', status: '禁用', createdAt: '2024-01-12 10:15:00' },
          { id: 4, name: '赵敏', email: 'zhaomin@example.com', role: '管理员', status: '启用', createdAt: '2024-02-01 08:45:00' },
          { id: 5, name: '刘洋', email: 'liuyang@example.com', role: '普通用户', status: '启用', createdAt: '2024-02-14 16:30:00' },
          { id: 6, name: '陈静', email: 'chenjing@example.com', role: '普通用户', status: '启用', createdAt: '2024-02-20 11:00:00' },
          { id: 7, name: '杨帆', email: 'yangfan@example.com', role: '管理员', status: '启用', createdAt: '2024-03-03 09:10:00' },
          { id: 8, name: '黄磊', email: 'huanglei@example.com', role: '普通用户', status: '禁用', createdAt: '2024-03-10 13:25:00' },
          { id: 9, name: '周婷', email: 'zhouting@example.com', role: '普通用户', status: '启用', createdAt: '2024-03-18 15:40:00' },
          { id: 10, name: '吴昊', email: 'wuhao@example.com', role: '普通用户', status: '启用', createdAt: '2024-04-02 10:50:00' },
          { id: 11, name: '郑雪', email: 'zhengxue@example.com', role: '管理员', status: '启用', createdAt: '2024-04-15 08:30:00' },
          { id: 12, name: '孙鹏', email: 'sunpeng@example.com', role: '普通用户', status: '禁用', createdAt: '2024-04-22 14:15:00' },
          { id: 13, name: '马丽', email: 'mali@example.com', role: '普通用户', status: '启用', createdAt: '2024-05-06 09:20:00' },
          { id: 14, name: '朱军', email: 'zhujun@example.com', role: '普通用户', status: '启用', createdAt: '2024-05-18 11:35:00' },
          { id: 15, name: '胡蝶', email: 'hudie@example.com', role: '管理员', status: '启用', createdAt: '2024-06-01 16:00:00' }
        ])

        // 分页
        const currentPage = ref(1)
        const pageSize = ref(10)

        // 过滤后的数据
        const filteredData = computed(() => {
          return tableData.value.filter(item => {
            const nameMatch = !searchForm.name || item.name.includes(searchForm.name)
            const roleMatch = !searchForm.role || item.role === searchForm.role
            const statusMatch = !searchForm.status || item.status === searchForm.status
            return nameMatch && roleMatch && statusMatch
          })
        })

        // 分页后的数据
        const pagedData = computed(() => {
          const start = (currentPage.value - 1) * pageSize.value
          const end = start + pageSize.value
          return filteredData.value.slice(start, end)
        })

        // 对话框
        const dialogVisible = ref(false)
        const dialogTitle = ref('新增用户')
        const dialogFormRef = ref(null)
        const editingId = ref(null)
        const dialogForm = reactive({
          name: '',
          email: '',
          role: '',
          status: '启用'
        })
        const dialogRules = {
          name: [{ required: true, message: '请输入姓名', trigger: 'blur' }],
          email: [
            { required: true, message: '请输入邮箱', trigger: 'blur' },
            { type: 'email', message: '请输入正确的邮箱格式', trigger: 'blur' }
          ],
          role: [{ required: true, message: '请选择角色', trigger: 'change' }],
          status: [{ required: true, message: '请选择状态', trigger: 'change' }]
        }

        // 方法
        const handleSearch = () => {
          currentPage.value = 1
        }

        const handleReset = () => {
          searchForm.name = ''
          searchForm.role = ''
          searchForm.status = ''
          currentPage.value = 1
        }

        const handleAdd = () => {
          dialogTitle.value = '新增用户'
          editingId.value = null
          dialogForm.name = ''
          dialogForm.email = ''
          dialogForm.role = ''
          dialogForm.status = '启用'
          dialogVisible.value = true
        }

        const handleEdit = (row) => {
          dialogTitle.value = '编辑用户'
          editingId.value = row.id
          dialogForm.name = row.name
          dialogForm.email = row.email
          dialogForm.role = row.role
          dialogForm.status = row.status
          dialogVisible.value = true
        }

        const handleDelete = (row) => {
          ElMessageBox.confirm(`确定要删除用户「${row.name}」吗？`, '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '取消',
            type: 'warning'
          }).then(() => {
            const index = tableData.value.findIndex(item => item.id === row.id)
            if (index > -1) {
              tableData.value.splice(index, 1)
              ElMessage.success('删除成功')
            }
          }).catch(() => {})
        }

        const handleSubmit = () => {
          if (!dialogFormRef.value) return
          dialogFormRef.value.validate((valid) => {
            if (valid) {
              if (editingId.value) {
                const index = tableData.value.findIndex(item => item.id === editingId.value)
                if (index > -1) {
                  tableData.value[index] = {
                    ...tableData.value[index],
                    name: dialogForm.name,
                    email: dialogForm.email,
                    role: dialogForm.role,
                    status: dialogForm.status
                  }
                  ElMessage.success('编辑成功')
                }
              } else {
                const newId = Math.max(...tableData.value.map(item => item.id)) + 1
                const now = new Date()
                const createdAt = `${now.getFullYear()}-${String(now.getMonth() + 1).padStart(2, '0')}-${String(now.getDate()).padStart(2, '0')} ${String(now.getHours()).padStart(2, '0')}:${String(now.getMinutes()).padStart(2, '0')}:${String(now.getSeconds()).padStart(2, '0')}`
                tableData.value.push({
                  id: newId,
                  name: dialogForm.name,
                  email: dialogForm.email,
                  role: dialogForm.role,
                  status: dialogForm.status,
                  createdAt
                })
                ElMessage.success('新增成功')
              }
              dialogVisible.value = false
            }
          })
        }

        const handleExport = () => {
          ElMessage.success('导出功能已触发')
        }

        const handleSizeChange = (val) => {
          pageSize.value = val
          currentPage.value = 1
        }

        const handleCurrentChange = (val) => {
          currentPage.value = val
        }

        return {
          searchForm,
          tableData,
          currentPage,
          pageSize,
          filteredData,
          pagedData,
          dialogVisible,
          dialogTitle,
          dialogFormRef,
          dialogForm,
          dialogRules,
          handleSearch,
          handleReset,
          handleAdd,
          handleEdit,
          handleDelete,
          handleSubmit,
          handleExport,
          handleSizeChange,
          handleCurrentChange
        }
      }
    })

    // 注册 Element Plus
    app.use(ElementPlus, { locale: ElementPlus.lang.zhCn })

    // 注册所有图标
    for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
      app.component(key, component)
    }

    app.mount('#app')
  </script>
</body>
</html>
```


## 2. 表单页模板

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>创建订单 - 表单页</title>
  <link rel="stylesheet" href="https:https:https:https://unpkg.com/element-plus@2.9.1/dist/index.css" />
  <style>
    body {
      margin: 0;
      padding: 20px;
      background: #f5f7fa;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
    }
    .page-container {
      max-width: 800px;
      margin: 0 auto;
      background: #fff;
      border-radius: 8px;
      padding: 24px;
      box-shadow: 0 2px 12px rgba(0, 0, 0, 0.06);
    }
    .page-header {
      margin-bottom: 24px;
    }
    .page-header h2 {
      margin: 0;
      font-size: 20px;
      color: #303133;
    }
    .form-actions {
      text-align: center;
      padding-top: 20px;
      border-top: 1px solid #ebeef5;
      margin-top: 20px;
    }
  </style>
</head>
<body>
  <div id="app">
    <div class="page-container">
      <!-- 页面标题 -->
      <div class="page-header">
        <h2>创建订单</h2>
      </div>

      <!-- 表单 -->
      <el-form
        ref="formRef"
        :model="form"
        :rules="rules"
        label-width="120px"
        status-icon
      >
        <el-form-item label="订单标题" prop="title">
          <el-input v-model="form.title" placeholder="请输入订单标题" maxlength="50" show-word-limit></el-input>
        </el-form-item>

        <el-form-item label="客户名称" prop="customerName">
          <el-input v-model="form.customerName" placeholder="请输入客户名称"></el-input>
        </el-form-item>

        <el-form-item label="联系邮箱" prop="email">
          <el-input v-model="form.email" placeholder="请输入联系邮箱"></el-input>
        </el-form-item>

        <el-form-item label="订单金额" prop="amount">
          <el-input-number
            v-model="form.amount"
            :min="0"
            :max="9999999"
            :precision="2"
            :step="100"
            controls-position="right"
            placeholder="请输入订单金额"
            style="width: 100%"
          ></el-input-number>
        </el-form-item>

        <el-form-item label="订单类型" prop="type">
          <el-radio-group v-model="form.type">
            <el-radio value="标准">标准</el-radio>
            <el-radio value="加急">加急</el-radio>
            <el-radio value="特殊">特殊</el-radio>
          </el-radio-group>
        </el-form-item>

        <el-form-item label="交付日期" prop="deliveryDate">
          <el-date-picker
            v-model="form.deliveryDate"
            type="date"
            placeholder="请选择交付日期"
            format="YYYY-MM-DD"
            value-format="YYYY-MM-DD"
            style="width: 100%"
          ></el-date-picker>
        </el-form-item>

        <el-form-item label="是否含税" prop="includeTax">
          <el-switch
            v-model="form.includeTax"
            active-text="含税"
            inactive-text="不含税"
          ></el-switch>
        </el-form-item>

        <el-form-item label="备注" prop="remark">
          <el-input
            v-model="form.remark"
            type="textarea"
            :rows="4"
            placeholder="请输入备注信息"
            maxlength="200"
            show-word-limit
          ></el-input>
        </el-form-item>

        <div class="form-actions">
          <el-button @click="handleCancel">取消</el-button>
          <el-button type="primary" @click="handleSubmit">
            <el-icon><Check></Check></el-icon>
            提交
          </el-button>
        </div>
      </el-form>
    </div>
  </div>

  <script src="https:https:https:https://unpkg.com/vue@3.4.38/dist/vue.global.prod.js"></script>
  <script src="https:https:https:https://unpkg.com/element-plus@2.9.1"></script>
  <script src="https:https:https:https://unpkg.com/@element-plus/icons-vue@2.3.1"></script>
  <script>
    const { createApp, ref, reactive, computed } = Vue

    const app = createApp({
      setup() {
        const formRef = ref(null)

        const form = reactive({
          title: '',
          customerName: '',
          email: '',
          amount: null,
          type: '标准',
          deliveryDate: '',
          includeTax: false,
          remark: ''
        })

        const rules = {
          title: [
            { required: true, message: '请输入订单标题', trigger: 'blur' },
            { min: 2, max: 50, message: '标题长度在 2 到 50 个字符之间', trigger: 'blur' }
          ],
          customerName: [
            { required: true, message: '请输入客户名称', trigger: 'blur' }
          ],
          email: [
            { required: true, message: '请输入联系邮箱', trigger: 'blur' },
            { type: 'email', message: '请输入正确的邮箱格式', trigger: 'blur' }
          ],
          amount: [
            { required: true, message: '请输入订单金额', trigger: 'blur' },
            { type: 'number', min: 0.01, message: '金额必须大于 0', trigger: 'blur' }
          ],
          type: [
            { required: true, message: '请选择订单类型', trigger: 'change' }
          ],
          deliveryDate: [
            { required: true, message: '请选择交付日期', trigger: 'change' }
          ]
        }

        const handleSubmit = () => {
          if (!formRef.value) return
          formRef.value.validate((valid) => {
            if (valid) {
              ElMessage.success('订单提交成功')
              console.log('提交数据:', JSON.stringify(form))
            } else {
              ElMessage.warning('请完善表单信息')
            }
          })
        }

        const handleCancel = () => {
          ElMessageBox.confirm('确定要取消吗？未保存的内容将丢失。', '提示', {
            confirmButtonText: '确定',
            cancelButtonText: '继续编辑',
            type: 'warning'
          }).then(() => {
            formRef.value && formRef.value.resetFields()
            ElMessage.info('已取消')
          }).catch(() => {})
        }

        return {
          formRef,
          form,
          rules,
          handleSubmit,
          handleCancel
        }
      }
    })

    // 注册 Element Plus
    app.use(ElementPlus, { locale: ElementPlus.lang.zhCn })

    // 注册所有图标
    for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
      app.component(key, component)
    }

    app.mount('#app')
  </script>
</body>
</html>
```


## 3. 详情页模板

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>订单详情 - 详情页</title>
  <link rel="stylesheet" href="https:https:https:https://unpkg.com/element-plus@2.9.1/dist/index.css" />
  <style>
    body {
      margin: 0;
      padding: 20px;
      background: #f5f7fa;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
    }
    .page-container {
      max-width: 1000px;
      margin: 0 auto;
    }
    .page-header {
      display: flex;
      justify-content: space-between;
      align-items: center;
      margin-bottom: 20px;
      background: #fff;
      border-radius: 8px;
      padding: 16px 24px;
      box-shadow: 0 2px 12px rgba(0, 0, 0, 0.06);
    }
    .page-header h2 {
      margin: 0;
      font-size: 20px;
      color: #303133;
    }
    .detail-card {
      margin-bottom: 20px;
    }
    .card-header {
      display: flex;
      align-items: center;
      font-size: 16px;
      font-weight: 600;
      color: #303133;
    }
    .card-header .el-icon {
      margin-right: 8px;
      color: #409eff;
    }
  </style>
</head>
<body>
  <div id="app">
    <div class="page-container">
      <!-- 页面标题 -->
      <div class="page-header">
        <h2>订单详情</h2>
        <div>
          <el-button type="primary" @click="handleEdit">
            <el-icon><Edit></Edit></el-icon>
            编辑
          </el-button>
          <el-button @click="handleBack">
            <el-icon><Back></Back></el-icon>
            返回
          </el-button>
        </div>
      </div>

      <!-- 基本信息 -->
      <el-card class="detail-card" shadow="hover">
        <template #header>
          <div class="card-header">
            <el-icon><Document></Document></el-icon>
            基本信息
          </div>
        </template>
        <el-descriptions :column="2" border>
          <el-descriptions-item label="订单编号">{{ detail.orderNo }}</el-descriptions-item>
          <el-descriptions-item label="订单标题">{{ detail.title }}</el-descriptions-item>
          <el-descriptions-item label="客户名称">{{ detail.customerName }}</el-descriptions-item>
          <el-descriptions-item label="联系邮箱">{{ detail.email }}</el-descriptions-item>
          <el-descriptions-item label="订单金额">
            <span style="color: #f56c6c; font-weight: 600; font-size: 16px">¥ {{ detail.amount }}</span>
          </el-descriptions-item>
          <el-descriptions-item label="订单类型">
            <el-tag type="warning">{{ detail.type }}</el-tag>
          </el-descriptions-item>
          <el-descriptions-item label="订单状态">
            <el-tag :type="detail.status === '已完成' ? 'success' : detail.status === '进行中' ? 'primary' : 'info'">
              {{ detail.status }}
            </el-tag>
          </el-descriptions-item>
          <el-descriptions-item label="是否含税">
            <el-tag :type="detail.includeTax ? 'success' : 'info'">
              {{ detail.includeTax ? '含税' : '不含税' }}
            </el-tag>
          </el-descriptions-item>
        </el-descriptions>
      </el-card>

      <!-- 时间信息 -->
      <el-card class="detail-card" shadow="hover">
        <template #header>
          <div class="card-header">
            <el-icon><Calendar></Calendar></el-icon>
            时间信息
          </div>
        </template>
        <el-descriptions :column="2" border>
          <el-descriptions-item label="创建时间">{{ detail.createdAt }}</el-descriptions-item>
          <el-descriptions-item label="更新时间">{{ detail.updatedAt }}</el-descriptions-item>
          <el-descriptions-item label="交付日期">{{ detail.deliveryDate }}</el-descriptions-item>
          <el-descriptions-item label="创建人">{{ detail.createdBy }}</el-descriptions-item>
        </el-descriptions>
      </el-card>

      <!-- 备注信息 -->
      <el-card class="detail-card" shadow="hover">
        <template #header>
          <div class="card-header">
            <el-icon><ChatDotRound></ChatDotRound></el-icon>
            备注信息
          </div>
        </template>
        <el-descriptions :column="1" border>
          <el-descriptions-item label="备注">{{ detail.remark }}</el-descriptions-item>
        </el-descriptions>
      </el-card>
    </div>
  </div>

  <script src="https:https:https:https://unpkg.com/vue@3.4.38/dist/vue.global.prod.js"></script>
  <script src="https:https:https:https://unpkg.com/element-plus@2.9.1"></script>
  <script src="https:https:https:https://unpkg.com/@element-plus/icons-vue@2.3.1"></script>
  <script>
    const { createApp, ref, reactive, computed } = Vue

    const app = createApp({
      setup() {
        const detail = reactive({
          orderNo: 'ORD-2024-001568',
          title: '华东区域Q3季度办公设备采购',
          customerName: '上海锦程科技有限公司',
          email: 'procurement@jincheng-tech.com',
          amount: '128,500.00',
          type: '加急',
          status: '进行中',
          includeTax: true,
          deliveryDate: '2024-08-15',
          createdAt: '2024-06-20 10:30:00',
          updatedAt: '2024-06-25 14:20:00',
          createdBy: '张伟',
          remark: '客户要求在8月15日前完成全部设备交付，包含50台笔记本电脑、20台显示器及相关配件。需提供三年质保服务，发票开具增值税专用发票。'
        })

        const handleEdit = () => {
          ElMessage.info('跳转到编辑页面')
        }

        const handleBack = () => {
          ElMessage.info('返回列表页面')
        }

        return {
          detail,
          handleEdit,
          handleBack
        }
      }
    })

    // 注册 Element Plus
    app.use(ElementPlus, { locale: ElementPlus.lang.zhCn })

    // 注册所有图标
    for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
      app.component(key, component)
    }

    app.mount('#app')
  </script>
</body>
</html>
```


## 4. 仪表盘模板

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>销售概览 - 仪表盘</title>
  <link rel="stylesheet" href="https:https:https:https://unpkg.com/element-plus@2.9.1/dist/index.css" />
  <style>
    body {
      margin: 0;
      padding: 20px;
      background: #f5f7fa;
      font-family: -apple-system, BlinkMacSystemFont, "Segoe UI", Roboto, "Helvetica Neue", Arial, sans-serif;
    }
    .page-container {
      max-width: 1200px;
      margin: 0 auto;
    }
    .page-header {
      margin-bottom: 20px;
    }
    .page-header h2 {
      margin: 0;
      font-size: 20px;
      color: #303133;
    }
    .stat-card {
      height: 100%;
    }
    .stat-card .stat-content {
      display: flex;
      align-items: center;
      justify-content: space-between;
    }
    .stat-card .stat-icon {
      width: 56px;
      height: 56px;
      border-radius: 12px;
      display: flex;
      align-items: center;
      justify-content: center;
      font-size: 28px;
      color: #fff;
      flex-shrink: 0;
    }
    .stat-card .stat-info {
      text-align: right;
    }
    .stat-card .stat-label {
      font-size: 14px;
      color: #909399;
      margin-bottom: 8px;
    }
    .stat-card .stat-value {
      font-size: 24px;
      font-weight: 700;
      color: #303133;
    }
    .stat-card .stat-footer {
      margin-top: 12px;
      padding-top: 12px;
      border-top: 1px solid #f0f0f0;
      font-size: 13px;
      color: #909399;
    }
    .stat-card .stat-footer .trend-up {
      color: #67c23a;
    }
    .stat-card .stat-footer .trend-down {
      color: #f56c6c;
    }
    .section-row {
      margin-top: 20px;
    }
    .card-header-title {
      display: flex;
      align-items: center;
      justify-content: space-between;
    }
    .card-header-title span {
      font-size: 16px;
      font-weight: 600;
      color: #303133;
    }
    .progress-item {
      margin-bottom: 20px;
    }
    .progress-item:last-child {
      margin-bottom: 0;
    }
    .progress-label {
      display: flex;
      justify-content: space-between;
      margin-bottom: 8px;
      font-size: 14px;
      color: #606266;
    }
  </style>
</head>
<body>
  <div id="app">
    <div class="page-container">
      <!-- 页面标题 -->
      <div class="page-header">
        <h2>销售概览</h2>
      </div>

      <!-- 统计卡片 -->
      <el-row :gutter="20">
        <el-col :span="6" v-for="stat in statCards" :key="stat.label">
          <el-card class="stat-card" shadow="hover">
            <div class="stat-content">
              <div class="stat-icon" :style="{ background: stat.color }">
                <el-icon><component :is="stat.icon"></component></el-icon>
              </div>
              <div class="stat-info">
                <div class="stat-label">{{ stat.label }}</div>
                <div class="stat-value">{{ stat.value }}</div>
              </div>
            </div>
            <div class="stat-footer">
              较上月
              <span :class="stat.trendUp ? 'trend-up' : 'trend-down'">
                <el-icon><component :is="stat.trendUp ? 'Top' : 'Bottom'"></component></el-icon>
                {{ stat.trend }}
              </span>
            </div>
          </el-card>
        </el-col>
      </el-row>

      <!-- 第二行：最近订单 + 业绩指标 -->
      <el-row :gutter="20" class="section-row">
        <!-- 最近订单 -->
        <el-col :span="14">
          <el-card shadow="hover">
            <template #header>
              <div class="card-header-title">
                <span>最近订单</span>
                <el-button type="primary" link>
                  查看全部
                  <el-icon><ArrowRight></ArrowRight></el-icon>
                </el-button>
              </div>
            </template>
            <el-table :data="recentOrders" stripe style="width: 100%">
              <el-table-column prop="orderNo" label="订单编号" width="150"></el-table-column>
              <el-table-column prop="customer" label="客户" min-width="120"></el-table-column>
              <el-table-column prop="amount" label="金额" width="120" align="right">
                <template #default="{ row }">
                  <span style="color: #f56c6c; font-weight: 600">¥ {{ row.amount }}</span>
                </template>
              </el-table-column>
              <el-table-column prop="status" label="状态" width="90" align="center">
                <template #default="{ row }">
                  <el-tag
                    :type="row.status === '已完成' ? 'success' : row.status === '进行中' ? 'primary' : row.status === '待审核' ? 'warning' : 'info'"
                    size="small"
                  >{{ row.status }}</el-tag>
                </template>
              </el-table-column>
              <el-table-column prop="date" label="日期" width="110" align="center"></el-table-column>
            </el-table>
          </el-card>
        </el-col>

        <!-- 业绩指标 -->
        <el-col :span="10">
          <el-card shadow="hover">
            <template #header>
              <div class="card-header-title">
                <span>业绩指标</span>
                <el-button type="primary" link>
                  详情
                  <el-icon><ArrowRight></ArrowRight></el-icon>
                </el-button>
              </div>
            </template>
            <div class="progress-item" v-for="item in progressData" :key="item.label">
              <div class="progress-label">
                <span>{{ item.label }}</span>
                <span>{{ item.value }}%</span>
              </div>
              <el-progress
                :percentage="item.value"
                :color="item.color"
                :stroke-width="10"
                :show-text="false"
              ></el-progress>
            </div>
          </el-card>
        </el-col>
      </el-row>
    </div>
  </div>

  <script src="https:https:https:https://unpkg.com/vue@3.4.38/dist/vue.global.prod.js"></script>
  <script src="https:https:https:https://unpkg.com/element-plus@2.9.1"></script>
  <script src="https:https:https:https://unpkg.com/@element-plus/icons-vue@2.3.1"></script>
  <script>
    const { createApp, ref, reactive, computed } = Vue

    const app = createApp({
      setup() {
        // 统计卡片数据
        const statCards = ref([
          {
            label: '总销售额',
            value: '¥ 1,286,500',
            icon: 'Money',
            color: 'linear-gradient(135deg, #667eea 0%, #764ba2 100%)',
            trend: '12.5%',
            trendUp: true
          },
          {
            label: '订单数',
            value: '1,893',
            icon: 'ShoppingCart',
            color: 'linear-gradient(135deg, #f093fb 0%, #f5576c 100%)',
            trend: '8.2%',
            trendUp: true
          },
          {
            label: '客户数',
            value: '568',
            icon: 'User',
            color: 'linear-gradient(135deg, #4facfe 0%, #00f2fe 100%)',
            trend: '3.1%',
            trendUp: true
          },
          {
            label: '转化率',
            value: '24.8%',
            icon: 'TrendCharts',
            color: 'linear-gradient(135deg, #43e97b 0%, #38f9d7 100%)',
            trend: '1.2%',
            trendUp: false
          }
        ])

        // 最近订单数据
        const recentOrders = ref([
          { orderNo: 'ORD-2024-1568', customer: '锦程科技', amount: '128,500', status: '进行中', date: '2024-06-25' },
          { orderNo: 'ORD-2024-1567', customer: '华美集团', amount: '86,200', status: '已完成', date: '2024-06-24' },
          { orderNo: 'ORD-2024-1566', customer: '天宇电子', amount: '45,800', status: '待审核', date: '2024-06-24' },
          { orderNo: 'ORD-2024-1565', customer: '博远贸易', amount: '92,100', status: '已完成', date: '2024-06-23' },
          { orderNo: 'ORD-2024-1564', customer: '新创软件', amount: '156,000', status: '进行中', date: '2024-06-22' }
        ])

        // 业绩指标数据
        const progressData = ref([
          { label: '季度目标完成率', value: 78, color: '#409eff' },
          { label: '客户满意度', value: 92, color: '#67c23a' },
          { label: '订单交付率', value: 85, color: '#e6a23c' },
          { label: '新客户增长率', value: 64, color: '#f56c6c' },
          { label: '复购率', value: 71, color: '#909399' }
        ])

        return {
          statCards,
          recentOrders,
          progressData
        }
      }
    })

    // 注册 Element Plus
    app.use(ElementPlus, { locale: ElementPlus.lang.zhCn })

    // 注册所有图标
    for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
      app.component(key, component)
    }

    app.mount('#app')
  </script>
</body>
</html>
```