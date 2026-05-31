# 持久化策略指南

本健身 Agent 使用 `.claude/fitness-data/` 目录存储所有用户数据。所有文件均为 JSON 格式，用 Read/Write 工具进行读写。

## 存储位置

```
.claude/fitness-data/
  ├── user-profile.json        # 用户身体数据 + 目标
  ├── current-plan.json        # 当前训练计划
  ├── nutrition-plan.json      # 饮食方案
  ├── progress-log.json        # 进度记录数组
  └── plan-adjustments.json    # 计划调整建议（临时）
```

## 读写方法

### 读取数据
用 Read 工具读取对应文件。
- 文件存在 → 解析 JSON，使用数据
- 文件不存在 → 这是新用户或该数据尚未创建，开始采集流程

### 写入/覆盖数据
用 Write 工具写入文件。JSON 内容格式化（缩进 2 空格）。

### 追加数据（progress-log）
progress-log.json 是 JSON 数组，追加新记录时：
1. 用 Read 读取 `.claude/fitness-data/progress-log.json`
2. 解析为数组
3. push 新条目
4. 用 Write 写回

## 数据生命周期

| 文件 | 保留策略 | 说明 |
|------|---------|------|
| `user-profile.json` | 长期保留 | 用户基础数据，变更不频繁 |
| `current-plan.json` | 覆盖更新 | 每次生成新计划直接覆盖 |
| `nutrition-plan.json` | 覆盖更新 | 每次计算新方案直接覆盖 |
| `progress-log.json` | 保留 12 个月 | 长期追踪需要，定期清理旧记录 |
| `plan-adjustments.json` | 处理后清除 | 临时调整建议，被 workout-plan 读取后覆盖为空对象 `{}` |

## 隐私声明

所有健身数据存储在本地 `.claude/fitness-data/` 目录下，不会上传到外部服务器。用户可以随时删除此目录或其中的文件来清理数据。

## 数据清理

用户可通过以下方式清理数据：
- 删除 `.claude/fitness-data/` 目录下对应文件
- 说「清除我的所有健身数据」→ Agent 删除 `.claude/fitness-data/` 下所有 JSON 文件
- 说「删除我的训练记录」→ Agent 将 `progress-log.json` 写为空数组 `[]`
