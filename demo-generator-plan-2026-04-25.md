# 实施计划：Element Plus 标准交互 DEMO 自动生成 Skill

> **Source PRD**: `demo-generator-prd-2026-04-25.md`
> **Tier**: Standard
> **Date**: 2026-04-25

---

## 上下文与研究

- PRD 定义了 10 个需求（R1-R10），1 个验收示例（AE1），4 个成功指标（F1-F4）
- 技术栈：Vue 3 + Element Plus + CDN，输出独立 HTML 文件
- 架构：模板填充式（方案 A）——SKILL.md 指令 + 4 套 HTML 模板参考文档 + 组件/图标映射参考文档
- 代码库状态：全新 skill，无现有代码可复用
- 现有 skill 约定：`.kiro/skills/{name}/SKILL.md` + `references/` + `assets/`，frontmatter 含 name + description，SKILL.md < 500 行，reference 文件 lazy-load

---

## 关键技术决策

| # | 决策 | 理由 | 替代方案 |
|---|------|------|---------|
| D1 | 单步生成策略（一次 LLM 调用输出完整 HTML） | PRD F1 要求 < 60 秒；减少失败点 | 多步提取+填充（V2） |
| D2 | 模板作为 reference 文件（page-templates.md），非独立 .html | LLM 需在上下文中看到模板 | 独立 assets/ 文件 |
| D3 | 组件映射和图标映射作为独立 reference 文件 | 遵循 lazy-load 模式，SKILL.md 不超 500 行 | 内联到 SKILL.md |

---

## 实施单元

#### U1: SKILL.md 主指令文件
- **Goal**: 创建 demo-generator skill 的核心指令文件，定义角色、流程、规则和触发条件
- **Requirements**: R1, R2, R3, R4, R5, R6, R7, R8, R9, R10, AE1
- **Dependencies**: None
- **Files**: .kiro/skills/demo-generator/SKILL.md
- **Approach**: 编写 SKILL.md，包含 YAML frontmatter（name: demo-generator, description 含触发词）、role 声明（"我是 DEMO 生成器"）、架构概览（输入→解析→选模板→生成HTML→校验→输出）、边界表（输入/输出/约束）、关键规则（Element Plus 组件约束、CDN 版本锁定、单页面范围、i18n 默认关闭）、分步指令（Phase 0-5）、反模式表。指令中引用 references/ 下的 3 个文件并标注 lazy-load 时机。控制在 400-500 行内。
- **Test scenarios**:
  - Skill 被"生成 DEMO"、"生成页面"、"创建原型"等触发词正确触发
  - SKILL.md frontmatter 通过 quick_validate.py 校验
  - 指令中引用的所有 reference 文件路径正确
- **Verification**: 运行 `python .kiro/skills/skill-creator/scripts/quick_validate.py .kiro/skills/demo-generator` 返回 "Skill is valid!"

#### U2: 页面模板参考文档（page-templates.md）
- **Goal**: 创建包含 4 种页面类型完整 HTML 代码模板的参考文档，供 SKILL.md 在生成时 lazy-load
- **Requirements**: R2, R3, R7
- **Dependencies**: None
- **Files**: .kiro/skills/demo-generator/references/page-templates.md
- **Approach**: 编写 page-templates.md，包含 4 个 section（列表页、表单页、详情页、仪表盘），每个 section 提供一套完整的、可运行的 HTML 模板代码。模板使用 Vue 3 CDN + Element Plus CDN + @element-plus/icons-vue CDN（版本锁定）。每套模板包含：HTML 骨架、CDN 引用、Vue app 初始化、Element Plus 注册、图标注册、模拟数据占位、组件结构。模板中用 `{{PLACEHOLDER}}` 标记注入点（页面标题、字段列表、表格列、搜索字段、表单字段、模拟数据、操作按钮）。列表页模板额外包含 CRUD 弹窗（el-dialog + el-form）。每套模板确保双击 .html 文件即可在浏览器中正常渲染。
- **Test scenarios**:
  - 每套模板的 HTML 代码复制到 .html 文件后，在浏览器中打开无 JS 错误
  - 模板包含所有 PRD 规格图中定义的组件（el-table, el-pagination, el-form, el-dialog 等）
  - 占位符标记清晰且不与 Vue 模板语法冲突
  - CDN 版本与 PRD R2 中锁定的版本一致
- **Verification**: 将列表页模板代码保存为 test-list.html，用浏览器打开确认无控制台错误，页面正常渲染 Element Plus 组件

#### U3: 组件映射参考文档（component-mapping.md）
- **Goal**: 创建字段类型到 Element Plus 组件的完整决策树文档，供 SKILL.md 在解析需求时 lazy-load
- **Requirements**: R4
- **Dependencies**: None
- **Files**: .kiro/skills/demo-generator/references/component-mapping.md
- **Approach**: 编写 component-mapping.md，包含：(1) 字段特征→组件映射表（直接复用 PRD R4 的完整表格，含决策规则列）；(2) 表格列渲染规则（枚举→el-tag、日期→格式化、布尔→el-tag、其他→纯文本）；(3) 表单验证规则映射（required 字段→必填规则、邮箱→email 格式、数字→number 范围）；(4) 歧义处理指南（当字段特征不明确时的默认行为）。
- **Test scenarios**:
  - 文档覆盖 PRD R4 中列出的所有 10 种字段类型
  - 每种字段类型有明确的决策规则，无歧义
  - 表格列渲染规则覆盖 4 种情况
