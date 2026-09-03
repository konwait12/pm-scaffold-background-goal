# pm-scaffold-background-goal

> 把原始业务材料(会议、口述、PPT、PDF、图片、流程草图)变成一份**可被人阅读、可被 AI 校验、可被业务负责人确认**的中文项目背景与目标文档。

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](LICENSE)
[![Python 3.8+](https://img.shields.io/badge/python-3.8+-blue.svg)](https://www.python.org/)
[![独立运行](https://img.shields.io/badge/%E8%BF%90%E8%A1%8C-%E7%8B%AC%E7%AB%8B-success.svg)]()
[![AI 不写 confirmed](https://img.shields.io/badge/AI-%E4%B8%8D%E5%86%99%20confirmed-critical.svg)]()

## 这是什么

`project-background-goal` 是一个为 PM 设计的 **AI Skill / Agent 单元**,回答四个问题:

1. 为什么现在要做这件事?
2. 现在是怎么做的,哪里出了问题?
3. 做成后要改变什么结果,怎样判断成功?
4. 哪些角色、约束和未知会影响判断?

它**不写用户旅程、功能清单、页面、字段、API、技术实现**;这些留给下游 skill。

## 为什么不同(Smaply / FigJam / Spec Kit / Notion AI 都做不到的)

| 护城河 | 我们怎么做的 |
|---|---|
| **AI 永远不能写 `confirmed`** | 治理伴随文件强制记录 PM/业务方人工确认,AI 仅能产出 `ready_for_human_review` |
| **治理伴随文件外置** | 主文档 = 人读,治理伴随文件 = 机器读;**主文档绝不出现 SRC/六态表/哈希锚点/Constitution** |
| **项目类型三选一强制** | 重构 / 从 0 到 1 / 迭代 三选一,AI 先给推荐 + 证据,PM 选择,治理文件记录 AI vs PM 覆盖 |
| **可选项目专属基线读取** | 若 PM 指定基线(如飞书/钉钉上的项目章程),用对应 CLI 读取原文,**强制把基线拆为 4 类**(明确决定 / 业务示例 / 讨论建议 / AI 解读) |
| **材料隔离测试模式** | 测试时只把 PM 明确指定的材料当作事实输入;reference / 旧产物 / 模型记忆只能作为方法背景 |
| **统一校验器错误格式** | `validate_artifact.py` 输出 8+ 字段(severity / blocking / check_id / location / expectation / repair_hint 等) |
| **可独立运行** | 不依赖 pipeline / registry / 下游产物,单文件可跑 |

## 5 分钟上手

### 1. 直接对话使用

把以下 prompt 贴给支持 SKILL 协议的 AI(Claude Code / Codex / Trae 等),并附上你的原始材料:

```text
使用 $project-background-goal 处理以下项目材料:
[在这里贴原始材料 — 会议纪要 / 邮件 / PPT / 口述]

请按 SKILL.md 的 8 步循环执行:
Preflight → Intake → Think → Clarify → Generate → Audit → Human Gate → Commit。
```

### 2. 安装到项目里

把 `skills/project-background-goal/` 复制到以下任一目录:

- `<projectRoot>/.claude/skills/project-background-goal/`
- `<projectRoot>/.codex/skills/project-background-goal/`
- `<projectRoot>/.trae/skills/project-background-goal/`
- `<projectRoot>/.agents/skills/project-background-goal/`

支持 SKILL frontmatter 的 IDE 会自动识别。

### 3. 单文件校验产物

```bash
python3 skills/project-background-goal/scripts/validate_artifact.py path/to/background-goal.md --json
```

校验器自动查找同目录 `background-goal.governance.md`,只依赖 Python 标准库。

### 4. 项目级会议基线(可选)

本 skill 不绑定任何平台或工具。若 PM 指定了项目级基线材料(会议纪要、项目章程、既有 PRD 等),在治理伴随文件登记「项目级会议基线(可选)」段即可:

```bash
python3 skills/project-background-goal/scripts/validate_artifact.py background-goal.md --json
```

校验器只在该段**已登记**时检查三项内容:读取命令、四类拆分、使用位置。未登记不做任何要求。

## 产物结构

```
your-project/
├── background-goal.md              # 人读:背景、现状、问题、目标、角色、约束、边界、待确认
└── background-goal.governance.md   # 机器读:类型判断、PM 选择、来源、知识状态、澄清、Audit、哈希
```

模板文件见 [`skills/project-background-goal/templates/background-goal.md`](skills/project-background-goal/templates/background-goal.md) 和 [`skills/project-background-goal/templates/background-goal.governance.md`](skills/project-background-goal/templates/background-goal.governance.md)。

## 8 步工作循环

```
Preflight    理解材料,预判业务事件
   ↓
Intake       按角色还原实际行为
   ↓
Think        第一性原理 / 系统思维 / 对抗性审视 / 逆向验证 / 知识边界
   ↓
Clarify      每轮最多 5 高影响问题,A/B/C 选项,渐进式(先业务本质 → 现状 → 目标 → 范围)
   ↓
Generate     生成主文档 + 治理伴随文件
   ↓
Audit        以"项目组成员"视角反读主文档
   ↓
Human Gate   PM / 业务方在治理文件记录 `confirmed`
   ↓
Commit       机器校验产物结构与一致性
```

详细规则见 [`skills/project-background-goal/SKILL.md`](skills/project-background-goal/SKILL.md)。

## 项目类型三选一(本 skill 的强制机制)

```text
我判断是【推荐类型】,依据是【证据】。
请选择:重构 / 从 0 到 1 / 迭代
```

| 类型 | 必须说清楚 |
|---|---|
| **重构** | 现状、问题证据、before -> after,尽量量化基线/目标值/衡量方式/时间 |
| **从 0 到 1** | 已有人工做法或首个完整业务流程、参与角色、要建立的业务过程 |
| **迭代** | 为什么加/改、加/改后做什么 — 简短,需求级,不膨胀成项目章程 |

## 反模式(本 skill 明确禁止)

- ❌ 不用成品 PRD 自证
- ❌ 不把 AI 推断写成事实
- ❌ 不替 PM 确认项目类型、范围、目标或取舍
- ❌ 不把功能、页面、字段、API 或技术实现写进背景目标
- ❌ 不为了"完整"复制一套固定长模板
- ❌ 不把治理记录混入人读主文档

## 与下游 skill 的边界

`project-background-goal` **不替下游定义**:

- 用户旅程 → `user-journey` skill
- 功能清单 → `feature-list`
- 功能流程 → `functional-flow`
- 页面设计 → `page-design`
- 业务规则 → `business-rules`
- 验收标准 → `acceptance-criteria`

主文档写"我们要把货物销售流程重构,目标是 30 天发货率从 70% → 95%",**不写**"在货物列表页加一个'一键发货'按钮"。

## 引用文档(可选阅读)

- [`skills/project-background-goal/references/thinking-framework.md`](skills/project-background-goal/references/thinking-framework.md) — 思考透镜
- [`skills/project-background-goal/references/background-by-type.md`](skills/project-background-goal/references/background-by-type.md) — 三类项目差异
- [`skills/project-background-goal/references/output-contract.md`](skills/project-background-goal/references/output-contract.md) — 输出契约
- [`skills/project-background-goal/references/audit-checklist.md`](skills/project-background-goal/references/audit-checklist.md) — 审计清单
- [`skills/project-background-goal/references/anti-patterns.md`](skills/project-background-goal/references/anti-patterns.md) — 反模式
- [`skills/project-background-goal/references/question-patterns.md`](skills/project-background-goal/references/question-patterns.md) — 高影响提问模式

## 调研与对比

本 skill 在 2026-08-27 项目会议中讨论形成,基于以下对照:

- 与 Smaply / Custellence / FigJam / Miro 相比 — 我们不画图,我们写可被签字的文档
- 与 GitHub Spec Kit 相比 — 我们不只产 spec.md,我们治理 type / scope / goal 三个 PM 决策
- 与 Notion AI 相比 — 我们**不**让 AI 替 PM 确认;AI 只产 `ready_for_human_review`
- 与 prd-generator / prd-development 相比 — 我们**不**在背景里写功能方案

### 市场已有 vs 我们差异化（README 只突出差异化）

下列能力是市场已有、不必再强调:

- Markdown / 多角色 / 多阶段 / 触点 / 痛点 / 机会 — Smaply / NN/g / FigJam / Miro 标配
- "AI 写 PRD" — prd-generator / Notion AI / Copilot 等十几家在做
- "PM-AI 对话流程" — req-clarifier / incremental-prd-collaboration 都有

下列能力是本 skill 独有:

- **AI 永远不能写 confirmed**(治理伴随文件 + 授权清单 + 哈希)
- **治理伴随文件外置**(主文档不被机器治理信息污染)
- **项目类型三选一强制**(AI 推荐 + PM 选择 + 覆盖记录)
- **可选项目专属基线 + CLI 读取命令记录在治理文件**
- **材料隔离测试模式**(只把 PM 指定的材料当作事实输入)
- **统一校验器错误格式**(8+ 字段,可机器消费)
- **可独立运行**(不依赖 pipeline / registry)

## 单点验证示例

```bash
# 把会议纪要 / 邮件 / PPT / 口述 放进一个目录
mkdir /tmp/bg-test
cp your-meeting.md /tmp/bg-test/
# 让支持 SKILL 协议的 AI 跑本 skill,贴材料给它
# 产物应放在 /tmp/bg-test/background-goal.md 与 background-goal.governance.md

python3 skills/project-background-goal/scripts/validate_artifact.py \
  /tmp/bg-test/background-goal.md --json
```

验证通过后,再让 PM 或另一位产品经理读主文档,问四个问题:
1. 项目为什么做?
2. 现在怎么做 / 哪里有问题?
3. 做成什么样 / 怎样判断成功?
4. 谁 / 约束 / 未知?

任何一题答不出 → 回到 skill §1 Preflight / §3 Clarify 补料,**不要绕过 skill 修补产物**。

## License

MIT — 见 [LICENSE](LICENSE)。

## 致谢

本 skill 由 `01_项目仓库区/Project_001_产品AI脚手架` 项目独立开源,与上游项目共享 001 会议基线方法论,但不绑定任何特定组织或内部项目数据。