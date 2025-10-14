# 数据流向图解

本文档用可视化方式展示项目的数据流向，帮助理解本地调用如何使用 Supabase 数据。

---

## 📊 整体架构图

```
┌─────────────────────────────────────────────────────────────────┐
│                         用户界面层                                │
│                    (React Components)                           │
│                                                                 │
│  DeviceList.tsx  │  DeviceDetail.tsx  │  Inventory.tsx  │ ...  │
└────────────┬────────────────────────────────────────────────────┘
             │
             │ 调用数据接口
             ↓
┌─────────────────────────────────────────────────────────────────┐
│                         数据层                                   │
│                    (src/data/*.ts)                              │
│                                                                 │
│  devices.ts  │  inventory.ts  │  其他数据模块                    │
│                                                                 │
│  • getDevices()      • getInventory()                          │
│  • updateDevice()    • updateInventory()                       │
│  • getDevice()       • checkStockLevel()                       │
└────────────┬────────────────────────────────────────────────────┘
             │
             │ 检查配置
             ↓
┌─────────────────────────────────────────────────────────────────┐
│                      配置检查层                                  │
│                  (src/lib/supabase.ts)                          │
│                                                                 │
│              isSupabaseConfigured ?                             │
│                     ↙        ↘                                  │
│                 是 ✅         否 ❌                               │
└─────────────────┬──────────────┬──────────────────────────────┘
                  │              │
      使用 Supabase │              │ 使用本地数据
                  ↓              ↓
┌──────────────────────┐  ┌──────────────────────┐
│    服务层             │  │   本地缓存            │
│ (src/services/*.ts)  │  │ (内存中的数据)         │
│                      │  │                      │
│ • deviceService.ts   │  │ • devicesData[]      │
│ • inventoryService.ts│  │ • inventoryData{}    │
│ • outboundService.ts │  │ • localRecords[]     │
└──────────┬───────────┘  └──────────────────────┘
           │
           │ Supabase 客户端调用
           ↓
┌──────────────────────────────────────────────────────────────┐
│                    Supabase 客户端                             │
│                 (src/lib/supabase.ts)                         │
│                                                               │
│  const supabase = createClient(url, key)                     │
└──────────┬────────────────────────────────────────────────────┘
           │
           │ HTTP/WebSocket
           ↓
┌──────────────────────────────────────────────────────────────┐
│                    Supabase 云数据库                           │
│          (https://sbp-a2e2xuudcasoe44t.supabase.co)          │
│                                                               │
│  • devices 表                                                 │
│  • maintenance_logs 表                                        │
│  • issues 表                                                  │
│  • inventory 表                                               │
│  • outbound_records 表                                        │
│  • audit_logs 表                                              │
└───────────────────────────────────────────────────────────────┘
```

---

## 🔄 数据读取流程（以设备列表为例）

### 场景：用户访问设备列表页面

```
1️⃣ 用户操作
   用户打开设备列表页面
   ↓

2️⃣ 组件渲染
   DeviceList.tsx 组件加载
   useEffect(() => {
     loadDevices();
   }, []);
   ↓

3️⃣ 调用数据层
   const devices = await getDevices();
   ↓
   📄 文件: src/data/devices.ts
   ↓

4️⃣ 配置检查
   if (checkSupabaseConfig()) {  // ← 检查是否配置了 Supabase
     ✅ 已配置 → 继续第 5 步
     ❌ 未配置 → 跳到第 8 步
   }
   ↓

5️⃣ 调用服务层
   const devices = await fetchDevices();
   ↓
   📄 文件: src/services/deviceService.ts
   ↓

6️⃣ Supabase 查询
   const { data: devices, error } = await supabase
     .from('devices')           // ← 查询 devices 表
     .select('*')               // ← 选择所有字段
     .order('name');            // ← 按名称排序
   ↓

7️⃣ 关联数据查询
   // 同时查询维护日志
   const { data: logs } = await supabase
     .from('maintenance_logs')
     .select('*')
     .order('date', { ascending: false });
   
   // 同时查询故障记录
   const { data: issues } = await supabase
     .from('issues')
     .select('*')
     .order('date', { ascending: false });
   ↓

8️⃣ 数据映射和组合
   return devices.map(device => {
     const deviceLogs = logs?.filter(log => log.device_id === device.id);
     const deviceIssues = issues?.filter(issue => issue.device_id === device.id);
     return mapRowToDevice(device, deviceLogs, deviceIssues);
   });
   ↓

9️⃣ 更新缓存
   devicesData = devices;  // ← 更新本地缓存
   ↓

🔟 返回数据
   return devices;  // ← 返回 Supabase 数据
   ↓

1️⃣1️⃣ 组件更新
   setDevices(devices);
   ↓

1️⃣2️⃣ 界面渲染
   显示设备列表
```

### 降级场景：Supabase 未配置

