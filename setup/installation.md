# Claude Code 安装与初始配置

官方快速入门文档：https://code.claude.com/docs/zh-CN/quickstart

> Windows 用户请看 [Windows 安装 Claude Code](windows-installation.md)，那边有完整流程与踩坑记录。

## 前提条件

- Pro / Max / Team / Enterprise / Console 账号（**免费版 Claude.ai 不含 Claude Code**），或第三方 API 提供商（Bedrock / Vertex / Foundry）
- 走 npm 安装才需要 Node.js 22+；原生安装器不需要 Node

## 安装

**原生安装器（推荐，会后台自动更新）：**

```bash
curl -fsSL https://claude.ai/install.sh | bash
```

**Homebrew：**

```bash
brew install --cask claude-code
```

`claude-code` 跟踪稳定渠道（约落后一周，跳过有重大回归的版本），`claude-code@latest` 跟踪最新渠道。Homebrew 安装不自动更新。

**npm：**

```bash
npm install -g @anthropic-ai/claude-code
```

> npm 包和原生安装器装的是**同一个原生二进制**，npm 只是通过 optionalDependencies 把它拉下来链接好，`claude` 运行时不调用 Node —— Node 版本只影响安装那一步。
>
> 不要用 `sudo npm install -g`，会导致权限问题和安全风险。

## 验证

```bash
claude --version    # 应打印类似 2.1.211 (Claude Code)
claude doctor       # 只读诊断：安装健康度、配置校验错误
```

## 首次登录授权

```bash
claude
```

首次运行会拉起浏览器完成授权。若已设置 `ANTHROPIC_API_KEY` 环境变量，则改为提示确认该密钥。

## 升级

原生安装会后台自动更新，手动触发：

```bash
claude update
```

Homebrew / npm 安装需手动升级：

```bash
brew upgrade claude-code
npm install -g @anthropic-ai/claude-code@latest
```

> npm 升级别用 `npm update -g` —— 它遵循原始安装的 semver 范围，可能根本不动。

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
