# 嵌入式人自己的健身教练 AI Agent

你的专属 AI 健身私人教练，基于 **Claude Code 自定义 Agent** 构建。提供训练计划制定、运动营养指导、进度追踪调整和动作技术教学。

> **零额外依赖** — 不需要 OMC，不需要 MCP Server，只要装 Claude Code 就能用。

## 功能模块

| 模块 | 说明 | 触发场景 |
|------|------|---------|
| 训练计划 | 根据身体数据、目标、器械生成个性化周训练计划 | 减脂、增肌、塑形、安排训练 |
| 饮食营养 | 基于 TDEE 计算每日热量和营养素配比 | 饮食、营养、热量、蛋白质 |
| 进度追踪 | 记录体重/围度/训练数据，分析趋势，检测平台期 | 记录进展、汇报训练、体重变化 |
| 动作指导 | 覆盖 8 大肌群 30+ 动作的标准要领和纠错 | 深蹲/硬拉/卧推等动作问答 |

## 项目结构

```
健身Agent/
├── agents/
│   └── fitness-coach.md              # Agent 定义 → 安装到 ~/.claude/agents/
├── skills/
│   ├── fitness-workout-plan/SKILL.md   # 训练计划生成 → 安装到 ~/.claude/skills/
│   ├── fitness-nutrition/SKILL.md      # 饮食营养建议
│   ├── fitness-progress/SKILL.md       # 进度追踪与调整
│   ├── fitness-exercise-guide/SKILL.md # 动作知识问答
│   └── fitness-shared/references/      # 共享协议和模型
└── README.md
```

## 安装（2 步）

### 前提条件

- 已安装 [Claude Code](https://claude.ai/code)

### 获取代码

```bash
git clone https://github.com/sorrytomorrow/fitness-agent.git
```

或手动下载项目文件夹。

### 步骤 1：复制 Agent

```bash
# macOS / Linux
cp agents/fitness-coach.md ~/.claude/agents/

# Windows (PowerShell)
copy agents\fitness-coach.md %USERPROFILE%\.claude\agents\
```

### 步骤 2：复制 Skills

```bash
# macOS / Linux
cp -r skills/fitness-* ~/.claude/skills/

# Windows (PowerShell)
xcopy skills\fitness-* %USERPROFILE%\.claude\skills\ /E /I
```

用户数据自动创建在当前工作目录的 `.claude/fitness-data/` 下。在哪个文件夹启动 Claude Code，数据就存在哪里。

## 使用方式

安装后重启 Claude Code，直接对话即可：

- 「帮我设计一个减脂训练计划」→ 训练计划模块
- 「我今天吃了什么，热量够不够」→ 营养模块
- 「记录一下今天体重 78kg」→ 进度追踪模块
- 「深蹲的正确姿势是什么样的」→ 动作指导模块

## 工作原理

```
用户说话 → Agent 识别意图 → 读取 ~/.claude/skills/fitness-*/SKILL.md → 按指令执行
                                                                          │
                                              读写 ./.claude/fitness-data/*.json
```

四个 Skill 通过 `.claude/fitness-data/` 下的 JSON 文件共享用户数据：
- 训练计划 → `fitness-data/user-profile.json` + `current-plan.json`
- 营养模块 → 读取用户画像 → `fitness-data/nutrition-plan.json`
- 进度追踪 → `fitness-data/progress-log.json` + `plan-adjustments.json`

## 路线图

- [x] 训练计划生成（5种分化方式）
- [x] 饮食营养计算（Mifflin-St Jeor 公式）
- [x] 进度追踪与平台期检测
- [x] 30+ 动作标准指导
- [x] 零额外依赖（纯 Claude Code 原生）
- [ ] 训练视频链接集成
- [ ] Nutritionix API 食物数据集成
- [ ] 可视化进度图表

## 免责声明

本 Agent 提供的所有建议仅供参考，不构成医疗建议。在开始任何新的训练或饮食计划前，请咨询医生或专业教练。如训练中出现疼痛或不适，请立即停止。
