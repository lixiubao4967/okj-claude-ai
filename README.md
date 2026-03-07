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

## 注意事项

- 升级：`npm update -g @anthropic-ai/claude-code`
- 如使用公司网络，可能需要配置代理