- **Verification**: 人工审查文档，确认每种字段类型都有唯一的组件映射路径

#### U4: 图标映射参考文档（icon-mapping.md）
- **Goal**: 创建业务概念到 Element Plus 图标的映射表文档
- **Requirements**: R6
- **Dependencies**: None
- **Files**: .kiro/skills/demo-generator/references/icon-mapping.md
- **Approach**: 编写 icon-mapping.md，包含：(1) 操作按钮图标映射表（直接复用 PRD R6 的 7 行映射）；(2) 扩展的业务概念图标映射（从研究报告 Finding 6 中提取的 30+ 映射，覆盖仪表盘、用户、设置、通知、文档、数据分析、订单、位置等常见业务概念）；(3) 图标使用指南（在 Vue 3 CDN 模式下如何引用图标组件）。
- **Test scenarios**:
  - 映射表覆盖 PRD R6 中的所有 7 个场景
  - 所有图标名称是 Element Plus @element-plus/icons-vue 中的有效组件名
  - 包含 CDN 模式下的图标使用代码示例
- **Verification**: 人工审查文档，确认所有图标名称在 Element Plus 图标库中存在

#### U5: i18n 模式参考文档（i18n-patterns.md）
- **Goal**: 创建国际化代码模式文档，供 SKILL.md 在用户请求 i18n 时 lazy-load
- **Requirements**: R5
- **Dependencies**: None
- **Files**: .kiro/skills/demo-generator/references/i18n-patterns.md
- **Approach**: 编写 i18n-patterns.md，包含：(1) CDN 加载配置（vue-i18n v11 + Element Plus locale）；(2) Vue 3 app 中 i18n 初始化代码模式；(3) Element Plus locale 切换代码模式（ElConfigProvider + computed locale）；(4) i18n key 命名规范（`{feature}.{elementType}.{name}` 嵌套模式）；(5) 完整的 i18n 模式 HTML 示例（包含语言切换器 el-select）；(6) 与非 i18n 模式的差异对照表（哪些地方需要从硬编码改为 $t()）。
- **Test scenarios**:
  - i18n 示例代码在浏览器中可运行，语言切换功能正常
  - key 命名规范与 PRD R5 一致
  - CDN 版本与 PRD 锁定版本兼容
- **Verification**: 将 i18n 示例代码保存为 test-i18n.html，在浏览器中打开确认语言切换正常工作

#### U6: Steering 文件模板
- **Goal**: 创建 steering 文件模板，定义公司级定制接口的格式和示例
- **Requirements**: R10
- **Dependencies**: U1
- **Files**: .kiro/steering/demo-generator.md
- **Approach**: 编写 steering 文件模板，包含 4 个 section（用 H2 标题标识）：(1) `## 图标映射覆盖` — 示例格式和说明；(2) `## 组件偏好覆盖` — 示例格式和说明；(3) `## 默认主题色` — CSS 变量覆盖示例；(4) `## i18n 默认行为` — 开关配置。每个 section 包含格式说明 + 示例 + 注释。文件默认内容为注释掉的示例，用户取消注释即可启用。
- **Test scenarios**:
  - Steering 文件格式清晰，PM 或研发可理解
  - 每个 section 有可用的示例
  - SKILL.md 中的 steering 文件读取逻辑与此格式匹配
- **Verification**: 人工审查文件，确认格式与 SKILL.md 中的解析逻辑一致

---

## 并行执行计划

```
Wave 1: [U1] [U2] [U3] [U4] [U5]    // 5 个独立单元并行，无前置依赖
Wave 2: [U6 after U1]                 // 依赖 U1（需要 SKILL.md 中的 steering 解析逻辑作为参照）
```

---

## 完成状态

`DONE`

### Execution Log (2026-04-25)

**Waves executed:** W1 (5 units: U1, U2, U3, U4, U5) · W2 (1 unit: U6)
**Artifacts:**
- `.kiro/skills/demo-generator/SKILL.md` (401 lines, validated via quick_validate.py)
- `.kiro/skills/demo-generator/references/page-templates.md` (1099 lines, 4 complete HTML templates)
- `.kiro/skills/demo-generator/references/component-mapping.md` (5 sections, 10 field types)
- `.kiro/skills/demo-generator/references/icon-mapping.md` (4 sections, 50+ icon mappings)
- `.kiro/skills/demo-generator/references/i18n-patterns.md` (6 sections, complete i18n example)
- `.kiro/steering/demo-generator.md` (4 customization sections, commented-out defaults)
**Notes:** U5 failed on first dispatch (missing prompt parameter), succeeded on retry. All other units succeeded on first attempt.
