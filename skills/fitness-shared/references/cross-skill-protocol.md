# 跨 Skill 通信协议

健身 skills 之间通过 `~/.claude/.claude/fitness-data/` 目录下的 JSON 文件共享数据。

## 共享文件

| 文件 | 写入者 | 读取者 | 内容 |
|------|--------|--------|------|
| `.claude/fitness-data/user-profile.json` | workout-plan (首次) | 所有 skills | 用户身体数据 + 目标 (JSON) |
| `.claude/fitness-data/current-plan.json` | workout-plan | nutrition, progress | 当前训练计划 (JSON) |
| `.claude/fitness-data/nutrition-plan.json` | nutrition | workout-plan, progress | 当前饮食方案 (JSON) |
| `.claude/fitness-data/progress-log.json` | progress | workout-plan, nutrition | 进度记录数组 [{date, weight, ...}] |
| `.claude/fitness-data/plan-adjustments.json` | progress | workout-plan | 计划调整建议 (JSON) |

## 写入规范

### 单写者原则
- 每个文件只有一个主要写入者
- 如果 workout-plan 需要更新 `user-profile.json` 中的 `weight_kg`（用户告知体重变化），也可以写入，但需提示用户

### 数据格式

**.claude/fitness-data/user-profile.json:**
```json
{
  "height_cm": 175,
  "weight_kg": 80,
  "goal": "增肌",
  "equipment": ["哑铃", "杠铃"],
  "days_per_week": 4,
  "experience_level": "中级",
  "dietary_restrictions": [],
  "age": 30,
  "gender": "male"
}
```

**.claude/fitness-data/current-plan.json:**
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

**.claude/fitness-data/progress-log.json:**
```json
[
  {"date": "2026-05-14", "weight_kg": 79.5, "notes": "感觉力量有进步"}
]
```

**.claude/fitness-data/plan-adjustments.json:**
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
1. Progress 分析 `progress-log.json` 和 `current-plan.json`
2. 写入 `plan-adjustments.json`
3. 在响应中提示用户：「检测到可能需要调整训练计划。你可以跟我说"更新训练计划"，我会根据最新进展为你生成更新后的方案。」
4. 用户触发 workout-plan skill
5. Workout-plan 读取 `plan-adjustments.json`，生成调整后的计划
6. Workout-plan 将 `plan-adjustments.json` 覆盖为空对象 `{}`

## 一致性规则
- 同一数据避免同时写入冲突（单写者原则已预防）
- 关键修改应提示用户确认

## 数据生命周期
- 体重/围度日志：保留 12 个月
- 训练计划：保留最近 4 周
- 饮食方案：保留最近 4 周
- 调整建议：处理后立即清除
