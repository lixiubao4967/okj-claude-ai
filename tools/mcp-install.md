# MCP 安装记录

记录在本机为 Claude Code 安装的 MCP 服务器，包含安装命令、关键决策与踩坑点。

> 来源：自编

---

## 已安装清单

| MCP | 用途 | 安装日期 | 作用域 |
|-----|------|----------|--------|
| context7 | 实时拉取最新库 / SDK 文档 | 2026-05-04 | user |

## 尝试过但未采用

| MCP | 尝试日期 | 放弃原因 |
|-----|----------|----------|
| GitHub MCP（远程托管 + 只读）| 2026-05-04 | OAuth 协议不兼容；PAT 备选方案配置成本超过收益（详见下方"GitHub MCP：尝试与放弃记录"） |

---

## 前提条件

| 条件 | 说明 |
|------|------|
| Claude Code | 已安装并登录 |
| Node.js | `npx` 可用（Context7 通过 npx 启动）|

---

## 作用域选择

| 作用域 | 标志 | 适用场景 |
|--------|------|----------|
| user | `-s user` | 个人工具，跨所有项目通用（本次选这个）|
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

### 升级

`npx -y` 每次启动都会拉最新，无需手动升级。

### 卸载

```bash
claude mcp remove context7 -s user
```

---

## GitHub MCP：尝试与放弃记录

> **结论：放弃 MCP 路线，继续用 `gh` CLI。**
> 本节记录失败原因与权衡过程，避免将来再尝试同样路径。

### 当时的安装命令

```bash
claude mcp add --transport http github -s user https://api.githubcopilot.com/mcp/readonly
```

`/readonly` 在路径层做了写权限隔离，方向是对的。

> **不要用 `@modelcontextprotocol/server-github`。** 一些旧文章推荐的这个 npm 包，已被 GitHub 官方接管并归档，不再维护。

### 失败点：OAuth 不支持 RFC 7591 动态客户端注册

在 Claude Code 内 `/mcp` → Authenticate 时报错：

```
SDK auth failed: Incompatible auth server: does not support dynamic client registration
```

**原因：** Claude Code 的 OAuth 客户端依赖 RFC 7591「Dynamic Client Registration」自动向 OAuth 服务器注册。GitHub 出于安全考虑只接受**预先在后台注册**的 OAuth App——双方对不上，OAuth 流程走不通。

### 备选方案：PAT + Header 认证（可行但繁琐）

绕开 OAuth，改用 fine-grained PAT 通过 header 注入：

```bash
claude mcp add --transport http github -s user \
  --header "Authorization: Bearer github_pat_xxx" \
  https://api.githubcopilot.com/mcp/readonly
```

**为什么没选这条：**

- 要在 GitHub 后台手动配置 fine-grained PAT：指定仓库 + 勾选 Read-only 权限（Contents / Issues / Pull requests / Metadata）
- PAT 有过期时间，需要周期性轮换
- PAT 明文写入 `~/.claude.json`，没有加密存储
- 文档维护场景下，MCP 的能力跟 `gh pr list` / `gh issue view` 等命令高度重合

### 决策依据

文档维护工作流里 GitHub 操作以"查"为主，`gh` CLI 通过 Bash 调用已经够用。MCP 的增益（结构化工具调用、工具发现）不足以抵消 PAT 维护成本。

> ★ 经验：选 MCP 别只看"功能炫"，要评估它**是否真正替代或叠加在已有工具流之上**。能力高度重叠的工具最好只保留一个，避免决策与维护负担。

### 如果以后改主意

直接回顾上面"备选方案：PAT + Header 认证"那段的命令即可。
