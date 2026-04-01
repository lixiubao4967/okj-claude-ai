# Claude Code Channels（手机远程控制）

通过 Telegram 或 Discord 远程给 Claude Code 发任务，Claude 在后台执行完成后再通知你。

## 前提条件

| 条件 | 说明 |
|------|------|
| Claude Code 版本 | v2.1.80 或更高（`claude --version` 查看） |
| 登录方式 | 必须使用 **claude.ai 账号登录**，不支持 API Key 认证 |
| 运行环境 | 本地需安装 **Bun**（`curl -fsSL https://bun.sh/install \| bash`） |
| 功能状态 | 目前处于 **Research Preview** 阶段 |

> **Team / Enterprise 用户额外步骤**：Channels 默认关闭，需要组织管理员先开启：
> `claude.ai → Admin settings → Claude Code → Channels → 开启`
> 或在 managed settings 中设置 `channelsEnabled: true`。

## Telegram 配置步骤

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

## Discord 配置步骤

```bash
/discord:configure
```

流程与 Telegram 类似，配置完成后向 Bot 发送任意消息完成配对。

## 工作原理

Channels 基于 **MCP（Model Context Protocol）** 实现，是一个 MCP Server，将外部消息推送进正在运行的 Claude Code 会话。**Session 必须保持开启**（建议跑在后台终端或 tmux 中），消息才能实时送达。

## 安全说明

每个 Channel 维护一份**发送者白名单**，只有配对过的 ID 能推送消息，其他人的消息会被静默丢弃。
