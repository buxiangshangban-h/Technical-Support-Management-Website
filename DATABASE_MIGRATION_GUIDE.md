# 🚀 数据库迁移工具使用文档

## 概述

这是一个为技术支持设备管理网站打造的**智能数据库迁移系统**，可以自动扫描、追踪和执行数据库迁移。

### 核心特性 ✨

- ✅ **自动扫描**：自动发现 `supabase/migrations/` 目录下的所有 SQL 文件
- ✅ **追踪记录**：记录已执行的迁移，避免重复执行
- ✅ **事务安全**：每个迁移在事务中执行，失败自动回滚
- ✅ **零配置**：数据库连接已内置，无需额外配置
- ✅ **智能判断**：自动判断哪些迁移需要执行

---

## 快速开始

### 1. 查看迁移状态

```bash
npm run migrate:status
```

**输出示例：**
```
📊 数据库迁移状态
════════════════════════════════════════════════
迁移文件列表:
┌─────┬─────────────────────────────────┬──────────┐
│ 序号│ 文件名                          │ 状态     │
├─────┼─────────────────────────────────┼──────────┤
│    1│ 0001_init.sql                   │ ✅ 已执行│
│    2│ 0002_outbound_inventory_simple.sql│ ✅ 已执行│
│    3│ 0003_devices_table.sql          │ ✅ 已执行│
│    4│ 0004_add_original_fields.sql    │ ⏳ 待执行│
└─────┴─────────────────────────────────┴──────────┘

📈 统计: 总计 4 个，已执行 3 个，待执行 1 个
```

### 2. 执行待处理的迁移

```bash
npm run migrate
```

**输出示例：**
```
🚀 智能数据库迁移工具启动
════════════════════════════════════════════════
📡 连接数据库: postgresql://postgres:****@...
✅ 数据库连接成功

📂 发现 4 个迁移文件
✓  已执行 3 个迁移

⏳ 待执行 1 个迁移:
   1. 0004_add_original_fields.sql

📄 执行迁移: 0004_add_original_fields.sql
━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━
✅ 迁移成功: 0004_add_original_fields.sql

🎉 迁移完成！成功执行 1/1 个迁移
════════════════════════════════════════════════
```

---

## 可用命令

| 命令 | 说明 | 使用场景 |
|------|------|----------|
| `npm run migrate` | 执行所有待处理的迁移 | 部署新版本、同步数据库结构 |
| `npm run migrate:status` | 查看迁移状态 | 检查数据库是否最新 |
| `npm run migrate:reset` | 重置迁移记录表 | **危险操作**，仅用于开发环境 |

---

## 工作流程

### 典型开发流程 🔄

```mermaid
graph LR
    A[修改代码] --> B[创建迁移文件]
    B --> C[本地测试迁移]
    C --> D[提交到 Git]
    D --> E[Push 到 GitHub]
    E --> F[数据库自动更新]
```

### 详细步骤

#### 1. 创建新的迁移文件

在 `supabase/migrations/` 目录下创建新文件：

```bash
# 文件命名规范：XXXX_description.sql
# XXXX 是递增的序号（如 0005、0006）

supabase/migrations/0005_add_new_field.sql
```

**示例迁移文件：**

```sql
-- 添加新字段到 devices 表
ALTER TABLE devices
ADD COLUMN IF NOT EXISTS warranty_date DATE;

-- 添加注释
COMMENT ON COLUMN devices.warranty_date IS '保修截止日期';
```

#### 2. 本地测试

```bash
# 查看待执行的迁移
npm run migrate:status

# 执行迁移
npm run migrate
```

#### 3. 提交和推送

```bash
git add supabase/migrations/0005_add_new_field.sql
git commit -m "feat: 添加设备保修日期字段"
git push
```

#### 4. 自动同步到生产环境

**方式A：手动触发（推荐）**

在服务器上运行：
```bash
npm run migrate
```

**方式B：在 Vercel 部署时自动执行**

在 `package.json` 中已配置：
```json
"postinstall": "npm run migrate"
```

这样每次 Vercel 部署时会自动执行迁移！

---

## 迁移文件编写规范

### ✅ 推荐写法

```sql
-- 描述性注释
-- 说明这个迁移的目的

-- 使用 IF NOT EXISTS 避免重复执行错误
ALTER TABLE table_name
ADD COLUMN IF NOT EXISTS column_name TYPE;

-- 添加索引（如果不存在）
CREATE INDEX IF NOT EXISTS idx_name ON table_name(column);

-- 添加注释
COMMENT ON COLUMN table_name.column_name IS '字段说明';
```

### ❌ 不推荐写法

```sql
-- ❌ 没有 IF NOT EXISTS，重复执行会报错
ALTER TABLE table_name ADD COLUMN column_name TYPE;

-- ❌ 没有事务保护的多步操作
BEGIN;
  ALTER TABLE...
  UPDATE...
COMMIT;  -- 迁移工具已自动处理事务
```

