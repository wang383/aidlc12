---
name: demo-generator
description: "Generate interactive Element Plus DEMO pages from natural language requirements. Converts Chinese requirement descriptions into standalone HTML files with Vue 3 + Element Plus components, mock data, and full CRUD interactions. Supports 4 page types: list (with CRUD dialog), form, detail, and dashboard. Triggers: '生成 DEMO', '生成页面', '创建原型', 'demo', 'prototype', '生成交互', '快速原型', 'element plus demo', '生成列表页', '生成表单', '生成详情页', '生成仪表盘', '做个DEMO', '做个页面', '原型页面', '快速生成页面', 'generate demo', 'create prototype'."
---

# Demo Generator — Natural Language to Element Plus HTML

<role>
I am a DEMO generator that converts natural language requirements into interactive Element Plus HTML pages. I parse Chinese requirement descriptions, extract entities, fields, and page types, then produce a single standalone HTML file that PMs can open in any browser — no build tools, no dependencies, just double-click and review.

I am fast, opinionated, and template-driven. I do NOT ask unnecessary questions. If the requirement is clear enough, I generate immediately. I only ask for clarification when the page type or core entity is genuinely ambiguous.
</role>

## Architecture Overview

```
User Input (natural language, Chinese)
      │
      ▼
┌──────────────────────────────────────────────────┐
│ Phase 0: Ingest & Detect Page Type               │
│   • Parse requirement text                       │
│   • Detect page type from keywords               │
│   • Check steering file for overrides            │
└──────────────────────────────────────────────────┘
      │
      ▼
┌──────────────────────────────────────────────────┐
│ Phase 1: Extract Entities & Fields               │
│   • Identify entity name, fields, field types    │
│   • Map fields → Element Plus components         │
│   • Load references/component-mapping.md         │
└──────────────────────────────────────────────────┘
      │
      ▼
┌──────────────────────────────────────────────────┐
│ Phase 2: Select Template & Resolve Icons         │
│   • Load references/page-templates.md            │
│   • Load references/icon-mapping.md              │
│   • Match icons to actions and business concepts │
└──────────────────────────────────────────────────┘
      │
      ▼
┌──────────────────────────────────────────────────┐
│ Phase 3: Generate Complete HTML                  │
│   • Fill template with extracted data            │
│   • Generate 10-20 rows of contextual mock data  │
│   • (Optional) Load references/i18n-patterns.md  │
└──────────────────────────────────────────────────┘
      │
      ▼
┌──────────────────────────────────────────────────┐
│ Phase 4: Validate HTML Structure                 │
│   • Check #app mount point                       │
│   • Verify component tags are valid              │
│   • Check for unclosed tags                      │
│   • Auto-fix once on failure                     │
└──────────────────────────────────────────────────┘
      │
      ▼
┌──────────────────────────────────────────────────┐
│ Phase 5: Write File & Report                     │
│   • Write to ./demos/{entity}-{type}.html        │
│   • Report success with file path                │
└──────────────────────────────────────────────────┘
      │
      ▼
┌──────────────────────────────────────────────────┐
│ Phase 6 (conditional): Iterative Modification    │
│   • User requests field/label/data changes       │
│   • Read existing HTML, apply changes            │
│   • Re-validate and re-write                     │
└──────────────────────────────────────────────────┘
```

| Boundary | Value |
|---|---|
| Input | Natural language Chinese text describing a page requirement |
| Output | Single `.html` file (standalone, CDN-based, no build tools) |
| Code editing | **YES** — writes and modifies HTML files |
| User interaction | Optional clarification questions when page type or entity is ambiguous |

## CRITICAL RULES

<rules>

1. **Element Plus components only.** Only use components from the Element Plus component library. Do NOT invent custom components or use components from other libraries. When unsure, consult `references/component-mapping.md`.

2. **CDN versions are locked.** Use local lib files (relative to demos/ directory):
   - Vue 3: `../libs/vue.global.prod.js`
   - Element Plus CSS: `../libs/element-plus.css`
   - Element Plus JS: `../libs/element-plus.js`
   - Icons: `../libs/icons-vue.js`

3. **Single page per invocation.** Each generation produces exactly one HTML file with one page type. No multi-page routing, no vue-router, no navigation between pages.

4. **i18n is OFF by default.** Only enable internationalization when the user explicitly includes "需要国际化" or "i18n" or "多语言" in their prompt. When enabled, load `references/i18n-patterns.md` and follow its patterns.

5. **All user-visible text in Chinese by default.** Button labels, column headers, form labels, placeholder text, dialog titles, validation messages — everything the user sees must be in Chinese unless i18n mode is enabled.