```
4️⃣ 配置检查
   if (checkSupabaseConfig()) {
     ❌ 未配置 → 执行降级逻辑
   }
   ↓

8️⃣ 使用本地数据
   return [...devicesData];  // ← 返回内存中的示例数据
   ↓

9️⃣ 组件更新
   setDevices(devices);  // ← 使用本地数据
   ↓

🔟 界面渲染
   显示设备列表（本地数据）
   ⚠️ 控制台提示: "Supabase 未配置：正在使用本地模拟数据模式"
```

---

## ✏️ 数据写入流程（以更新设备为例）

### 场景：用户修改设备信息

```
1️⃣ 用户操作
   用户在设备详情页修改设备信息
   点击"保存"按钮
   ↓

2️⃣ 组件处理
   const handleSave = async () => {
     const success = await updateDevice(deviceId, updates);
     if (success) {
       toast.success('保存成功');
     }
   };
   ↓

3️⃣ 调用数据层
   await updateDevice(deviceId, updates);
   ↓
   📄 文件: src/data/devices.ts
   ↓

4️⃣ 配置检查
   if (checkSupabaseConfig()) {
     ✅ 已配置 → 继续第 5 步
     ❌ 未配置 → 跳到第 9 步（本地更新）
   }
   ↓

5️⃣ 调用服务层
   const success = await updateDeviceData(deviceId, updates);
   ↓
   📄 文件: src/services/deviceService.ts
   ↓

6️⃣ 数据转换
   const row = mapDeviceToRow(updates);
   // 将前端格式转换为数据库格式
   // 例如: printerModel → printer_model_field
   ↓

7️⃣ Supabase 更新
   const { error } = await supabase
     .from('devices')           // ← 更新 devices 表
     .update(row)               // ← 更新数据
     .eq('id', deviceId);       // ← 条件：设备 ID
   ↓

8️⃣ 返回结果
   if (error) {
     console.error('更新失败:', error);
     return false;
   }
   return true;  // ← 更新成功
   ↓

9️⃣ 更新本地缓存
   const deviceIndex = devicesData.findIndex(d => d.id === deviceId);
   devicesData[deviceIndex] = {
     ...devicesData[deviceIndex],
     ...updates
   };
   ↓

🔟 组件反馈
   toast.success('保存成功');
   // 刷新页面数据保持（因为已写入 Supabase）
```

---

## 🔄 出库流程（复杂事务示例）

### 场景：用户创建出库记录

```
1️⃣ 用户填写出库表单
   • 选择设备
   • 填写目的地
   • 选择出库物资
   • 填写数量
   ↓

2️⃣ 提交出库请求
   const result = await createOutboundRecord({
     deviceId: 'dev-01',
     deviceName: '魔镜1号',
     destination: '上海展厅',
     operator: '张三',
     items: {
       printerModel: 'EPSON-L8058',
       paperType: 'A4',
       paperQuantity: 100,
       inkC: 2,
       routers: 1
     }
   });
   ↓

3️⃣ 服务层处理
   📄 文件: src/services/outboundService.ts
   ↓

4️⃣ 步骤 1: 获取设备信息
   const device = await getDevice(deviceId);
   // 记录原始位置和负责人（用于归还时恢复）
   const originalLocation = device.location;  // "杭州展厅A区"
   const originalOwner = device.owner;        // "李四"
   ↓

5️⃣ 步骤 2: 检查库存
   const stockCheck = await checkStock(items);
   if (!stockCheck.sufficient) {
     return { success: false, error: '库存不足' };
   }
   ↓

6️⃣ 步骤 3: 创建出库记录
   if (isSupabaseConfigured) {
     const { data, error } = await supabase
       .from('outbound_records')
       .insert({
         device_id: 'dev-01',
         device_name: '魔镜1号',
         destination: '上海展厅',
         operator: '张三',
         items: {...},
         status: 'outbound',
         original_location: '杭州展厅A区',  // ← 记录原始位置
         original_owner: '李四'              // ← 记录原负责人
       })
       .select()
       .single();
   }
   ↓

7️⃣ 步骤 4: 创建审计日志
   await supabase.from('audit_logs').insert({
     action_type: '出库',
     entity_type: 'outbound_record',
     entity_id: data.id,
     operator: '张三',
     details: {
       deviceId: 'dev-01',
       destination: '上海展厅',
       items: {...}
     }
   });
   ↓

8️⃣ 步骤 5: 更新设备位置和负责人
   await updateDevice('dev-01', {
     location: '上海展厅',     // ← 更新位置
     owner: '张三'             // ← 更新负责人
   });
   
   // 这会触发另一个 Supabase 更新：
   await supabase
     .from('devices')
     .update({
       location: '上海展厅',
       owner: '张三',
       updated_at: new Date().toISOString()
     })
     .eq('id', 'dev-01');
   ↓

9️⃣ 步骤 6: 更新库存
   await updateInventoryStock(items, 'decrement');
   
   // 这会更新库存表：
   await supabase
     .from('inventory')
     .update({
       paper_stock: {
         'EPSON-L8058': {
           'A4': 450 - 100  // ← 扣减 100 张
         }
       },
       epson_ink_stock: {
         C: 8 - 2  // ← 扣减 2 瓶
       },
       equipment_stock: {
         routers: 15 - 1  // ← 扣减 1 个
       }
     })
     .eq('id', inventoryId);
   ↓

🔟 返回成功
   return { success: true };
   ↓

1️⃣1️⃣ 界面反馈
   toast.success('出库成功');
   
   // 数据变化汇总：
   ✅ outbound_records 表：新增 1 条记录
   ✅ audit_logs 表：新增 1 条审计日志
   ✅ devices 表：更新设备位置和负责人
   ✅ inventory 表：扣减库存数量
```

