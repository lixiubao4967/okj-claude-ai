# OKG Claude 快速入门完整指南

> 基于 OKG 内部文档整理，涵盖 Skills、MCP、Agents、Hooks、Plugins 全流程。

---

## 目录

1. [什么是 Skill](#1-什么是-skill)
2. [什么时候该创建 Skill](#2-什么时候该创建-skill)
3. [如何创建 Skill（4步）](#3-如何创建-skill)
4. [发布 Skill 到 Marketplace](#4-发布-skill-到-marketplace)
5. [使用团队的 Skills/Plugins](#5-使用团队的-skillsplugins)
6. [加入支持群](#6-加入支持群)
7. [进阶：Claude 生态系统全景](#7-进阶claude-生态系统全景)
8. [进阶：编写 SKILL.md 完整规范](#8-进阶编写-skillmd-完整规范)
9. [进阶：MCP 连接外部系统](#9-进阶mcp-连接外部系统)
10. [进阶：Agents 虚拟团队](#10-进阶agents-虚拟团队)
11. [进阶：Hooks 自动化规则](#11-进阶hooks-自动化规则)
12. [进阶：Plugins 打包一切](#12-进阶plugins-打包一切)

---

## 1. 什么是 Skill

> 想象你雇了一个很厉害的人，但每周一都要重新解释你的汇报格式、语气偏好、具体清单。每次都要说。

**Skill = Claude 的"剧本"（saved playbook）**

- 把重复的指令写一次，之后每次触发 Claude 自动执行
- 本质：把你的 prompt 封装成可复用的命令

---

## 2. 什么时候该创建 Skill

**判断标准：可复用的 → 就该变成 Skill**

当你发现自己：

- 反复输入同样的指令
- 从笔记里复制粘贴"完美 prompt"
- 心里想："要是 Claude 能直接知道我喜欢怎么做就好了"

→ 是时候建 Skill 了。

---

## 3. 如何创建 Skill

**业务场景示例**：你上线了新功能，想了解用户反应。

### Step 1 — 创建 Skill

把一句话发给 Claude：

```
Create a skill to pull report on customer sentiment analysis of OKX
```

### Step 2 — 审查输出

Claude 会自动生成：
- Skill 的指令内容
- 需要的参数（如有）
- 预期的输出格式

### Step 3 — 测试 Skill

用真实输入跑一遍，验证：
- 格式是否正确？
- 语气是否合适？
- 结构是否可复用？

### Step 4 — 下载 Skill 文件

Claude 会产出两个文件：
- `xxx.skill`（Skill 定义文件，用于复用和分享）
- 实际生成的报告（如 `.pdf`）

---

## 4. 发布 Skill 到 Marketplace

### 前提条件

- `oli-cli` 已安装（IT 自动部署）
- 连接公司 WiFi 或 VPN

> 检查方法：Claude Desktop → Co-work → `+` → Connectors，确认 `oli-cli` 存在。

### 如果 oli-cli 未安装，手动安装

```bash
sudo jamf policy -id 308
```

输入 Mac 登录密码，等待安装完成，看到以下提示即成功：

```
[SUCCESS] oli-cli installed
Registered oli-cli MCP server in Claude Desktop config.
Restart Claude Desktop to apply changes.
```

重启 Claude Desktop 后生效。

### 提交步骤（3步）

1. 在 Claude Cowork 输入框输入：`upload this skill`
2. 点 `+` → `Add files or photos`，上传 `.skill` 文件（也支持 `.zip`、`.md`）
3. 点 `Let's go`

### 提交后的审核流程

```
提交 → 工程团队审核（相似 Skill 会合并处理）
     → 批准：上架 Marketplace，所有人可下载
     → 拒绝：Reviewer 通知你
```

**遇到问题联系：** 于畅 Hilo Yu、Kevin Qu 曲小军、Kieran Hu Cang 胡仓

---

## 5. 使用团队的 Skills/Plugins

**路径：** Claude Desktop → `Co-work` 标签 → `+` → `Add plugins...`

在弹出面板中选择 `Your organization` Tab，找到 OKG 内部已上架的 Plugin，点击安装即可。

---

## 6. 加入支持群

Lark 群：**Claude: adoption**（OKG 内部）

仅限 OKG 组织成员加入，二维码永久有效。用于遇到问题、交流 Skill 创建、寻求咨询。

---

## 7. 进阶：Claude 生态系统全景

```
Skills ──instructs──▶
MCP    ──connects──▶   Claude (AI Core)  ──spawns──▶ Agents ──▶ Sub-agents
Hooks  ──intercepts──▶
                │
                └──distributed via──▶ Plugins
```

| 组件 | 作用 |
|------|------|
| **Skills** | Markdown 指令文件，教 Claude 怎么做某件事 |
| **MCP** | 连接外部工具/API（Jira、Slack、数据库等） |
| **Agents** | 自主完成多步骤任务，可并行派生子 Agent |
| **Hooks** | 在生命周期关键节点自动触发（事件驱动脚本） |
| **Plugins** | 打包 Skills + MCPs + Tools 的可安装合集 |

**进阶路径：**

```
Skill（会做事）→ MCP（能拿数据）→ Agent（自主执行）→ Hook（自动化护栏）→ Plugin（打包分发）
```

---

## 8. 进阶：编写 SKILL.md 完整规范

### 文件结构

```
my-skill/
└── SKILL.md
```

SKILL.md 包含两部分：**frontmatter 元数据** + **正文指令**

### 完整示例

```markdown
---
name: meeting-notes-formatter
description: Use this skill whenever someone wants to clean up,
  format, or summarize meeting notes. Trigger when the user
  pastes raw notes, mentions a meeting recap, or asks for
  action items to be extracted from a conversation.
---

# Meeting Notes Formatter

Help the user turn messy meeting notes into a clean summary.

## Step 1: Read the notes
Ask the user to paste their raw meeting notes if they haven't already.

## Step 2: Extract the key points
Find and list: who attended, what was decided, and what the next steps are.

## Step 3: Format the output
Always produce the summary in this format:

- **Meeting date:**
- **Attendees:**
- **Key decisions:**
- **Action items:** (with owner's name)
```

### Description 写法（最关键）

description 决定 Claude 是否会在正确时机调用你的 Skill。

❌ 太模糊：
```
Use this for meeting notes.
```

✅ 够清晰（3-5句，包含触发短语）：
```
Use this skill whenever someone wants to clean up, organize, or summarize
meeting notes. Trigger when the user pastes raw notes, mentions a meeting
recap, asks to extract action items, or wants a cleaner version of notes
from a call or standup. Use it even if they don't say "meeting notes" explicitly.
```

**好的 description 回答三个问题：**
1. 这个 Skill 做什么？
2. 什么情况下用？（用户会说什么？）
3. 相关的同义词/短语有哪些？

> 最常见错误：description 写得太短。宁可多写，不要少写。

### 写指令的四个原则

**1. 用编号步骤**

❌ 难以遵循：
```
Process the data, make sure to clean it first, then output something useful, also include totals.
```

✅ 清晰可执行：
```
Step 1: Ask the user for their data file.
Step 2: Remove empty rows and fix column headers.
Step 3: Add a totals row at the bottom.
Step 4: Save the cleaned file and give the user a download link.
```

**2. 展示期望的输出格式**

```markdown
## Output format
Always produce the weekly report in this format:

**Week of:** [date]
**Completed this week:** (bullet list)
**In progress:** (bullet list)
**Blockers:** (bullet list, or "None")
**Planned for next week:** (bullet list)
```

**3. 解释"为什么"而不只是"做什么"**

❌ 只说规则：
```
ALWAYS save the file to the outputs folder. NEVER save anywhere else.
```

✅ 说明原因：
```
Save the file to the outputs folder — that's the only place where the user
can access and download it. Files saved elsewhere won't be visible to them.
```

**4. 保持简短聚焦**

- 目标：1-2 页纯文字指令
- 超出了就拆成两个 Skill，一个 Skill 只做一件事

### 可复用模板

```markdown
---
name: [your-skill-name — 小写+连字符，如 weekly-report-writer]
description: [描述 Skill 做什么 + 什么时候用，包含具体触发短语，3-5句]
---

# [Your Skill Name]

[一句话概述这个 Skill 做什么]

## Step 1: [第一步]
[具体指令]

## Step 2: [第二步]
[具体指令]

## Step 3: [依此类推]
[具体指令]

## Output
[描述最终输出是什么样的，提供模板示例]
```

### 发布前检查清单

- [ ] 名称：小写 + 连字符（如 `weekly-report-writer`）
- [ ] description 包含 3-5 个触发短语
- [ ] description 说明了"即使用户没有明确说 X 也要触发"
- [ ] 指令有编号步骤
- [ ] 有输出格式示例
- [ ] 指令解释了为什么，不只是做什么
- [ ] 用 2-3 种不同问法测试过
- [ ] 只做一件专注的事

---

## 9. 进阶：MCP 连接外部系统

MCP（Model Context Protocol）让 Claude 直接访问数据库、CRM、API，无需手动粘贴数据。

### 创建 MCP 的起手式

直接告诉 Claude：

```
I want to build an MCP server that connects Claude to [数据源]. Help me create it.
```

例如：
```
I want to build an MCP server that connects Claude to our MySQL database so it can query customer orders.
I want to build an MCP server for the Payment API so Claude can look up payment status.
I want to build an MCP that reads from our internal REST API at api.company.com.
```

### Claude 会生成什么

- 完整可运行的 MCP server 代码
- 安装说明（如何安装和运行）
- 可调用的工具列表（如 `search_customers`、`get_order_by_id`）

### 部署步骤

1. 先本地测试（Claude 会引导你）
2. 验证工具列表：让 Claude 列出它通过 MCP 能做什么
3. 部署到服务器或公司基础设施
4. 添加到 Claude 的 MCP 设置，让团队使用

### 上线前必须做到

- [ ] API 访问权限已申请（key 或 OAuth 已配置）
- [ ] **写操作** MCP 已经过 IT/安全团队审批
- [ ] 访问权限限定在正确的人员范围
- [ ] 敏感数据字段已从 Claude 可访问范围中排除
- [ ] 至少一人用真实查询测试过

> 只读 MCP 风险低，可直接使用；写操作 MCP 必须先获得审批。

---

## 10. 进阶：Agents 虚拟团队

Skill + MCP 让一个 Claude 很强大。但有些工作太大、太复杂、步骤太多，一个人做不来——这就是 Agent 的场景。

### Sub-agent vs Agent Team

| | Sub-agent | Agent Team |
|--|--|--|
| **架构** | 主 Agent 分配任务，各自独立执行 | Team Lead + 多个可互相通信的 Claude |
| **通信** | 只向主 Agent 汇报结果 | Agent 之间可直接传消息、广播、共享发现 |
| **任务分配** | 主 Agent 手动分配 | 任务列表共享，完成后自动领取下一个 |
| **适合场景** | 并行独立任务，成本低 | 复杂任务、需要互相质疑/同行评审 |
| **Token 消耗** | 低 | 显著更高 |

### 如何启动

**Sub-agent（简单并行）：**
```
Use sub-agents to research our top 4 competitors simultaneously —
each agent fetches pricing, product features, and recent news for one competitor.
```

**Agent Team（协作自协调）：**
```
Create an agent team to investigate why our checkout flow has a high drop-off rate.
Spawn 3 teammates: one on UX/design issues, one on technical/performance issues,
one playing devil's advocate. Have them challenge each other's theories and
converge on the most likely root cause.
```

### OKG 实战案例：PMO + Star Agent

```
start an agent team with 3 pmo agent and 1 star agent

Based on the last 3 days of issues from "OEX Feedback Channel" and
"All-staff Product Feedback" lark channel:
- 3 PMO agents independently find top 3 user frictions for US, EEA and other markets
- VIP issues get prioritized
- 3 PMO agents debate with facts and vote, majority wins
- Final outcome reviewed by Star agent
- Report: top 3 user frictions + strategist's views
```

**流程：**
1. 3个 PMO Agent 独立分析 → 各自得出结论
2. 三方科学辩论 + 民主投票 → 确定 Top 3
3. 交给 Star Agent（用 Star 的 claude.md）审核
4. 产出供人工复核、修改、批准

### 选择建议

| 用 Sub-agent | 用 Agent Team |
|--|--|
| 任务相互独立，不需要彼此输出 | 任务需要共享发现、互相审查 |
| 想降低 Token 成本 | 需要竞争假说或同行评审 |
| 简单并行工作已足够 | 任务复杂，值得自协调 |

> Agent Teams 是实验性功能，默认关闭，需显式启用。

---

## 11. 进阶：Hooks 自动化规则

Hook = 在 Claude 工作的特定时刻自动执行的 shell 命令。定义一次，永远生效。

### 有 Hook vs 没 Hook

| 没有 Hook | 有 Hook |
|-----------|---------|
| Claude 改代码 → 你手动跑测试 | 改代码 → 测试自动跑 → Claude 看结果 → 自动修复 |
| Claude 保存文件 → 你记得去格式化 | 保存 → 自动格式化，无需提醒 |
| Claude 完成任务 → 你去检查 | 完成 → 自动发 Slack 消息/写日志/备份 |

### 三类触发时机

| 时机 | 用途 |
|------|------|
| **Before**（行动前） | 拦截、验证、请求确认 |
| **After**（行动后） | 跑测试、记日志、格式化 |
| **When done**（完成时） | 发通知、触发下一步流程 |

### 典型使用场景

**工程/DevOps：**
- 每次改文件自动跑 linter，结果反馈给 Claude
- 拦截所有触及 production 的 shell 命令，要求确认
- 代码改完自动跑测试套件

**运营：**
- 记录 Claude 每次使用的工具到审计系统
- 保存文件时自动备份到归档目录
- 工作流完成后推送摘要到 Slack

**安全/合规：**
- Claude 读文件前检查是否在受限目录
- 每次调用外部 API 记录请求日志
- 自动标记并拦截匹配敏感数据模式的命令

### 创建步骤

**Step 1 — 描述触发时机和行为：**
```
I want tests to run automatically every time Claude edits a Python file.
I want a Slack message sent to #workflow-done whenever Claude finishes a task.
I want Claude blocked from running any command with 'delete' in it unless I confirm.
```

**Step 2 — 由 IT 或开发配置到 Claude 设置文件**（简单 hook 约 10-15 分钟）

**Step 3 — 测试一次，确认生效**

不知道怎么写，直接问 Claude：
```
I want something to happen automatically when Claude [描述时机]. Can you help me write the hook?
```

### 值得建 Hook 的场景清单

- [ ] 每次 Claude 改文件后跑测试/验证
- [ ] 记录 Claude 操作的审计日志
- [ ] Claude 编辑前自动备份文件
- [ ] 工作流完成后通知团队
- [ ] 禁止 Claude 碰某些文件或命令
- [ ] 完成后触发流水线下一步

---

## 12. 进阶：Plugins 打包一切

Plugin = 把 Skills + MCP + Agents + Hooks 打包成一个可安装/可分享的单元。

### 目录结构

```
plugin-name/
├── .claude-plugin/
│   └── plugin.json     ← 插件元数据（必须）
├── commands/           ← 斜杠命令（可选）
├── agents/             ← 专属 Agent（可选）
├── skills/             ← Skills（可选）
├── hooks/              ← 事件钩子（可选）
├── .mcp.json           ← 外部工具配置（可选）
└── README.md           ← 文档（可选）
```

**一句话理解：**

> 装一个 Plugin = 一次性装好所有配套能力，团队成员直接用，不用分别配置每个组件。

---

## 延伸学习资源

- Anthropic 官方免费课程
- Claude 官方 Skills 指南
- Andrew Ng AI Agent 入门课程
- Plugins 官方文档：https://github.com/anthropics/claude-code/blob/main/plugins/README.md