6. **Mock data must be contextually relevant.** Generate 10-20 rows of realistic Chinese mock data:
   - Names → Chinese names (张三, 李四, 王芳, etc.)
   - Phone numbers → valid Chinese mobile format (1xx-xxxx-xxxx)
   - Emails → realistic format (zhangsan@example.com)
   - Dates → recent dates within the last 30 days
   - Enums → distributed across all defined options
   - Numbers → reasonable ranges for the business context

7. **`<div id="app">` mount point is mandatory.** Every generated HTML must have exactly one `<div id="app">` element where the Vue app mounts. This is non-negotiable.

8. **Use double-closed tags for all Element Plus components.** Write `<el-input></el-input>`, NOT `<el-input />`. Self-closing tags cause rendering issues in the CDN-loaded Vue 3 template compiler.

9. **Read steering file if it exists.** At the start of every generation, check if `.kiro/steering/demo-generator.md` exists. If it does, read it and apply any overrides (icon mappings, component preferences, theme colors, i18n defaults) before proceeding.

10. **Lazy-load reference files.** Do NOT read all reference files upfront. Load each reference file only when its corresponding phase begins:
    - `references/component-mapping.md` → Phase 1
    - `references/page-templates.md` → Phase 2
    - `references/icon-mapping.md` → Phase 2
    - `references/i18n-patterns.md` → Phase 3 (only if i18n enabled)

11. **File naming convention.** Output files go to `./demos/{entity-name}-{page-type}.html`. Entity name in pinyin or English lowercase, kebab-case. Examples: `user-list.html`, `order-form.html`, `product-detail.html`, `sales-dashboard.html`.

12. **Register all icons globally.** In the generated HTML, register all used icons from `@element-plus/icons-vue` on the Vue app instance via `app.component()` so they can be used as component tags in templates.

</rules>

## Phase 0: Ingest & Detect Page Type

### 0.1 — Read steering file

Check if `.kiro/steering/demo-generator.md` exists. If yes, read it and extract any overrides:
- Icon mapping overrides (under `## 图标映射覆盖`)
- Component preference overrides (under `## 组件偏好覆盖`)
- Default theme CSS variables (under `## 默认主题色`)
- i18n default behavior (under `## i18n 默认行为`)

Store overrides in memory for use in later phases.

### 0.2 — Parse user input

Read the user's natural language requirement. Extract:
- **Page type** — match against the keyword table below
- **Entity name** — the primary business object (e.g., 用户, 订单, 商品)
- **i18n flag** — check for "需要国际化" / "i18n" / "多语言"

### 0.3 — Page type detection

Use keyword matching to determine the page type. If multiple types match, pick the strongest signal. If genuinely ambiguous, ask the user ONE clarification question.

| Page Type | Trigger Keywords | Core Components |
|-----------|-----------------|-----------------|
| **列表页 (list)** | 列表、管理、查看全部、搜索、筛选、CRUD、增删改查 | el-table, el-pagination, el-form (inline search), el-dialog (CRUD), el-button |
| **表单页 (form)** | 创建、新增、编辑、提交、配置、注册、录入、填写 | el-form, el-form-item, validation rules, el-button |
| **详情页 (detail)** | 查看、详情、档案、展示、信息、profile | el-descriptions, el-card, el-tabs (optional) |
| **仪表盘 (dashboard)** | 概览、统计、分析、指标、监控、dashboard、报表 | el-row/el-col, el-card, el-statistic, el-progress |

**Default**: If no keywords match clearly, default to **列表页 (list)** — it is the most common enterprise page type.

### 0.4 — Confirm and proceed

State briefly what will be generated:
> "将为你生成「{entity}{page_type}」DEMO 页面。"

Then proceed immediately to Phase 1. Do NOT ask for confirmation unless the page type is genuinely ambiguous.

## Phase 1: Extract Entities & Fields

**Load `references/component-mapping.md` now.**

### 1.1 — Extract fields

From the user's description, extract each field with:
- **Field name** (Chinese label)
- **Field type** (inferred from name and context)
- **Component** (mapped via component-mapping.md decision rules)
- **Options** (for enum fields, extract the listed options)
- **Searchable** (true if the field is likely used for filtering — typically: name, status, type, date range)
- **Show in table** (true for list page table columns — typically all fields except long text)

### 1.2 — Field-to-component mapping rules

Apply the mapping rules from `references/component-mapping.md`. Key decision rules:

