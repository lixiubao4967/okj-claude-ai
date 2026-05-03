# MCP 安装记录

记录在本机为 Claude Code 安装的 MCP 服务器，包含安装命令、关键决策与踩坑点。

> 来源：自编

---

## 已安装清单

| MCP | 用途 | 安装日期 | 作用域 |
|-----|------|----------|--------|
| context7 | 实时拉取最新库 / SDK 文档 | 2026-05-04 | user |
| github | 读取 GitHub PR / Issue / 代码（只读）| 2026-05-04 | user |

---

## 前提条件

| 条件 | 说明 |
|------|------|
| Claude Code | 已安装并登录 |
| Node.js | `npx` 可用（Context7 通过 npx 启动）|
| GitHub 账号 | GitHub MCP 走 OAuth，需在浏览器中授权 |

---

## 作用域选择

| 作用域 | 标志 | 适用场景 |
|--------|------|----------|
| user | `-s user` | 个人工具，跨所有项目通用（本次两个 MCP 都选这个）|
| project | `-s project` | 写入仓库 `.mcp.json`，与协作者共享 |
| local | 默认 | 仅当前用户 + 当前项目 |

> 个人用的查文档 / 管 PR 类 MCP **不应**写进仓库给协作者看到，因此走 `-s user`。

---

## Context7 MCP（无需认证）

实时拉取最新版的库文档与代码示例，避免凭印象写错 API。

### 安装

```bash
claude mcp add context7 -s user -- npx -y @upstash/context7-mcp
```

### 验证

```bash
claude mcp list
# 期望输出：context7: npx -y @upstash/context7-mcp - ✓ Connected
```

### 使用示例

在对话中明确点名工具，效果最稳：

```
用 context7 查一下 Anthropic SDK 最新的 prompt caching 用法
```

---

## GitHub MCP（远程托管 + 只读）

让 Claude 直接查 GitHub 的 PR / Issue / 代码，省掉手动跑 `gh` 命令。

### 关键决策

> **不要用 `@modelcontextprotocol/server-github`。**
> 这是一些旧文章里推荐的 npm 包，已被 GitHub 官方接管并归档，不再维护。改用 GitHub 官方提供的远程 MCP endpoint。

> **走远程托管，不用本地 Docker。**
> 远程 endpoint 一次 OAuth 授权即可，无需本机跑容器。

> **走 `/readonly` endpoint，不走默认全功能 endpoint。**
> 只读模式在 URL 路径层做隔离（`/mcp/readonly` 与 `/mcp/`），即使 OAuth token 有写权限，该 endpoint 也不会暴露写工具——比"在客户端过滤工具"更可靠。

### 安装

```bash
claude mcp add --transport http github -s user https://api.githubcopilot.com/mcp/readonly
```

### OAuth 授权（必须做一次）

安装完后必须在 Claude Code 内完成 OAuth，否则连不上：

1. 在 Claude Code 输入 `/mcp` 打开 MCP 管理面板
2. 找到 `github` 项，按提示触发 OAuth
3. 浏览器跳到 GitHub 授权页 → 同意 → 自动回到 Claude Code

> 未授权时 `claude mcp list` 会显示 `github: ... - ✗ Failed to connect`，这是预期状态，不是装坏了。OAuth 完成后变 ✓。

### 远程 MCP 认证特性

> OAuth 状态**按用户存**，不按项目。一次授权后所有项目都能直接用，不用每个仓库重新登录。

### 验证

```bash
claude mcp list
# 期望（OAuth 后）：github: https://api.githubcopilot.com/mcp/readonly (HTTP) - ✓ Connected
```

### 使用示例

```
用 github mcp 看一下 anthropics/claude-code 仓库最近的 5 个 issue
```

调用成功时 tool call 名称会形如 `mcp__github__*`，可据此判断是否真的走了 MCP。

---

## 升级

| MCP | 升级方式 |
|-----|----------|
| context7 | `npx -y` 每次启动都拉最新，无需手动升级 |
| github | 远程托管，server 端由 GitHub 自动维护 |

> 远程 MCP 的好处：不用关心版本升级，服务端发版即生效。

---

## 卸载

```bash
claude mcp remove context7 -s user
claude mcp remove github -s user
```
