# 技术支持管理网站

基于React 18 + TypeScript + Vite的现代化技术支持管理系统，支持设备管理、打印机监控、维护记录、位置追踪、单据化操作、统计分析等功能。

## ✨ 功能特性

### 核心功能
- 📱 **设备管理**: 支持打印机、路由器、物联网卡等设备的全生命周期管理
- 📊 **统计看板**: 实时设备状态、库存水平、操作趋势可视化展示
- 📋 **单据化操作**: 所有设备操作通过事务化单据系统，确保数据一致性
- 🎯 **参数化调度**: 声明式配置调度参数，系统自动生成并执行联动动作
- 🔍 **审计日志**: 完整的操作记录和数据变更追踪
- 🔧 **SOP流程**: 标准操作程序指导和进度跟踪
- 📦 **库存管理**: 耗材库存监控、低库存告警、兼容性验证
- 🚀 **安装向导**: 模板化设备安装流程，支持预检和批量操作

### 业务特性
- **兼容性检查**: DNP只允许专码，诚研/西铁城支持专码通码二选一
- **专码绑定**: 确保专码与设备的唯一绑定关系
- **库存联动**: 实时库存余额计算和变更记录
- **位置追踪**: 设备位置变更历史和当前状态
- **维护记录**: 设备维护历史和计划管理

## 🛠 技术栈

- **前端框架**: React 18 + TypeScript
- **构建工具**: Vite
- **样式系统**: TailwindCSS + shadcn/ui
- **状态管理**: @tanstack/react-query
- **路由管理**: react-router-dom
- **表单处理**: react-hook-form + zod
- **数据可视化**: recharts
- **后端服务**: Supabase (PostgreSQL + Edge Functions)
- **测试框架**: Vitest + Playwright + Storybook

## 🚀 快速开始

### 环境要求
- Node.js >= 18
- npm >= 8
- PostgreSQL >= 14 (通过Supabase提供)

### 本地开发

1. **克隆项目**
   ```bash
   git clone [repository-url]
   cd Technical-Support-Management-Website
   ```

2. **安装依赖**
   ```bash
   npm install
   ```

3. **环境配置**
   ```bash
   cp .env.example .env
   ```
   编辑 `.env` 文件，配置以下变量：
   ```env
   VITE_SUPABASE_URL=your_supabase_url
   VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
   ```

4. **数据库迁移**
   ```bash
   # 执行数据库迁移
   npm run db:migrate

   # 插入种子数据
   npm run db:seed
   ```

5. **启动开发服务器**
   ```bash
   npm run dev
   ```

6. **访问应用**
   打开浏览器访问 http://localhost:5173

## 📋 NPM 脚本

### 开发相关
```bash
npm run dev          # 启动开发服务器
npm run build        # 构建生产版本
npm run preview      # 预览构建结果
npm run lint         # 代码检查
npm run typecheck    # 类型检查
```

### 数据库相关
```bash
npm run db:migrate   # 执行数据库迁移
npm run db:seed      # 插入种子数据
npm run db:reset     # 重置数据库（谨慎使用）
```

### 测试相关
```bash
npm run test:unit    # 运行单元测试
npm run test:e2e     # 运行E2E测试
npm run test:coverage # 测试覆盖率
npm run storybook    # 启动Storybook
```

##  部署到 Vercel

### 方法 1: 通过 Vercel CLI

```bash
npm install -g vercel
vercel
```

### 方法 2: 通过 Git 集成

1. 将代码推送到 GitHub/GitLab/Bitbucket
2. 在 Vercel 中导入项目
3. 配置环境变量 (见下方)
4. 部署

### 环境变量配置

在 Vercel 项目设置中添加以下环境变量:

```
VITE_SUPABASE_URL=your_supabase_url
VITE_SUPABASE_ANON_KEY=your_supabase_anon_key
```

**重要**: 如果不配置 Supabase,系统将以演示模式运行,编辑功能在刷新后会丢失。

##  数据库配置

本项目使用 Supabase 作为后端数据库。完整配置指南请参考:

 **[Supabase 配置指南](./SUPABASE_SETUP.md)**

配置步骤概览:
1. 创建 Supabase 项目
2. 执行 SQL 脚本创建表结构
3. 配置环境变量
4. 部署应用

##  技术栈

- **前端框架**: React 18 + TypeScript
- **构建工具**: Vite
- **UI 组件**: shadcn/ui + Radix UI
- **样式**: TailwindCSS
- **图标**: Lucide React
- **数据库**: Supabase (PostgreSQL)
- **部署**: Vercel

##  项目结构

```
├── src/
│   ├── components/        # React 组件
│   │   ├── ui/           # UI 基础组件
│   │   ├── DeviceDetail.tsx
│   │   ├── EditDeviceDialog.tsx
│   │   ├── HomePage.tsx
│   │   └── ...
│   ├── data/             # 数据模型和初始数据
│   │   └── devices.ts
│   ├── lib/              # 工具库
│   │   ├── supabase.ts   # Supabase 客户端
│   │   └── database.types.ts
│   ├── services/         # 业务逻辑层
│   │   └── deviceService.ts
│   └── App.tsx           # 应用入口
├── SUPABASE_SETUP.md     # Supabase 配置指南
├── .env.example          # 环境变量示例
└── package.json
```

