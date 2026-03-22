# Claude Code 安装与使用说明

官方快速入门文档：https://code.claude.com/docs/zh-CN/quickstart

---

## 前提条件

- Node.js 18+
- 有效的 Anthropic API Key（或通过 Claude.ai Pro/Max 订阅授权）

---

## 安装

```bash
npm install -g @anthropic-ai/claude-code
```

---

## 首次登录授权

```bash
claude
```

首次运行会提示登录，按照提示完成授权即可（支持 API Key 或 Claude.ai 账号授权）。

---

## 常用命令

| 命令 | 说明 |
|------|------|
| `claude` | 在当前目录启动交互式会话 |
| `claude "你的问题"` | 直接执行单次任务 |
| `claude --help` | 查看帮助 |
| `/help` | 会话中查看可用命令 |
| `/clear` | 清除当前会话上下文 |
| `/memory` | 查看或编辑持久记忆 |
| `/quit` | 退出 |

---

## 在项目中使用

进入项目目录后运行 `claude`，Claude Code 会自动读取当前 git 仓库的上下文，可直接对话完成编码任务。

```bash
cd /your/project
claude
```

---

## 配置文件位置

| 文件 | 说明 |
|------|------|
| `~/.claude/settings.json` | 全局配置（权限、模型等） |
| `~/.claude/CLAUDE.md` | 全局持久指令 |
| `.claude/settings.json` | 项目级配置 |
| `CLAUDE.md` | 项目级持久指令 |

---

## 常用命令与快捷键速查表

### 核心命令（必须会）

| 命令 | 说明 |
|------|------|
| `/init` | 初始化 CLAUDE.md，新项目第一件事 |
| `/compact` | 压缩上下文，长对话续命神器 ⭐ |
| `/model` | 切换模型（简单任务用 haiku，硬骨头用 opus）⭐ |
| `/clear` | 清空对话重新来 |
| `/resume` | 恢复之前的对话 |
| `/memory` | 编辑记忆文件，实现跨会话记忆 |
| `/add-dir` | 添加工作目录 |
| `/mcp` | 管理 MCP 服务器 |
| `/export` | 导出对话到文件 |
| `/vim` | 切换 Vim 模式 |

### 进阶命令（按需使用）

| 命令 | 说明 |
|------|------|
| `/cost` | 查看当前会话花费 |
| `/context` | 可视化上下文使用情况 |
| `/review` | 智能代码审查 |
| `/agents` | 创建和管理智能体 |
| `/hooks` | 管理工具事件钩子 |
| `/config` | 打开配置面板 |
| `/doctor` | 诊断安装问题 |
| `/status` | 查看版本、模型、账户状态 |
| `/security-review` | 安全审查待提交代码 |
| `/pr-comments` | 获取 GitHub PR 评论 |

### 快捷键（效率翻倍）

| 快捷键 | 说明 |
|--------|------|
| `Shift+Tab` | 一键接受所有变更（最常用！）⭐ |
| `ESC` | 中断执行，跑偏了立刻刹车 |
| `ESC+ESC` | 跳转到之前的消息 |
| `Cmd+K` | 打开命令面板 |
| `Cmd+B` | 后台执行命令 |
| `Cmd+O` | 展开/折叠过程信息 |
| `@文件名` | 引用文件 |
| `!命令` | 直接执行 bash 命令 |
| `#信息` | 写入 CLAUDE.md 记忆 |
| `Control+V` | 粘贴图片 |

### 彩蛋

```bash
claude --dangerously-skip-permissions
```

一条命令跳过所有权限确认，Claude 直接放飞自我。适合对项目足够熟悉、代码有 git 兜底的时候用。**新手慎用。**

> **效率组合推荐**：光会 `/compact` + `/model` + `Shift+Tab` 这三个组合，独立开发效率就能甩开大部分人。

---

## Claude Code Channels（手机远程控制）

通过 Telegram 或 Discord 远程给 Claude Code 发任务，Claude 在后台执行完成后再通知你。

### 前提条件

| 条件 | 说明 |
|------|------|
| Claude Code 版本 | v2.1.80 或更高（`claude --version` 查看） |
| 登录方式 | 必须使用 **claude.ai 账号登录**，不支持 API Key 认证 |
| 运行环境 | 本地需安装 **Bun**（`curl -fsSL https://bun.sh/install \| bash`） |
| 功能状态 | 目前处于 **Research Preview** 阶段 |

> **Team / Enterprise 用户额外步骤**：Channels 默认关闭，需要组织管理员先开启：
> `claude.ai → Admin settings → Claude Code → Channels → 开启`
> 或在 managed settings 中设置 `channelsEnabled: true`。
> 开启后，组织内成员才能使用。

---

### Telegram 配置步骤

**第一步：创建 Telegram Bot**
1. 在 Telegram 中搜索 `@BotFather` 并打开
2. 发送 `/newbot`，按提示填写 Bot 名称，获取 **Bot Token**

**第二步：在 Claude Code 中配置**
```bash
/telegram:configure
```
输入 Bot Token，系统会生成一个安全配对码。

**第三步：完成配对**

在 Telegram 中找到你刚创建的 Bot，发送任意消息，完成账号绑定（首次消息会将你的 ID 加入白名单）。

**第四步：开始使用**

直接在 Telegram 向 Bot 发送编程任务，Claude Code 后台执行，完成后回复通知你。

---

### Discord 配置步骤

```bash
/discord:configure
```
流程与 Telegram 类似，配置完成后向 Bot 发送任意消息完成配对。

---

### 工作原理

Channels 基于 **MCP（Model Context Protocol）** 实现，是一个 MCP Server，将外部消息推送进正在运行的 Claude Code 会话。**Session 必须保持开启**（建议跑在后台终端或 tmux 中），消息才能实时送达。

---

### 安全说明

每个 Channel 维护一份**发送者白名单**，只有配对过的 ID 能推送消息，其他人的消息会被静默丢弃。

---

## Claude Code 官方插件指南

> 详见 [`claude-code-plugins-guide.md`](./claude-code-plugins-guide.md)｜**来源：第三方文章解读 + 官方文档**

Anthropic 官方持续扩充 Claude Code 插件生态，涵盖代码审查、安全指导、LSP 语言服务器、开发流程自动化等多个方向。

**推荐插件一览：**

| 插件 | 功能 |
|------|------|
| `claude-code-setup` | 分析代码库，自动推荐 Hooks/Skills/MCP 配置 |
| `claude-md-management` | 维护 CLAUDE.md，补充缺失上下文 |
| `learning-output-style` | 交互式学习模式，鼓励用户亲自编写代码 |
| `hookify` | 自定义 Hook 创建与管理 |
| `security-guidance` | 安全问题实时监控（XSS、注入等） |
| `code-review` | 5 个并行 Agent 自动代码审查 |
| `pr-review-toolkit` | 6 个专业 Agent 全面 PR 分析 |

**快速安装：**

```bash
# 在 Claude Code 中安装任意官方插件
/plugin install <plugin-name>@claude-plugins-official
/reload-plugins
```

---

## 本仓库文档来源说明

| 文档 | 来源 |
|------|------|
| `README.md` | 自编 |
| `gstack-install.md` | 自编 |
| `claude-code-plugins-guide.md` | 第三方文章解读 + 官方文档整理 |

- **自编**：由本仓库维护者原创编写
- **官方**：直接翻译或搬运自 Anthropic 官方文档
- **第三方文章解读**：基于第三方技术博客，结合官方文档整理，非 Anthropic 官方出品

---

## 注意事项

- 升级：`npm update -g @anthropic-ai/claude-code`
- 如使用公司网络，可能需要配置代理
