# Claude 认证架构师（基础）备考指南

> **来源**：第三方文章解读。整理自社区开源备考仓库 [paullarionov/claude-certified-architect](https://github.com/paullarionov/claude-certified-architect)（[guide_zh.md](https://github.com/paullarionov/claude-certified-architect/blob/main/guide_zh.md)）。非 Anthropic 官方出品，仅作学习参考；完整原文与练习题请见原仓库。

## 概述

**Claude Certified Architect — Foundational**（Claude 认证架构师 — 基础）是面向**解决方案架构师**的认证，验证使用 Claude 技术栈构建生产应用的能力。考察四大核心技术：

- **Claude API** — 消息接口、`tool_use`、批处理
- **Claude Agent SDK** — 多代理编排、子代理、生命周期钩子
- **Claude Code** — `CLAUDE.md`、MCP、技能、规划模式
- **模型上下文协议（MCP）** — 后端集成的工具与资源

考题基于真实场景：客服代理、多代理研究管道、CI/CD 集成、开发者生产力工具、结构化数据提取等。

理想候选人应具备 **6 个月以上**上述技术的实践经验。

---

## 考试格式

| 参数 | 值 |
|---|---|
| 题型 | 单选题（4 选 1） |
| 评分 | 100–1000 分制，**及格 720 分** |
| 猜测惩罚 | 无（每题都要作答） |
| 场景 | 从 8 个候选场景中随机抽 4 个 |

### 五大考试领域（按权重）

| 领域 | 权重 | 核心内容 |
|---|---|---|
| 1. 代理架构与编排 | **27%** | 代理循环、协调者-子代理、多步工作流、SDK 钩子 |
| 2. 工具设计与 MCP 集成 | **18%** | 工具描述、结构化错误、MCP 服务器配置 |
| 3. Claude Code 配置与工作流 | **20%** | `CLAUDE.md` 层级、自定义命令、技能、规划 vs 直接执行 |
| 4. 提示工程与结构化输出 | **20%** | 少样本、JSON 模式、验证-重试、批处理 |
| 5. 上下文管理与可靠性 | **15%** | 长对话处理、升级模式、错误传播、人在回路 |

### 八个考试场景

1. **客户支持代理** — 用 Agent SDK 处理退货/账单/账户，MCP 工具（`get_customer`、`lookup_order`、`process_refund`、`escalate_to_human`），目标首次解决率 80%+ 且正确升级
2. **用 Claude Code 生成代码** — 代码生成/重构/调试，配合自定义斜杠命令与 `CLAUDE.md`
3. **多代理研究系统** — 协调代理委派子代理（网络研究/文档分析/综合/报告），输出带引用
4. **开发者生产力工具** — 探索陌生代码库、生成样板，用内置工具 + MCP
5. **CI/CD 中的 Claude Code** — 自动代码审查、测试生成、PR 反馈，最小化误报
6. **结构化数据提取** — 从非结构化文档提取，用 JSON 模式验证，高精度
7. **对话式 AI 架构** — 多轮上下文管理、指令持久、记忆策略、模糊输入处理
8. **Agentic AI 工具** — 原指南标注内容缺失（社区征集中）

---

## 核心考点速记

### 1. 代理循环基础

判断 API 响应的 `stop_reason` 字段：

- `"tool_use"` → 执行工具，把结果回传，**继续循环**
- `"end_turn"` → 给出最终答案，**停止**

### 2. 工具设计

- **工具描述是 LLM 选择工具的首要依据** — 描述要清晰、彼此区分，写明输入格式与边界
- MCP 错误用 `isError` 标志返回，且应**结构化**：错误类别、是否可重试、上下文，让协调代理能智能恢复
- 区分"无结果"与"搜索失败"，不要用空结果冒充成功

### 3. 子代理与编排

- 子代理运行在**隔离上下文**中，**不继承**父对话历史 → 所需上下文必须在任务提示里**显式传入**
- 协调代理的 `allowedTools` 必须包含 `"Task"` 才能生成子代理
- 一个协调代理响应里可包含多个 `Task` 调用 → **并行**运行子代理

### 4. 错误处理（四类）

| 类别 | 示例 | 可重试 | 动作 |
|---|---|---|---|
| 瞬态 | 超时、503、网络故障 | 是 | 指数退避重试 |
| 验证 | 格式无效、缺字段 | 否（先修输入） | 改请求后重试 |
| 业务 | 违反政策、超阈值 | 否 | 向用户解释，给替代方案 |
| 权限 | 拒绝访问 | 否 | 升级 |

> **反模式**：通用错误文案、静默抑制、单点失败即中止全流程、子代理内无限重试。子代理应本地恢复 1–2 次后把错误传给协调代理。

### 5. 升级与人在回路

**可靠的升级触发条件**：用户明确要求人工、政策未覆盖、无法继续推进、操作超过阈值（最好用钩子强制而非提示）、客户搜索出现多个匹配。

> **不可靠**：情感分析、模型自评置信度（1–10）、自动分类器——这些与案例复杂度不相关或校准差。

升级时传递**结构化交接摘要**（人工只看摘要，看不到完整对话，须自包含）。

### 6. Claude Code 配置

- **`CLAUDE.md` 层级**：用户级 `~/.claude/`（个人偏好）；项目级 `.claude/` 或仓库根（团队规范）
- **`.claude/rules/`**：带 YAML frontmatter，用 `paths` glob 模式条件加载
- **`.claude/skills/`**：可复用命令，`context: fork` 隔离输出，`allowed-tools` 限制范围
- **MCP 配置**：项目级 `.mcp.json`（支持环境变量替换）；个人/实验性用 `~/.claude.json`

**规划模式 vs 直接执行**：

| 用规划模式 | 用直接执行 |
|---|---|
| 大规模更改（45+ 文件）、库迁移 | 有明确堆栈跟踪的单文件修复 |
| 存在多种可行方案、架构决策 | 添加一个校验检查 |
| 不熟悉的代码库 | 已充分理解、无歧义的更改 |

组合：规划模式调查设计 → 用户审批 → 直接执行实施。

### 7. CI/CD 集成

```bash
claude -p "Analyze this pull request for security issues"
```

`-p`（`--print`）= 非交互模式，处理提示后打印到 stdout 退出，是在 CI/CD 中运行 Claude 的**唯一正确方式**。

### 8. 提示工程与结构化输出

- 结构化输出用 `tool_use` + JSON 模式，**不要**用纯文本请求 JSON
- 验证-重试：重试提示里要包含**原文 + 错误的提取结果 + 具体错误**
- 复杂审查拆成「逐文件分析 + 单独的集成检查」，而非一次性多文件审查
- 批处理 API（`custom_id` 关联结果）适合大批量、非实时任务

### 9. 上下文管理

- 把关键事实**提取到独立块**中，避免被汇总冲淡
- 用 `PostToolUse` 钩子**裁剪工具返回**，只保留相关字段
- 长任务用草稿文件、委托子代理保护主上下文、结构化状态持久化（崩溃恢复）
- `/compact` 压缩历史（风险：精确数值/日期可能丢失）；`/memory` 编辑 `CLAUDE.md` 跨会话持久

---

## 备考建议

- 偏重**动手实现**：跑通真实的代理循环、为真实团队项目配置 Claude Code、设计描述清晰的 MCP 工具、用具体输入/输出迭代提示
- 原仓库练习区含 **75+ 道场景题**（客服、研究系统、CI/CD、代码生成等），强烈建议刷完

## 官方文档（备考权威来源）

- [Claude API — Messages](https://platform.claude.com/docs/en/api/messages) / [Tool Use](https://platform.claude.com/docs/en/build-with-claude/tool-use) / [Message Batches](https://platform.claude.com/docs/en/build-with-claude/message-batches)
- Claude Agent SDK — [概述](https://platform.claude.com/docs/en/agent-sdk/overview) / [Hooks](https://platform.claude.com/docs/en/agent-sdk/hooks) / [Subagents](https://platform.claude.com/docs/en/agent-sdk/subagents) / [Sessions](https://platform.claude.com/docs/en/agent-sdk/sessions)
- [模型上下文协议（MCP）](https://modelcontextprotocol.io/) — [Tools](https://modelcontextprotocol.io/docs/concepts/tools) / [Resources](https://modelcontextprotocol.io/docs/concepts/resources)
- Claude Code — [文档](https://code.claude.com/docs/en/overview) / [内存](https://code.claude.com/docs/en/memory) / [技能](https://code.claude.com/docs/en/skills) / [Hooks](https://code.claude.com/docs/en/hooks) / [子代理](https://code.claude.com/docs/en/sub-agents) / [MCP](https://code.claude.com/docs/en/mcp) / [GitHub Actions](https://code.claude.com/docs/en/github-actions) / [无头模式](https://code.claude.com/docs/en/headless)
- [提示工程指南](https://platform.claude.com/docs/en/build-with-claude/prompt-engineering/overview) / [扩展思维](https://platform.claude.com/docs/en/build-with-claude/extended-thinking) / [Anthropic Cookbook](https://github.com/anthropics/anthropic-cookbook)
