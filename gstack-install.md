# gstack 安装指南

gstack 是一个开源工具包，由 Garry Tan（YC 总裁）创建，能将 Claude Code 变成一个结构化的虚拟工程团队。提供 15 个专业 Skill + 6 个工具，覆盖从需求到发版的完整开发流水线。

GitHub：https://github.com/garrytan/gstack

---

## 前提条件

| 条件 | 说明 |
|------|------|
| Claude Code | 已安装并登录 |
| Git | 已安装 |
| Bun v1.0+ | `bun --version` 检查，未安装则运行 `curl -fsSL https://bun.sh/install \| bash` |

---

## 安装步骤

### 1. 全局安装（推荐）

```bash
git clone https://github.com/garrytan/gstack.git ~/.claude/skills/gstack && cd ~/.claude/skills/gstack && ./setup
```

### 2. 项目级安装（可选，让团队成员共享）

```bash
cp -Rf ~/.claude/skills/gstack .claude/skills/gstack && rm -rf .claude/skills/gstack/.git && cd .claude/skills/gstack && ./setup
```

### 3. 多 Agent 支持（Codex / Gemini CLI / Cursor）

```bash
git clone https://github.com/garrytan/gstack.git ~/gstack
cd ~/gstack && ./setup --host auto
```

---

## 安装后配置

在项目 `CLAUDE.md` 中添加：

```
使用 gstack 的 /browse skill 进行所有网页浏览，不要使用 mcp__claude-in-chrome__* 工具。
```

---

## 核心开发流水线

```
描述需求 → /plan-ceo-review → /plan-eng-review → 退出 plan 模式写代码 → /review → /ship → /qa → /retro
```

| 阶段 | 命令 | 作用 |
|------|------|------|
| 需求梳理 | `/office-hours` | 在写代码前重新梳理产品想法 |
| 产品评审 | `/plan-ceo-review` | 压力测试产品方向和范围 |
| 技术评审 | `/plan-eng-review` | 敲定架构和技术方案 |
| 设计评审 | `/plan-design-review` | 设计质量审查 |
| 设计咨询 | `/design-consultation` | 构建完整设计系统 |
| 代码审查 | `/review` | 偏执级代码审查，找生产环境 Bug |
| 调试 | `/investigate` | 系统化调试方法论 |
| 测试 | `/qa` | 测试 + 修 Bug + 回归测试 |
| 仅测试 | `/qa-only` | 只报告测试结果，不改代码 |
| 发版 | `/ship` | 发布自动化 + 创建 PR |
| 文档 | `/document-release` | 自动同步文档与已发布代码 |
| 回顾 | `/retro` | 每周团队回顾 |
| 浏览器 | `/browse` | 真实 Chromium 浏览器测试 |

---

## 安全工具

| 命令 | 作用 |
|------|------|
| `/careful` | 破坏性命令警告 |
| `/freeze` | 限制只能编辑指定目录 |
| `/guard` | `/careful` + `/freeze` 组合 |
| `/unfreeze` | 解除编辑限制 |

---

## 升级

```bash
/gstack-upgrade
```

gstack 支持自更新，直接在 Claude Code 中运行即可。

---

## 许可证

MIT 开源协议，免费使用，无付费版。
