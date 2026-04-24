# 国际化（i18n）代码模式参考文档

> 本文档定义 i18n 模式下的代码模式。仅当用户明确请求国际化时加载。

## 1. CDN 加载配置

Additional CDN references needed for i18n mode (add to the standard CDN set):
```html
<script src="https:https://unpkg.com/vue-i18n@11"></script>
<script src="https:https://unpkg.com/element-plus@2.9.1/dist/locale/zh-cn.mjs"></script>
```

## 2. Vue 3 App 中 i18n 初始化代码

```javascript
const { createI18n } = VueI18n

const i18n = createI18n({
  locale: 'zh-CN',
  fallbackLocale: 'en',
  messages: {
    'zh-CN': {
      // Chinese locale messages here
    },
    'en': {
      // English locale messages here
    }
  }
})

const app = createApp({ /* ... */ })
app.use(i18n)
app.use(ElementPlus)
app.mount('#app')
```

## 3. Element Plus Locale 切换

```javascript
// In setup()
const elementLocales = {
  'zh-CN': ElementPlusLocaleZhCn,  // loaded via CDN
  'en': undefined  // Element Plus defaults to English
}

const currentLocale = ref('zh-CN')
const elementLocale = computed(() => elementLocales[currentLocale.value])

const switchLocale = (lang) => {
  currentLocale.value = lang
  i18n.global.locale = lang
}
```

Template wrapper:
```html
<el-config-provider :locale="elementLocale">
  <!-- page content -->
</el-config-provider>
```

## 4. i18n Key 命名规范

Pattern: `{feature}.{elementType}.{name}`

| 元素类型 | Key 模式 | 示例 |
|---------|---------|------|
| 页面标题 | `{feature}.title` | `userManagement.title` → "用户管理" |
| 表格列头 | `{feature}.columns.{field}` | `userManagement.columns.name` → "姓名" |
| 表单标签 | `{feature}.labels.{field}` | `userManagement.labels.email` → "邮箱" |
| 按钮文字 | `{feature}.buttons.{action}` | `userManagement.buttons.create` → "新增" |
| 占位符 | `{feature}.placeholders.{field}` | `userManagement.placeholders.name` → "请输入姓名" |
| 提示消息 | `{feature}.messages.{type}` | `userManagement.messages.deleteConfirm` → "确定删除？" |
| 验证消息 | `{feature}.validation.{field}` | `userManagement.validation.nameRequired` → "请输入姓名" |
| 通用文字 | `common.actions.{action}` | `common.actions.save` → "保存" |

Common keys (shared across pages):
```json
{
  "common": {
    "actions": {
      "save": "保存",
      "cancel": "取消",
      "confirm": "确定",
      "delete": "删除",
      "edit": "编辑",
      "create": "新增",
      "search": "搜索",
      "reset": "重置",
      "export": "导出",
      "refresh": "刷新",
      "back": "返回",
      "submit": "提交"
    },
    "messages": {
      "deleteConfirm": "确定要删除吗？",
      "deleteSuccess": "删除成功",
      "saveSuccess": "保存成功",
      "submitSuccess": "提交成功",
      "operationCanceled": "已取消"
    },
    "labels": {
      "status": "状态",
      "createdAt": "创建时间",
      "updatedAt": "更新时间",
      "operation": "操作"
    }
  }
}
```


