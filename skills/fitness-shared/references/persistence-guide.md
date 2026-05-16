# 持久化策略指南

健身 skills 使用两种持久化方案，按优先级排列。

## 主方案: shared_memory MCP 工具

使用 `shared_memory_write` 和 `shared_memory_read` MCP 工具，namespace 为 `fitness`。

### 读取操作
```
mcp__shared_memory_read(key="user_profile", namespace="fitness")
```

### 写入操作
```
mcp__shared_memory_write(key="user_profile", namespace="fitness", value="<JSON字符串>")
```

### TTL 设置
为避免数据意外过期，所有 fitness 数据的 TTL 应设置为较长时间：

| Key | TTL (秒) | 说明 |
|-----|----------|------|
| `user_profile` | 7776000 (90天) | 用户基础数据，变更不频繁 |
| `current_plan` | 2592000 (30天) | 当前计划，可能被更新替换 |
| `nutrition_plan` | 2592000 (30天) | 饮食方案 |
| `progress_log` | 31536000 (365天) | 进度日志，需长期保留 |
| `plan_adjustments` | 604800 (7天) | 临时调整建议，处理后清除 |

## 备选方案: 文件存储

当 `shared_memory` MCP 工具不可用时，使用 JSON 文件存储。

**文件路径:** `~/.claude/fitness-data.json`

**文件结构:**
```json
{
  "user_profile": { ... },
  "current_plan": { ... },
  "nutrition_plan": { ... },
  "progress_log": [ ... ],
  "plan_adjustments": { ... }
}
```

**读写方法:**
- 读取：使用 Read 工具读取 `~/.claude/fitness-data.json`
- 写入：使用 Write 工具覆盖写入整个文件
- 注意：文件模式下需要先读取 → 修改 → 完整覆盖写入

## 方案切换逻辑

```
1. 尝试 shared_memory_read(namespace="fitness", key="<key>")
2. 如果返回 null 或工具不可用 → 尝试读取 ~/.claude/fitness-data.json
3. 如果文件也不存在 → 这是新用户，开始采集流程
4. 写入时：优先使用 shared_memory_write
5. 如果 shared_memory_write 失败 → 降级到文件写入，并提示用户
```

## 隐私声明

所有健身数据存储在本地（`shared_memory` namespace 或 `~/.claude/fitness-data.json`），不会上传到外部服务器。用户可以随时删除这些数据。

## 数据清理

用户可通过以下方式清理数据：
- 删除 `~/.claude/fitness-data.json` 文件
- 使用 `shared_memory_delete(key="...", namespace="fitness")` 删除特定 key
- 说「清除我的所有健身数据」触发清理流程
