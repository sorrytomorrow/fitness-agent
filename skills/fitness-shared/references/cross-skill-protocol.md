# 跨 Skill 通信协议

健身 skills 之间通过 `shared_memory` MCP 工具的 `fitness` namespace 共享数据。

## 共享 Keys

| Key | 写入者 | 读取者 | 内容 |
|-----|--------|--------|------|
| `user_profile` | workout-plan (首次) | 所有 skills | 用户身体数据 + 目标 (JSON) |
| `current_plan` | workout-plan | nutrition, progress | 当前训练计划 (JSON) |
| `nutrition_plan` | nutrition | workout-plan, progress | 当前饮食方案 (JSON) |
| `progress_log` | progress | workout-plan, nutrition | 进度记录数组 [{date, weight, ...}] |
| `plan_adjustments` | progress | workout-plan | 计划调整建议 (JSON) |

## 写入规范

### 单写者原则
- 每个 key 只有一个主要写入者
- 如果 workout-plan 需要更新 `user_profile` 中的 `weight_kg`（用户告知体重变化），也可以写入，但需提示用户

### 数据格式

**current_plan:**
```json
{
  "created_at": "YYYY-MM-DD",
  "goal": "减脂",
  "weeks": 4,
  "days": [
    {
      "day": 1,
      "focus": "胸部+三头",
      "exercises": [
        {"name": "杠铃卧推", "sets": 4, "reps": "8-12", "rest_sec": 90},
        {"name": "上斜哑铃卧推", "sets": 3, "reps": "10-12", "rest_sec": 60}
      ],
      "warmup": "5分钟跳绳 + 动态拉伸",
      "cooldown": "胸部拉伸 + 三头拉伸"
    }
  ]
}
```

**progress_log:**
```json
[
  {"date": "2026-05-14", "weight_kg": 79.5, "notes": "感觉力量有进步"}
]
```

**plan_adjustments:**
```json
{
  "detected_at": "2026-05-21",
  "reason": "连续2周体重无变化，可能进入平台期",
  "suggestions": [
    "增加有氧训练至每周3次",
    "将卧推组数从4组增加到5组",
    "考虑碳水循环策略"
  ],
  "severity": "moderate"
}
```

## 委托模式

当 progress skill 检测到需要调整计划时：
1. Progress 分析 `progress_log` 和 `current_plan`
2. 写入 `plan_adjustments` key
3. 在响应中提示用户：「检测到可能需要调整训练计划。你可以跟我说"更新训练计划"，我会根据最新进展为你生成更新后的方案。」
4. 用户触发 workout-plan skill
5. Workout-plan 读取 `plan_adjustments`，生成调整后的计划
6. Workout-plan 清除或标记 `plan_adjustments` 为已处理

## 一致性规则
- 同一数据避免同时写入冲突（单写者原则已预防）
- 如果出现冲突（用户同时在两个 skill 中修改数据），优先采用最新的时间戳
- 关键修改应提示用户确认

## 数据生命周期
- 体重/围度日志：保留 12 个月
- 训练计划：保留最近 4 周
- 饮食方案：保留最近 4 周
- 调整建议：处理后立即清除
