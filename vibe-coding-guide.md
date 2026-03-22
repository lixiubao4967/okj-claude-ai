# Vibe Coding 指南

**与 AI 结对编程，将想法变为现实的工作方法论**

> 原始资料来源：[tukuaiai/vibe-coding-cn](https://github.com/tukuaiai/vibe-coding-cn)，本文为中文整理版。

---

## 这是什么？

**Vibe Coding** 是一套以 AI 为核心的编程工作流。它不是某个具体工具，而是一种**思维方式和操作方法**——让你通过和 AI 持续对话，把脑海中的想法一步步变成可运行的代码。

关键词是"结对"：你负责明确目标、审查方向、把控质量；AI 负责生成代码、处理细节、加速执行。

**核心理念只有一句话**：*规划就是一切。* 不要让 AI 自主规划，否则代码库会失控。

---

## 核心理念

### 道：你应该怎么想

- **上下文决定一切**：给 AI 的背景信息质量直接决定输出质量。垃圾进，垃圾出。
- **先结构，后代码**：动手前一定要把模块拆好、接口定好，否则技术债还不完。
- **目的主导**：开发过程中的一切动作都要围绕"这步的目的是什么"展开。
- **奥卡姆剃刀**：如无必要，勿增代码。能用现成库就不重复造轮子。
- **一次只做一件事**：极致专注，每次迭代只推进一个模块或一个功能点。
- **逆向思维**：先明确你的需求，再从需求出发逆向构建代码。

### 法：你应该怎么做

- 写一句话目标 + 明确非目标（哪些不在范围内）
- 按职责拆模块，功能不要重叠（正交性）
- 接口先行，实现后补
- 先问 AI 有没有合适的开源库，能抄不写
- 一次只改一个模块，改完验证再继续
- 文档要在开发过程中实时维护，而不是事后补写

### 术：技术层面的原则

- Debug 时只提供：**预期行为 vs 实际行为 + 最小复现步骤**，不要大段粘贴代码
- 测试逻辑可以交给 AI 生成，但断言内容要人工审查
- 代码量太大时，切换新会话，避免上下文污染

---

## 工具安装

### 必选：AI 编程助手

根据预算选一种即可：

**Claude Code（推荐）**

需要 Anthropic API Key 或 Claude.ai Pro/Max 订阅。

```bash
npm install -g @anthropic-ai/claude-code
claude  # 首次运行完成登录授权
```

**Codex CLI（OpenAI，免费额度）**

```bash
npm install -g @openai/codex
codex  # 首次运行完成登录授权
```

**Gemini CLI（Google，完全免费）**

```bash
npm install -g @google/gemini-cli
gemini  # 首次运行完成登录授权
```

### 可选：免费获取高端模型

| 工具 | 说明 | 获取方式 |
|------|------|---------|
| Kiro | 免费使用 Claude Opus 4.5 | 官网注册 |
| antigravity | 免费，支持 Claude Opus 4.5 和 Gemini 3.0 Pro | Google 提供 |
| AI Studio | 支持 Gemini 3.0 Pro | Google 官网 |
| Qwen CLI | 阿里巴巴，CLI 有免费额度 | 官网注册 |

### 可选：IDE 增强

- **Cursor**：内置 AI 的 VSCode 分支，开箱即用
- **Windsurf**：新用户有免费额度
- **Augment**：上下文引擎，提升 AI 理解能力

### 可选：辅助工具

- **Ollama**：在本地运行开源模型（完全离线，免费）

  ```bash
  brew install ollama       # macOS
  ollama run llama3         # 下载并运行模型
  ```

- **Superwhisper**：语音转文字，用说话代替打字与 AI 对话（macOS）
- **tmux**：多终端管理，方便同时运行多个 AI 会话

  ```bash
  brew install tmux         # macOS
  sudo apt install tmux     # Ubuntu/Debian
  ```

---

## 如何使用

记忆库（memory-bank）是整个工作流的核心。**AI 没有跨会话记忆**，每次新开对话它就忘了上次做了什么。记忆库就是用文件替代 AI 的记忆，每次开始新会话时把这些文件喂给它，它就能接着上次的进度继续干。

### 第一步：新建项目，创建记忆库目录

```bash
mkdir my-project && cd my-project
mkdir memory-bank
git init
```

### 第二步：生成设计文档

打开 Claude Code，把项目想法告诉 AI：

```
我想做一个 [用一两句话描述你的项目]。
请帮我生成一份简洁的设计文档，包含：
- 核心目标（一句话）
- 主要功能列表
- 非目标（明确不做什么）
输出 Markdown，保存为 memory-bank/game-design-document.md。
```

生成后**自己审阅修改**，确保它准确反映你的想法。

`game-design-document.md` 示例结构：

```markdown
# 项目名称

## 核心目标
用一句话描述这个项目要解决什么问题。

## 主要功能
- 功能 A
- 功能 B
- 功能 C

## 非目标（不做）
- 不支持 XX
- 暂不考虑 XX
```

### 第三步：确定技术栈

```
基于 memory-bank/game-design-document.md，
推荐最适合的技术栈，说明每个选型的理由。
输出为 memory-bank/tech-stack.md。
```

`tech-stack.md` 示例结构：

```markdown
# 技术栈

## 前端
- React — 理由：...

## 后端
- FastAPI — 理由：...

## 数据库
- SQLite — 理由：...
```

### 第四步：生成实施计划

```
基于 memory-bank/ 下的设计文档和技术栈，
生成详细的实施计划。
要求：
- 每步要小而具体（一步只做一件事）
- 每步包含验证方式（怎么确认这步做完了）
- 严禁包含代码
输出为 memory-bank/implementation-plan.md。
```

`implementation-plan.md` 示例结构：

```markdown
# 实施计划

## 阶段一：项目初始化
- [ ] 步骤 1：初始化仓库，安装依赖
  - 验证：运行 `npm install` 无报错
- [ ] 步骤 2：搭建基础目录结构
  - 验证：目录结构与技术栈文档一致

## 阶段二：核心功能
- [ ] 步骤 3：实现 XX 模块
  - 验证：手动测试 XX 功能正常
```

### 第五步：创建进度和架构文件（初始为空）

```bash
touch memory-bank/progress.md
touch memory-bank/architecture.md
```

此时记忆库目录结构：

```
memory-bank/
├── game-design-document.md   # 项目设计（你写的）
├── tech-stack.md             # 技术栈（AI 生成，你确认）
├── implementation-plan.md    # 实施计划（AI 生成，你确认）
├── progress.md               # 进度记录（开发中持续更新）
└── architecture.md           # 架构说明（开发中持续更新）
```

### 第六步：开始开发

**每次新开会话时**，先把记忆库喂给 AI：

```
请阅读 memory-bank/ 下的所有文件，了解项目背景和当前进度，
然后继续实施计划中的下一步。
```

**开发循环：**

1. 按 `Shift+Tab` 切换到 Plan Mode，让 AI 说明它打算怎么做，确认后再执行
2. 完成一步后，提交 Git
3. 更新 `memory-bank/progress.md`（标记已完成的步骤）
4. 开新会话，继续下一步

**`progress.md` 更新示例：**

```markdown
# 进度记录

## 已完成
- [x] 步骤 1：初始化仓库（2024-03-01）
- [x] 步骤 2：搭建目录结构（2024-03-01）

## 当前进行
- 步骤 3：实现登录模块

## 遇到的问题
- XX 问题：已通过 XX 方式解决
```

---

## 实用技巧

| 技巧 | 说明 |
|------|------|
| `/rewind` | 一键回滚到上一个状态（Claude Code） |
| `/clear` 或 `/compact` | 清理上下文，避免上下文污染 |
| `think` / `think hard` / `ultrathink` | 触发更深层的思考，强度依次递增 |
| `--dangerously-skip-permissions` | 跳过所有确认弹窗（风险自负） |
| `codex --yolo` | Codex CLI 的同等选项 |

**触发深度思考的关键词强度排序：**

```
think  <  think hard  <  think harder  <  ultrathink
```

---

## 模型选择参考

| 梯队 | 模型 | 特点 |
|------|------|------|
| 第一梯队 | codex-5.1-max、claude-opus-4.5 | 最强编码能力 |
| 第二梯队 | claude-sonnet-4.5、kimi-k2-thinking、gemini-3.0-pro | 性价比高 |
| 第三梯队 | qwen3、grok4 | 国产选项 |

入门建议：用 **Claude Code + claude-sonnet-4.5** 起步，有预算再升级到 Opus。

---

> **记住**：Vibe Coding 的本质不是让 AI 替你思考，而是让你用更少的时间把想法变成现实。规划好了，剩下的交给 AI。