## 5. 完整 i18n 模式 HTML 示例

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>用户管理 - i18n Demo</title>
  <link rel="stylesheet" href="https:https://unpkg.com/element-plus@2.9.1/dist/index.css" />
  <script src="https:https://unpkg.com/vue@3"></script>
  <script src="https:https://unpkg.com/element-plus@2.9.1"></script>
  <script src="https:https://unpkg.com/@element-plus/icons-vue"></script>
  <script src="https:https://unpkg.com/vue-i18n@11"></script>
  <script src="https:https://unpkg.com/element-plus@2.9.1/dist/locale/zh-cn.mjs"></script>
  <style>
    body { margin: 0; padding: 20px; background: #f5f7fa; }
    .page-header { display: flex; justify-content: space-between; align-items: center; margin-bottom: 20px; }
    .page-header h2 { margin: 0; }
    .lang-switcher { width: 120px; }
  </style>
</head>
<body>
  <div id="app">
    <el-config-provider :locale="elementLocale">
      <el-card>
        <!-- Page Header with Language Switcher -->
        <div class="page-header">
          <h2>{{ $t('userManagement.title') }}</h2>
          <el-select
            v-model="currentLocale"
            class="lang-switcher"
            @change="switchLocale"
          >
            <el-option label="中文" value="zh-CN" />
            <el-option label="English" value="en" />
          </el-select>
        </div>

        <!-- Search Bar -->
        <el-form :inline="true" style="margin-bottom: 16px;">
          <el-form-item :label="$t('userManagement.labels.name')">
            <el-input
              v-model="searchForm.name"
              :placeholder="$t('userManagement.placeholders.name')"
              clearable
            />
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="handleSearch">
              <el-icon><Search /></el-icon>
              {{ $t('common.actions.search') }}
            </el-button>
            <el-button @click="handleReset">
              {{ $t('common.actions.reset') }}
            </el-button>
          </el-form-item>
        </el-form>

        <!-- Toolbar -->
        <div style="margin-bottom: 16px;">
          <el-button type="primary" @click="handleCreate">
            <el-icon><Plus /></el-icon>
            {{ $t('common.actions.create') }}
          </el-button>
        </div>

        <!-- Data Table -->
        <el-table :data="tableData" border stripe>
          <el-table-column
            prop="name"
            :label="$t('userManagement.columns.name')"
          />
          <el-table-column
            prop="email"
            :label="$t('userManagement.columns.email')"
          />
          <el-table-column
            prop="status"
            :label="$t('common.labels.status')"
          >
            <template #default="{ row }">
              <el-tag :type="row.status === 'active' ? 'success' : 'info'">
                {{ $t(`userManagement.status.${row.status}`) }}
              </el-tag>
            </template>
          </el-table-column>
          <el-table-column
            prop="createdAt"
            :label="$t('common.labels.createdAt')"
          />
          <el-table-column
            :label="$t('common.labels.operation')"
            width="200"
          >
            <template #default="{ row }">
              <el-button type="primary" link @click="handleEdit(row)">
                {{ $t('common.actions.edit') }}
              </el-button>
              <el-popconfirm
                :title="$t('common.messages.deleteConfirm')"
                :confirm-button-text="$t('common.actions.confirm')"
                :cancel-button-text="$t('common.actions.cancel')"
                @confirm="handleDelete(row)"
              >
                <template #reference>
                  <el-button type="danger" link>
                    {{ $t('common.actions.delete') }}
                  </el-button>
                </template>
              </el-popconfirm>
            </template>
          </el-table-column>
        </el-table>

        <!-- Pagination -->
        <div style="margin-top: 16px; display: flex; justify-content: flex-end;">
          <el-pagination
            background
            layout="total, sizes, prev, pager, next"
            :total="100"
            :page-sizes="[10, 20, 50]"
          />
        </div>
      </el-card>
    </el-config-provider>
  </div>

  <script>
    const { createApp, ref, reactive, computed } = Vue
    const { createI18n } = VueI18n

    // i18n messages
    const messages = {
      'zh-CN': {
        userManagement: {
          title: '用户管理',
          columns: {
            name: '姓名',
            email: '邮箱'
          },
          labels: {
            name: '姓名',
            email: '邮箱'
          },
          placeholders: {
            name: '请输入姓名',
            email: '请输入邮箱'
          },
          status: {
            active: '启用',
            inactive: '禁用'
          }
        },
        common: {
          actions: {
            save: '保存',
            cancel: '取消',
            confirm: '确定',
            delete: '删除',
            edit: '编辑',
            create: '新增',
            search: '搜索',
            reset: '重置'
          },
          messages: {
            deleteConfirm: '确定要删除吗？',
            deleteSuccess: '删除成功',
            saveSuccess: '保存成功'
          },
          labels: {
            status: '状态',
            createdAt: '创建时间',
            updatedAt: '更新时间',
            operation: '操作'
          }
        }
      },
      'en': {
        userManagement: {
          title: 'User Management',
          columns: {
            name: 'Name',
            email: 'Email'
          },
          labels: {
            name: 'Name',
            email: 'Email'
          },
          placeholders: {
            name: 'Enter name',
            email: 'Enter email'
          },
          status: {
            active: 'Active',
            inactive: 'Inactive'
          }
        },
        common: {
          actions: {
            save: 'Save',
            cancel: 'Cancel',
            confirm: 'Confirm',
            delete: 'Delete',
            edit: 'Edit',
            create: 'Create',
            search: 'Search',
            reset: 'Reset'
          },
          messages: {
            deleteConfirm: 'Are you sure to delete?',
            deleteSuccess: 'Deleted successfully',
            saveSuccess: 'Saved successfully'
          },
          labels: {
            status: 'Status',
            createdAt: 'Created At',
            updatedAt: 'Updated At',
            operation: 'Actions'
          }
        }
      }
    }

    // Create i18n instance
    const i18n = createI18n({
      locale: 'zh-CN',
      fallbackLocale: 'en',
      messages
    })

    const app = createApp({
      setup() {
        // Element Plus locale switching
        const elementLocales = {
          'zh-CN': ElementPlusLocaleZhCn,
          'en': undefined
        }

        const currentLocale = ref('zh-CN')
        const elementLocale = computed(() => elementLocales[currentLocale.value])

        const switchLocale = (lang) => {
          currentLocale.value = lang
          i18n.global.locale = lang
        }

        // Search form
        const searchForm = reactive({ name: '' })

        // Mock table data
        const tableData = ref([
          { id: 1, name: '张三', email: 'zhangsan@example.com', status: 'active', createdAt: '2024-01-15' },
          { id: 2, name: '李四', email: 'lisi@example.com', status: 'inactive', createdAt: '2024-02-20' },
          { id: 3, name: '王五', email: 'wangwu@example.com', status: 'active', createdAt: '2024-03-10' }
        ])

        // Event handlers
        const handleSearch = () => { console.log('Search:', searchForm) }
        const handleReset = () => { searchForm.name = '' }
        const handleCreate = () => { ElMessage.success(i18n.global.t('common.messages.saveSuccess')) }
        const handleEdit = (row) => { console.log('Edit:', row) }
        const handleDelete = (row) => { ElMessage.success(i18n.global.t('common.messages.deleteSuccess')) }

        return {
          currentLocale,
          elementLocale,
          switchLocale,
          searchForm,
          tableData,
          handleSearch,
          handleReset,
          handleCreate,
          handleEdit,
          handleDelete
        }
      }
    })

    // Register icon components
    for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
      app.component(key, component)
    }

    app.use(i18n)
    app.use(ElementPlus)
    app.mount('#app')
  </script>
</body>
</html>
```

## 6. 非 i18n 模式 vs i18n 模式差异对照

| 位置 | 非 i18n 模式 | i18n 模式 |
|------|-------------|----------|
| CDN 引用 | 4 个（Vue + EP CSS + EP JS + Icons） | 6 个（+ vue-i18n + EP locale） |
| 页面标题 | `<h2>用户管理</h2>` | `<h2>{{ $t('userManagement.title') }}</h2>` |
| 表格列头 | `label="姓名"` | `:label="$t('userManagement.columns.name')"` |
| 按钮文字 | `新增` | `{{ $t('common.actions.create') }}` |
| 占位符 | `placeholder="请输入姓名"` | `:placeholder="$t('userManagement.placeholders.name')"` |
| 验证消息 | `message: '请输入姓名'` | `message: i18n.global.t('userManagement.validation.nameRequired')` |
| App 初始化 | `app.use(ElementPlus)` | `app.use(i18n); app.use(ElementPlus)` |
| 模板包裹 | 无 | `<el-config-provider :locale="elementLocale">` |