# Claude Code Channels（手机远程控制）

通过 Telegram 或 Discord 远程给 Claude Code 发任务，Claude 在后台执行完成后再通知你。

## 前提条件

| 条件 | 说明 |
|------|------|
| Claude Code 版本 | v2.1.80 或更高（`claude --version` 查看） |
| 登录方式 | 需使用 **claude.ai 账号登录**，或 Console API key；不支持 platform.claude.com 的纯 API-only 认证 |
| 运行环境 | 本地需安装 **Bun**（`curl -fsSL https://bun.sh/install \| bash`） |
| 功能状态 | 目前处于 **Research Preview** 阶段 |
| 不支持环境 | Amazon Bedrock、Google Cloud Agent Platform、Microsoft Foundry |

> **Team / Enterprise 用户额外步骤**：Channels 默认关闭，需要组织管理员先开启：
> `claude.ai → Admin settings → Claude Code → Channels → 开启`
> 或在 managed settings 中设置 `channelsEnabled: true`。

## Telegram 配置步骤

**第一步：创建 Telegram Bot**
1. 在 Telegram 中搜索 `@BotFather` 并打开
2. 发送 `/newbot`，按提示填写 Bot 名称，获取 **Bot Token**

**第二步：安装 Channels 插件**
```bash
/plugin install telegram@claude-plugins-official
/reload-plugins
```

**第三步：用 `--channels` 重启 Claude Code**
```bash
claude --channels plugin:telegram@claude-plugins-official
```
> Channels 功能默认不激活，必须带 `--channels` 参数重启会话才能生效。

**第四步：配置 Bot Token**
```bash
/telegram:configure <token>
```
Token 直接作为参数传入（不是交互式输入）。配置成功后，Bot 会在 Telegram 端生成一个配对码。

**第五步：完成配对**

在 Telegram 中向 Bot 发送任意消息，获取其回复的配对码，然后回到 Claude Code 执行：
```bash
/telegram:access pair <code>
```
完成后该 Telegram 账号会被加入白名单。

**第六步：开始使用**

直接在 Telegram 向 Bot 发送编程任务，Claude Code 后台执行，完成后回复通知你。

## Discord 配置步骤

```bash
/plugin install discord@claude-plugins-official
/reload-plugins
claude --channels plugin:discord@claude-plugins-official
/discord:configure <token>
/discord:access pair <code>
```

流程与 Telegram 一致：先装插件、带 `--channels` 重启、传入 Token 配置、再用配对码完成绑定。

> 来源：https://code.claude.com/docs/en/channels.md

## 工作原理

Channels 基于 **MCP（Model Context Protocol）** 实现，是一个 MCP Server，将外部消息推送进正在运行的 Claude Code 会话。**Session 必须保持开启**（建议跑在后台终端或 tmux 中），消息才能实时送达。

## 安全说明

每个 Channel 维护一份**发送者白名单**，只有配对过的 ID 能推送消息，其他人的消息会被静默丢弃。
