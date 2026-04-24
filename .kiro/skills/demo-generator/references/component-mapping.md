# Element Plus 组件映射决策树

> 本文档定义了「字段特征 → Element Plus 组件」的完整决策规则，供 demo-generator 技能在生成 CRUD 页面时参考。

---

## 1. 字段特征 → 组件映射表

| 字段特征 | 组件 | 决策规则 |
|---------|------|---------|
| 短文本（名称、标题、编号） | `el-input` | 默认，无特殊标记 |
| 长文本（描述、备注、内容） | `el-input type="textarea"` | 字段名含"描述/备注/内容/说明"，或明确标注"多行" |
| 邮箱 | `el-input` | 字段名含"邮箱/email" |
| 数字（价格、数量、金额） | `el-input-number` | 字段名含"价格/数量/金额/数/次数" |
| 日期 | `el-date-picker` | 字段名含"日期/时间/创建时间/更新时间/截止" |
| 枚举（≤5 个选项） | `el-radio-group` | 字段明确列出选项且 ≤5 个 |
| 枚举（>5 个选项） | `el-select` | 字段明确列出选项且 >5 个，或字段名含"类型/分类/类别" |
| 布尔 | `el-switch` | 字段名含"是否/启用/禁用/开关" |
| 文件/图片 | `el-upload` | 字段名含"图片/头像/文件/附件/上传" |
| 多选标签 | `el-select multiple` | 字段名含"标签/多选" |

---

## 2. 表格列渲染规则

| 字段类型 | 表格列渲染 | 代码模式 |
|---------|-----------|---------|
| 枚举状态字段 | `el-tag` with type variants | `<el-tag :type="statusTypeMap[scope.row.status]">` — success/warning/danger/info |
| 日期字段 | 格式化显示 | 直接显示或用 computed 格式化为 `YYYY-MM-DD` |
| 布尔字段 | `el-tag` | `<el-tag :type="scope.row.field ? 'success' : 'info'">` 是/否 |
| 其他 | 纯文本 | 直接 `{{ scope.row.fieldName }}` |

---

## 3. 表单验证规则映射

| 场景 | 验证规则 | 代码模式 |
|------|---------|---------|
| 必填字段 | `required: true` | `{ required: true, message: '请输入XXX', trigger: 'blur' }` |
| 邮箱格式 | `type: 'email'` | `{ type: 'email', message: '请输入正确的邮箱地址', trigger: 'blur' }` |
| 数字范围 | `type: 'number'` | `{ type: 'number', message: '请输入数字', trigger: 'blur' }` |
| 手机号 | `pattern` | `{ pattern: /^1[3-9]\d{9}$/, message: '请输入正确的手机号', trigger: 'blur' }` |

---

## 4. 歧义处理指南

When field characteristics are ambiguous:

- **无法确定类型时**，默认使用 `el-input`（短文本）
- **字段名含"状态"但未列出选项时**，默认使用 `el-select`，选项为 `["启用", "禁用"]`
- **字段名含"类型"但未列出选项时**，默认使用 `el-select`，提示用户补充选项
- **同时匹配多个规则时**，优先级：明确列出选项 > 字段名关键词 > 默认

---

## 5. 搜索栏字段选择规则

Not all fields should appear in the search bar. Rules:

- **枚举字段**（状态、类型、角色等）→ 放入搜索栏，使用 `el-select`
- **文本字段**（名称、标题）→ 放入搜索栏，使用 `el-input`
- **日期字段** → 放入搜索栏，使用 `el-date-picker`
- **数字、布尔、文件、长文本** → 不放入搜索栏
- 搜索栏最多 **4 个字段**，超出时选择最常用的 4 个