### 迁移文件命名规范

```
0001_init.sql                      # 初始化数据库
0002_add_user_table.sql            # 添加用户表
0003_update_device_schema.sql      # 更新设备表结构
0004_add_original_fields.sql       # 添加原始字段
```

**规则：**
- 使用 4 位数字前缀（0001, 0002, ...）
- 使用下划线分隔
- 使用小写字母
- 描述性命名

---

## 环境变量配置

### 数据库连接

迁移工具会按以下优先级查找数据库连接：

1. 环境变量 `DATABASE_URL`
2. 环境变量 `SUPABASE_DB_URL`
3. 默认值（已内置在脚本中）

**如何设置环境变量：**

#### Windows
```bash
set DATABASE_URL=postgresql://postgres:password@host:5432/database
npm run migrate
```

#### Linux/Mac
```bash
export DATABASE_URL=postgresql://postgres:password@host:5432/database
npm run migrate
```

#### Vercel 环境变量

在 Vercel Dashboard 中设置：
- 变量名：`DATABASE_URL`
- 变量值：`postgresql://...`

---

## 常见问题

### Q1: 迁移执行失败怎么办？

**A:** 迁移在事务中执行，失败会自动回滚。检查错误信息，修复 SQL 后重新执行。

```bash
# 查看详细错误
npm run migrate

# 错误示例：
❌ 迁移失败: column "xxx" already exists
```

**解决方案：**
- 在 SQL 中添加 `IF NOT EXISTS` 子句
- 或使用 `npm run migrate:reset` 重置记录（开发环境）

### Q2: 如何回滚迁移？

**A:** 创建一个新的迁移文件来撤销更改：

```sql
-- 0006_rollback_field.sql
ALTER TABLE devices
DROP COLUMN IF EXISTS warranty_date;
```

### Q3: 迁移记录表在哪里？

**A:** 迁移记录存储在 `schema_migrations` 表中：

```sql
SELECT * FROM schema_migrations ORDER BY name;
```

### Q4: 如何在新环境中初始化数据库？

**A:** 直接运行迁移即可：

```bash
npm install    # 会自动运行 postinstall
# 或手动执行
npm run migrate
```

---

## 高级用法

### 条件迁移

```sql
-- 仅当表不存在时创建
CREATE TABLE IF NOT EXISTS new_table (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL
);

-- 仅当字段不存在时添加
DO $$
BEGIN
  IF NOT EXISTS (
    SELECT FROM information_schema.columns
    WHERE table_name = 'devices' AND column_name = 'new_field'
  ) THEN
    ALTER TABLE devices ADD COLUMN new_field TEXT;
  END IF;
END $$;
```

### 数据迁移

```sql
-- 更新现有数据
UPDATE devices
SET warranty_date = created_at + INTERVAL '1 year'
WHERE warranty_date IS NULL;

-- 批量插入
INSERT INTO settings (key, value)
VALUES
  ('feature_flag_1', 'true'),
  ('feature_flag_2', 'false')
ON CONFLICT (key) DO NOTHING;
```

---

## 工具架构

```
scripts/
├── db-migrate.js           # 智能迁移工具（主程序）
└── migrate-db.js           # 旧版工具（兼容性保留）

supabase/migrations/        # 迁移文件目录
├── 0001_init.sql
├── 0002_outbound_inventory_simple.sql
├── 0003_devices_table.sql
└── 0004_add_original_fields.sql

数据库:
└── schema_migrations       # 迁移记录表
    ├── id                  # 自增ID
    ├── name                # 迁移文件名
    ├── executed_at         # 执行时间
    └── checksum            # 文件校验和
```

---

## 最佳实践 💡

1. **总是使用 `IF NOT EXISTS`**：避免重复执行错误
2. **小步迭代**：一个迁移文件只做一件事
3. **先测试后部署**：本地测试通过再推送
4. **记录变更原因**：在 SQL 文件中添加详细注释
5. **备份重要数据**：执行破坏性操作前先备份

---

## 故障排除

### 连接问题

```bash
# 测试数据库连接
npm run test:db

# 或直接测试迁移连接
npm run migrate:status
```

### 权限问题

确保数据库用户有以下权限：
- CREATE TABLE
- ALTER TABLE
- CREATE INDEX
- SELECT, INSERT, UPDATE

### 文件编码问题

确保迁移文件使用 UTF-8 编码。

---

## 支持

遇到问题？

1. 查看迁移状态：`npm run migrate:status`
2. 检查数据库日志
3. 联系开发团队

---

**文档版本：** 1.0
**最后更新：** 2025-10-14
**维护者：** 浮浮酱 🐱
