# 图标映射参考文档

> 本文档定义业务概念到 Element Plus 内置图标的映射关系。Skill 在生成 DEMO 时按需加载。

## 1. 操作按钮图标映射

| 操作 | 图标组件名 | 使用场景 |
|------|-----------|---------|
| 新增 | Plus | 列表页顶部"新增"按钮 |
| 编辑 | Edit | 表格操作列"编辑"按钮 |
| 删除 | Delete | 表格操作列"删除"按钮 |
| 搜索 | Search | 搜索栏"搜索"按钮 |
| 导出 | Download | 列表页顶部"导出"按钮 |
| 刷新 | Refresh | 列表页顶部"刷新"按钮 |
| 查看详情 | View | 表格操作列"查看"按钮 |
| 返回 | Back | 详情页"返回"按钮 |
| 提交 | Check | 表单页"提交"按钮 |
| 取消 | Close | 表单页/弹窗"取消"按钮 |
| 重置 | RefreshRight | 搜索栏"重置"按钮 |
| 上传 | Upload | 文件上传按钮 |
| 保存 | Select | 保存操作 |
| 打印 | Printer | 打印操作 |
| 批量删除 | Delete | 批量操作按钮 |

## 2. 业务概念图标映射（侧边栏/导航/卡片）

| 业务概念 | 图标组件名 | 常见场景 |
|---------|-----------|---------|
| 仪表盘/首页 | HomeFilled | 侧边栏导航、仪表盘标题 |
| 用户/账户 | User | 用户管理模块 |
| 用户组/团队 | UserFilled | 团队管理 |
| 设置/配置 | Setting | 系统设置 |
| 通知/消息 | Bell | 消息中心 |
| 邮件 | Message | 邮件模块 |
| 文档/文件 | Document | 文档管理 |
| 文件夹 | Folder | 文件分类 |
| 数据分析 | DataAnalysis | 数据分析模块 |
| 趋势/图表 | TrendCharts | 统计图表 |
| 数据看板 | DataBoard | 数据面板 |
| 订单/购物 | ShoppingCart | 订单管理 |
| 商品 | Goods | 商品管理 |
| 钱包/财务 | Wallet | 财务模块 |
| 金额/收入 | Money | 收入统计 |
| 价格标签 | PriceTag | 价格管理 |
| 日历/日程 | Calendar | 日程管理 |
| 时钟/时间 | Clock | 时间相关 |
| 位置/地图 | Location | 位置信息 |
| 安全/锁 | Lock | 权限管理 |
| 钥匙 | Key | 密钥管理 |
| 帮助/支持 | QuestionFilled | 帮助中心 |
| 信息 | InfoFilled | 信息提示 |
| 成功/完成 | CircleCheckFilled | 成功状态 |
| 警告 | WarningFilled | 警告状态 |
| 错误/失败 | CircleCloseFilled | 错误状态 |
| 链接/连接 | Link | 链接管理 |
| 手机/移动 | Cellphone | 移动端相关 |
| 电脑/桌面 | Monitor | 桌面端相关 |
| 菜单 | Menu | 菜单管理 |
| 更多 | MoreFilled | 更多操作 |
| 筛选/过滤 | Filter | 筛选功能 |
| 排序 | Sort | 排序功能 |
| 收藏/星标 | Star | 收藏功能 |
| 评分 | StarFilled | 评分功能 |
| 标签 | CollectionTag | 标签管理 |
| CPU/系统 | Cpu | 系统监控 |

## 3. 仪表盘统计卡片图标

| 统计指标类型 | 推荐图标 | 推荐颜色 |
|------------|---------|---------|
| 总数/数量 | DataLine | #409EFF (蓝色) |
| 金额/收入 | Money | #67C23A (绿色) |
| 用户/人数 | User | #E6A23C (橙色) |
| 增长/趋势 | TrendCharts | #F56C6C (红色) |
| 订单 | ShoppingCart | #909399 (灰色) |
| 完成率 | CircleCheckFilled | #67C23A (绿色) |

## 4. 图标使用指南（Vue 3 CDN 模式）

在 CDN 模式下，图标通过全局注册后直接作为组件使用：

**注册方式**（在 app.mount 之前）：
```javascript
for (const [key, component] of Object.entries(ElementPlusIconsVue)) {
  app.component(key, component)
}
```

**在按钮中使用**：
```html
<el-button type="primary" @click="handleAdd">
  <el-icon><Plus></Plus></el-icon>新增
</el-button>
```

**在表格操作列中使用**：
```html
<el-button link type="primary" @click="handleEdit(scope.row)">
  <el-icon><Edit></Edit></el-icon>编辑
</el-button>
```

**在统计卡片中使用**：
```html
<el-icon :size="48" color="#409EFF"><DataLine></DataLine></el-icon>
```