---

## 🔍 数据同步验证

### 如何验证本地调用使用 Supabase 数据？

#### 方法 1: 控制台日志验证

```javascript
// src/data/devices.ts
export const getDevices = async (): Promise<Device[]> => {
  if (checkSupabaseConfig()) {
    console.log('🔵 正在从 Supabase 获取设备数据...');
    const devices = await fetchDevices();
    if (devices.length > 0) {
      console.log('✅ 成功从 Supabase 获取', devices.length, '台设备');
      devicesData = devices;
      return devices;
    }
  }
  
  console.log('⚠️ 使用本地模拟数据');
  return [...devicesData];
};
```

**预期输出（已配置）：**
```
🔵 正在从 Supabase 获取设备数据...
✅ 成功从 Supabase 获取 10 台设备
```

**预期输出（未配置）：**
```
⚠️ Supabase 未配置：正在使用本地模拟数据模式
⚠️ 使用本地模拟数据
```

#### 方法 2: 数据修改验证

**步骤：**
1. 在本地修改设备信息（例如：修改设备名称）
2. 刷新页面
3. 观察数据是否保持

**结果判断：**
- ✅ **数据保持** → 使用 Supabase（数据已持久化）
- ❌ **数据丢失** → 使用本地内存（刷新后重置）

#### 方法 3: Supabase 控制台验证

**步骤：**
1. 访问 Supabase 控制台：https://supabase.opentrust.net
2. 进入项目：sbp-a2e2xuudcasoe44t
3. 打开 Table Editor
4. 查看 `devices` 表
5. 在本地修改设备信息
6. 刷新 Supabase 控制台
7. 观察数据是否变化

**结果判断：**
- ✅ **数据变化** → 本地调用使用 Supabase
- ❌ **数据不变** → 本地调用使用内存数据

#### 方法 4: 网络请求验证

**步骤：**
1. 打开浏览器开发者工具（F12）
2. 切换到 Network 标签
3. 刷新页面
4. 查找请求到 `supabase.co` 的 API 调用

**预期请求：**
```
GET https://sbp-a2e2xuudcasoe44t.supabase.opentrust.net/rest/v1/devices
POST https://sbp-a2e2xuudcasoe44t.supabase.opentrust.net/rest/v1/devices
```

**结果判断：**
- ✅ **有请求** → 使用 Supabase
- ❌ **无请求** → 使用本地数据

#### 方法 5: 运行测试脚本

```bash
# 运行 Supabase 同步测试
npm run test:sync
```

**预期输出：**
```
🧪 Supabase 数据同步测试

==========================================================
测试 1: 检查配置文件
==========================================================

✅ 环境变量配置文件 存在: .env
✅ 环境变量模板文件 存在: .env.example

==========================================================
测试 2: 检查环境变量
==========================================================

✅ Supabase 项目 URL 已配置: VITE_SUPABASE_URL
  值: https://sbp-a2e2xuudcasoe44t...
✅ Supabase 匿名密钥 已配置: VITE_SUPABASE_ANON_KEY
  值: eyJ0eXAiOiJKV1QiLCJhbGciOiJI...

==========================================================
测试 3: 测试数据库连接
==========================================================

ℹ️  正在连接到 Supabase...
✅ 数据库连接成功！

==========================================================
测试结果汇总
==========================================================

✅ 通过 配置文件检查 [关键]
✅ 通过 环境变量检查 [关键]
✅ 通过 数据库连接 [关键]
✅ 通过 数据表结构 [关键]
✅ 通过 数据读取 [关键]

🎉 所有测试通过！
✨ 项目已正确配置，本地调用使用 Supabase 数据
```

---

## 📝 总结

### 关键要点

1. **✅ 本地调用确实使用 Supabase 数据**
   - 所有读取操作优先查询 Supabase
   - 所有写入操作直接写入 Supabase
   - 数据变化实时同步到云端

2. **✅ 完善的降级机制**
   - 未配置时自动使用本地数据
   - 不会崩溃或报错
   - 用户体验平滑

3. **✅ 清晰的数据流向**
   - 组件 → 数据层 → 配置检查 → 服务层 → Supabase
   - 每一层职责明确
   - 易于维护和扩展

4. **⚠️ 需要完成配置**
   - 创建 `.env` 文件
   - 运行数据库迁移
   - 验证连接成功

### 快速验证命令

```bash
# 1. 配置环境
npm run setup

# 2. 测试连接
npm run test:sync

# 3. 启动开发
npm run dev

# 4. 打开浏览器
# http://localhost:5173
# 按 F12 查看控制台和网络请求
```

---

**文档版本：** v1.0  
**更新时间：** 2025-10-14
