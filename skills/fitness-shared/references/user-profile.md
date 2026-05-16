# 健身用户数据模型

所有健身 skills 共享的用户数据模型。每个 skill 在处理用户请求前应先尝试从 `shared_memory` 读取现有用户数据。

## JSON Schema

```json
{
  "height_cm": {"type": "number", "range": [100, 250], "required": true},
  "weight_kg": {"type": "number", "range": [30, 300], "required": true},
  "age": {"type": "number", "range": [10, 100], "required": false},
  "gender": {"type": "string", "enum": ["male", "female", "other"], "required": false},
  "goal": {"type": "string", "enum": ["减脂", "增肌", "塑形", "力量提升", "体能提升", "维持健康"], "required": true},
  "equipment": {"type": "string[]", "examples": ["哑铃", "杠铃", "器械", "自重", "弹力带", "壶铃"], "required": true},
  "days_per_week": {"type": "number", "range": [1, 7], "required": true},
  "dietary_restrictions": {"type": "string[]", "examples": ["素食", "乳糖不耐受", "坚果过敏"], "required": false},
  "experience_level": {"type": "string", "enum": ["零基础", "初学者", "中级", "高级"], "required": false}
}
```

## 字段说明

| 字段 | 说明 | 示例 |
|------|------|------|
| `height_cm` | 身高（厘米） | 175 |
| `weight_kg` | 体重（公斤） | 80 |
| `age` | 年龄 | 30 |
| `gender` | 性别 | "male" |
| `goal` | 健身目标 | "减脂" |
| `equipment` | 可用器械列表 | ["哑铃", "杠铃"] |
| `days_per_week` | 每周可训练天数 | 4 |
| `dietary_restrictions` | 饮食限制/偏好 | ["素食"] |
| `experience_level` | 训练经验 | "中级" |

## 输入校验规则

### 数值范围校验
- **身高**: 100-250cm。超出范围 → 提示用户复查，说明合理范围
- **体重**: 30-300kg。超出范围 → 提示用户复查
- **年龄**: 10-100岁。超出范围 → 提示用户复查
- **每周天数**: 1-7天。>7 → 自动裁剪为7

### 必填字段缺失处理
1. 先从 `shared_memory_read(key="user_profile", namespace="fitness")` 读取已有数据
2. 缺失字段逐个询问用户，每次最多 2 个字段
3. 不静默使用默认值——必须用户确认
4. 全部采集完毕后，通过 `shared_memory_write` 保存

### 健身目标与建议值的对应关系
| 目标 | 建议训练频率 | 建议热量调整 |
|------|-------------|-------------|
| 减脂 | 3-5天/周 | -300~500 kcal/天 |
| 增肌 | 4-6天/周 | +300~500 kcal/天 |
| 塑形 | 3-4天/周 | 维持热量平衡 |
| 力量提升 | 3-4天/周 | +100~300 kcal/天 |
| 体能提升 | 3-5天/周 | 维持或轻微盈余 |
| 维持健康 | 2-3天/周 | 维持热量平衡 |

## 存储位置

`shared_memory` namespace: `fitness`, key: `user_profile`