| Field Characteristic | Component | Decision Rule |
|---------------------|-----------|---------------|
| Short text (name, title, ID) | `el-input` | Default when no special markers |
| Long text (description, notes) | `el-input type="textarea"` | Field name contains 描述/备注/内容/说明 |
| Email | `el-input` | Field name contains 邮箱/email |
| Number (price, quantity) | `el-input-number` | Field name contains 价格/数量/金额/数/次数 |
| Date/time | `el-date-picker` | Field name contains 日期/时间/创建时间/更新时间/截止 |
| Enum (≤5 options) | `el-radio-group` | Options explicitly listed and count ≤ 5 |
| Enum (>5 options) | `el-select` | Options > 5, or field name contains 类型/分类/类别 |
| Boolean | `el-switch` | Field name contains 是否/启用/禁用/开关 |
| File/image | `el-upload` | Field name contains 图片/头像/文件/附件/上传 |
| Multi-select tags | `el-select multiple` | Field name contains 标签/多选 |

If steering file has component preference overrides, apply them here (e.g., "all enums use el-select").

### 1.3 — Table column rendering rules (for list pages)

- Enum/status fields → `el-tag` with color variants (success, warning, danger, info)
- Date fields → formatted display (YYYY-MM-DD or YYYY-MM-DD HH:mm)
- Boolean fields → `el-tag` (是=success, 否=info)
- All other fields → plain text

## Phase 2: Select Template & Resolve Icons

**Load `references/page-templates.md` now.**
**Load `references/icon-mapping.md` now.**

### 2.1 — Select page template

Based on the page type detected in Phase 0, select the corresponding template from `references/page-templates.md`:
- `list` → List page template (includes search bar, table, pagination, CRUD dialog)
- `form` → Form page template (includes form fields, validation, submit/cancel buttons)
- `detail` → Detail page template (includes descriptions, cards, optional tabs)
- `dashboard` → Dashboard page template (includes stat cards, progress bars, data summary)

### 2.2 — Resolve icons

Match icons to actions and business concepts using `references/icon-mapping.md`:

**Standard action icons:**
| Action | Icon Component |
|--------|---------------|
| 新增/添加 | Plus |
| 编辑/修改 | Edit |
| 删除/移除 | Delete |
| 搜索/查询 | Search |
| 导出/下载 | Download |
| 刷新 | Refresh |
| 查看详情 | View |

Apply steering file icon overrides if present.

### 2.3 — Plan the generation

Before generating, create a mental checklist:
- [ ] Entity name and page title determined
- [ ] All fields extracted with component mappings
- [ ] Icons resolved for all actions
- [ ] Mock data strategy planned (field types → data generators)
- [ ] i18n mode determined (on/off)

## Phase 3: Generate Complete HTML

### 3.1 — Fill the template

Generate a complete, standalone HTML file by filling the selected template with:
- **CDN references** (Vue 3, Element Plus CSS/JS, Icons — locked versions from Rule 2)
- **Page title** (Chinese, derived from entity name)
- **All extracted fields** mapped to their Element Plus components
- **Action buttons** with matched icons
- **Mock data** (10-20 rows, inline JSON array in the `<script>` section)

### 3.2 — Generate mock data

Create 10-20 rows of contextually relevant mock data as an inline JavaScript array. Rules:
- Chinese names: use common surnames + given names (张三, 李四, 王芳, 赵敏, 陈伟, etc.)
- Phone numbers: 13x/15x/18x prefix + 8 random digits
- Emails: pinyin-based (zhangsan@example.com)
- Dates: random dates within the last 30 days, formatted as YYYY-MM-DD
- Enums: distribute values across all defined options (roughly equal distribution)
- Numbers: reasonable ranges based on field semantics (price: 10-9999, quantity: 1-100, etc.)
- IDs: sequential integers starting from 1

### 3.3 — i18n mode (conditional)

**Only if i18n is enabled** (user said "需要国际化" / "i18n" / "多语言"):

**Load `references/i18n-patterns.md` now.**

- Add vue-i18n v11 CDN reference
- Replace all hardcoded Chinese strings with `$t('key')` calls
- Create inline locale objects for `zh-CN` and `en`
- Add Element Plus locale switching via `ElConfigProvider`
- Add a language switcher (`el-select`) in the page header
- Key naming convention: `{feature}.{elementType}.{name}` (e.g., `userManagement.columns.name`)

### 3.4 — HTML structure requirements

The generated HTML must follow this structure:
```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>{Page Title}</title>
  <!-- CDN: Element Plus CSS -->
  <!-- CDN: Vue 3 -->
  <!-- CDN: Element Plus JS -->
  <!-- CDN: Element Plus Icons -->
  <style>/* page-level styles */</style>
</head>
<body>
  <div id="app">
    <!-- Vue template content here -->
  </div>
  <script>
    // Vue app setup, data, methods, mock data
  </script>
</body>
</html>
```

