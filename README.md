# 嵌入式人自己的健身教练 AI Agent

你的专属 AI 健身私人教练，基于 Claude Code + oh-my-claudecode 构建的多 Agent 协作系统。提供训练计划制定、运动营养指导、进度追踪调整和动作技术教学。

## 功能模块

| 模块 | 说明 | 触发场景 |
|------|------|---------|
| 🏋️ **训练计划** | 根据身体数据、目标、器械生成个性化周训练计划 | 减脂、增肌、塑形、安排训练 |
| 🥗 **饮食营养** | 基于 TDEE 计算每日热量和营养素配比 | 饮食、营养、热量、蛋白质 |
| 📊 **进度追踪** | 记录体重/围度/训练数据，分析趋势，检测平台期 | 记录进展、汇报训练、体重变化 |
| 🎯 **动作指导** | 覆盖 8 大肌群 30+ 动作的标准要领和纠错 | 深蹲/硬拉/卧推等动作问答 |

## 项目结构

```
健身Agent/
├── agents/
│   └── fitness-coach.md          # 主 Agent 定义（路由 + 编排）
├── skills/
│   ├── fitness-shared/
│   │   └── references/
│   │       ├── user-profile.md       # 用户数据模型
│   │       ├── cross-skill-protocol.md # Skill 间通信协议
│   │       └── persistence-guide.md  # 数据持久化策略
│   ├── fitness-workout-plan/
│   │   └── SKILL.md              # 训练计划生成
│   ├── fitness-nutrition/
│   │   └── SKILL.md              # 饮食营养建议
│   ├── fitness-progress/
│   │   └── SKILL.md              # 进度追踪与调整
│   └── fitness-exercise-guide/
│       └── SKILL.md              # 动作知识问答
└── README.md
```

## 安装

### 前提条件

- 已安装 [Claude Code](https://claude.ai/code)
- 已安装 [oh-my-claudecode](https://github.com/anthropics/claude-code) 多 Agent 编排层

### 安装步骤

1. 克隆仓库：
```bash
git clone https://github.com/YOUR_USERNAME/fitness-agent.git
```

2. 复制 Agent 到 Claude Code 配置目录：
```bash
cp -r agents/fitness-coach.md ~/.claude/agents/
```

3. 复制 Skills 到 Claude Code 配置目录：
```bash
cp -r skills/fitness-* ~/.claude/skills/
```

4. 重启 Claude Code，Agent 自动生效。在对话中说「健身教练」即可激活。

## 使用方式

在 Claude Code 中直接对话即可：

- 「帮我设计一个减脂训练计划」→ 自动路由到训练计划模块
- 「我今天吃了什么，热量够不够」→ 自动路由到营养模块
- 「记录一下今天体重 78kg」→ 自动路由到进度追踪模块
- 「深蹲的正确姿势是什么样的」→ 自动路由到动作指导模块

## 工作原理

四个 Skill 通过 `shared_memory` 共享用户数据（`fitness` namespace），实现跨模块协作：

- 训练计划生成用户画像 → 营养模块据此计算热量
- 进度追踪检测平台期 → 自动写入调整建议 → 训练计划下次生成时融入
- 动作指导结合用户经验水平 → 给出针对性建议

## 路线图

- [x] 训练计划生成（5种分化方式）
- [x] 饮食营养计算（Mifflin-St Jeor 公式）
- [x] 进度追踪与平台期检测
- [x] 30+ 动作标准指导
- [ ] 训练视频链接集成
- [ ] Nutritionix API 食物数据集成
- [ ] 可视化进度图表

## 免责声明

本 Agent 提供的所有建议仅供参考，不构成医疗建议。在开始任何新的训练或饮食计划前，请咨询医生或专业教练。如训练中出现疼痛或不适，请立即停止。
