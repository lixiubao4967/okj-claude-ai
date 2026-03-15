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

## 注意事项

- 升级：`npm update -g @anthropic-ai/claude-code`
- 如使用公司网络，可能需要配置代理