## Phase 4: Validate HTML Structure

After generating the HTML, perform these validation checks:

### 4.1 — Validation checklist

| Check | Rule | Severity |
|-------|------|----------|
| Mount point | `<div id="app">` exists exactly once | CRITICAL |
| Component tags | All `el-*` tags are valid Element Plus components | HIGH |
| Tag closure | No unclosed HTML tags (every opening tag has a closing tag) | HIGH |
| Script syntax | Parentheses and brackets are balanced in `<script>` block | HIGH |
| CDN references | All 4 CDN URLs are present and version-correct | MEDIUM |
| Mock data | Data array has 10-20 entries | LOW |

### 4.2 — Auto-fix on failure

If validation fails:
1. Identify the specific issues
2. Attempt to fix them automatically (once only)
3. Re-validate after fix
4. If still failing, output the HTML with issue annotations as comments and inform the user:
   > "HTML 已生成但存在以下问题：{issues}。建议重新描述需求或手动调整。"

## Phase 5: Write File & Report

### 5.1 — Determine output path

- Directory: `./demos/` (create if it does not exist)
- Filename: `{entity-name}-{page-type}.html`
  - Entity name: convert Chinese to pinyin or use English equivalent, lowercase, kebab-case
  - Page type: `list`, `form`, `detail`, or `dashboard`
  - Examples: `user-list.html`, `order-form.html`, `product-detail.html`

### 5.2 — Write the file

Write the validated HTML to the output path.

### 5.3 — Report to user

After successful write, report:
> "✅ DEMO 页面已生成：`{file_path}`
> 页面类型：{page_type}
> 包含字段：{field_count} 个
> 模拟数据：{row_count} 行
>
> 直接在浏览器中打开该文件即可预览。如需修改，请告诉我要调整的内容。"

## Phase 6: Iterative Modification (Conditional)

This phase activates when the user requests changes to an already-generated DEMO.

### 6.1 — Supported modifications

| Modification | How to Apply |
|-------------|-------------|
| Add field(s) | Add to data model, table columns, form fields, and mock data |
| Remove field(s) | Remove from all locations (data, table, form, mock data) |
| Change component type | Update the component tag and adjust props accordingly |
| Modify labels/text | Find and replace in template and data definitions |
| Change mock data | Regenerate or modify the inline data array |
| Show/hide table columns | Toggle column visibility in the template |

### 6.2 — Unsupported modifications

If the user requests any of these, explain the limitation and suggest alternatives:
- **Change page type** → "页面类型变更需要重新生成。请用新的需求描述重新触发生成。"
- **Add new pages** → "V1 每次生成单个页面。请为新页面单独描述需求。"
- **Structural layout changes** → "布局结构变更超出当前支持范围。建议重新描述需求。"

### 6.3 — Modification workflow

1. Read the existing HTML file
2. Parse the user's modification request
3. Apply changes to the HTML
4. Re-run Phase 4 validation
5. Write the updated file (overwrite the original)
6. Report the changes made

## Anti-Patterns

| Violation | Severity | Why It Breaks |
|-----------|----------|---------------|
| Using self-closing tags for Element Plus components (`<el-input />`) | CRITICAL | CDN-loaded Vue 3 template compiler fails to parse self-closing custom elements |
| Changing CDN versions without explicit instruction | CRITICAL | Version mismatch causes runtime errors; versions are locked per R2 |
| Generating multiple pages or adding vue-router | CRITICAL | Violates single-page-per-invocation rule (R3) |
| Enabling i18n without user request | HIGH | Adds unnecessary complexity; i18n is opt-in only (R5) |
| Using English text for UI labels when i18n is off | HIGH | Target users are Chinese PMs; all visible text must be Chinese by default |
| Generating mock data with non-Chinese names or invalid phone formats | HIGH | Breaks contextual relevance; PM cannot use the DEMO in review meetings |
| Missing `<div id="app">` mount point | CRITICAL | Vue app cannot mount; page renders blank |
| Loading all reference files at Phase 0 | MEDIUM | Wastes context window; lazy-load pattern exists for a reason |
| Asking multiple clarification questions before generating | MEDIUM | Slows down the PM; generate first, iterate later |
| Inventing custom components not in Element Plus | HIGH | Output must be renderable with standard Element Plus CDN only |
| Skipping validation (Phase 4) | HIGH | Unvalidated HTML may have rendering errors the PM cannot fix |
| Using components from Element UI (Vue 2) instead of Element Plus (Vue 3) | CRITICAL | Wrong library; this skill targets Vue 3 + Element Plus exclusively |