##  开发说明

### 添加新设备

在 Supabase Dashboard 或通过应用界面添加。

### 修改设备信息

点击设备卡片进入详情页,点击"编辑设备"按钮。

### 添加维护记录

在设备详情页点击"添加维护"按钮。

## 🎯 参数化调度系统

参数化调度是一种声明式的设备和资源调度方式，通过配置单一的调度参数（spec），系统自动生成并执行所有相关的动作，联动更新资产位置、库存和绑定关系。

### 核心特性

- **声明式配置**: 只需配置"要达到的目标状态"，无需手动创建每个动作
- **自动差异计算**: 系统自动比对当前状态与目标状态，生成必要的动作
- **事务化执行**: 所有动作在单一事务中执行，确保数据一致性
- **幂等操作**: 重复应用相同参数不会产生重复副作用
- **可回滚**: 支持一键撤销整个调度单的所有影响

### 使用场景

1. **设备外出/投放**: 将设备从仓库调拨到展厅，同时携带耗材和绑定码
2. **设备转场**: 更改设备目的地，系统自动生成新的调拨动作
3. **耗材补充**: 修改耗材数量，系统自动调整领用记录
4. **码绑定调整**: 更换打印机或码，系统自动验证兼容性并更新绑定

### 快速开始

#### 1. 创建调度单

在设备详情页或列表页点击"参数化调度"按钮，打开调度配置抽屉：

```typescript
// 调度参数示例
{
  "destination_location_id": "展厅A",
  "source_location_id": "仓库",
  "items": [
    { "asset_type": "打印机", "asset_id": "printer-001" },
    { "asset_type": "路由器", "asset_id": "router-002" }
  ],
  "consumables": [
    { "consumable_id": "paper-6inch", "qty": 2 },
    { "consumable_id": "ribbon-dnp", "qty": 1 }
  ],
  "codes": [
    { "code_id": "code-专码-001", "bind_to_printer_id": "printer-001" }
  ]
}
```

#### 2. 保存草稿或直接应用

- **保存草稿**: 仅保存配置，不执行动作
- **保存并应用**: 立即执行，生成动作并更新数据库

#### 3. 修改参数

修改任意参数后保存，系统会自动：
- 计算与之前状态的差异
- 生成补差或冲销动作
- 更新资产位置和库存

#### 4. 撤销调度

如需撤销整个调度单的影响：
- 系统按记录的 `dispatch_generated_actions` 逐条反向动作
- 库存和位置回到调度前状态

### 兼容性规则

系统会自动验证以下兼容性规则：

| 打印机品牌 | 允许的码类型 | 说明 |
|-----------|------------|------|
| DNP | 仅专码 | 不支持通码 |
| 诚研 | 专码/通码 | 二选一 |
| 西铁城 | 专码/通码 | 二选一 |

- **专码**: 只能绑定到一台打印机，已绑定则报错
- **通码**: 可多次使用

### 数据库表结构

#### dispatch_orders (调度单表)
```sql
- id: UUID
- spec: JSONB (唯一参数源)
- status: ENUM (draft, applied, reverted)
- created_by, updated_by: TEXT
- effective_at: TIMESTAMPTZ
```

#### dispatch_generated_actions (生成动作关联表)
```sql
- dispatch_id: UUID (外键)
- action_id: UUID (外键)
- fingerprint: TEXT (幂等键)
- operation: TEXT (add/revert)
```

### Edge Function

`apply_dispatch` Edge Function 负责：
1. 读取调度单的 `spec`
2. 查询当前数据库状态
3. 计算差异（diff）
4. 在事务中执行新增/撤销动作
5. 更新 `dispatch_orders.status`

### API 示例

```typescript
// 创建调度单
const id = await createDispatchOrder(spec, 'user-001', '设备外出-展厅A');

// 更新调度单
await updateDispatchOrder(id, updatedSpec, 'user-001');

// 应用调度单
const result = await applyDispatchOrder(id);
// result: { success: true, actions_added: 3, actions_reverted: 0 }

// 撤销调度单
await revertDispatchOrder(id, 'user-001');
```

### 最佳实践

1. **先草稿后应用**: 复杂调度先保存草稿，确认无误后再应用
2. **分批调度**: 大批量设备分多个调度单，便于追踪和回滚
3. **备注完整**: 在 `notes` 字段记录调度原因和目的
4. **定期清理**: 已完成的调度单可定期归档

##  原始设计

This project is based on the Figma design: https://www.figma.com/design/pLarcIzb1aIELsn6Kwg8nw/%E6%8A%80%E6%9C%AF%E6%94%AF%E6%8C%81%E8%AE%BE%E5%A4%87%E7%AE%A1%E7%90%86%E7%BD%91%E7%AB%99

##  License

MIT