# Claude Code 安装与初始配置

官方快速入门文档：https://code.claude.com/docs/zh-CN/quickstart

## 前提条件

- Node.js 18+
- 有效的 Anthropic API Key（或通过 Claude.ai Pro/Max 订阅授权）

## 安装

```bash
npm install -g @anthropic-ai/claude-code
```

## 首次登录授权

```bash
claude
```

首次运行会提示登录，按照提示完成授权即可（支持 API Key 或 Claude.ai 账号授权）。

## 升级

```bash
npm update -g @anthropic-ai/claude-code
```

## 在项目中使用

```bash
cd /your/project
claude
```

Claude Code 会自动读取当前 git 仓库的上下文，可直接对话完成编码任务。

## 配置文件位置

| 文件 | 说明 |
|------|------|
| `~/.claude/settings.json` | 全局配置（权限、模型等） |
| `~/.claude/CLAUDE.md` | 全局持久指令 |
| `.claude/settings.json` | 项目级配置 |
| `CLAUDE.md` | 项目级持久指令 |
