# Anthropic 官方 Claude Code Plugin 指南

> 原文来源：[Anthropic公式のClaude Code Pluginが増えてたので、改めて眺める（2026年3月）](https://dev.classmethod.jp/articles/claude-plugins-official-view-2026-03/)
>
> 作者：藤井元貴 | 发布于 DevelopersIO（Classmethod）| 2026.03.12

---

## 文章摘要

作者发现 Anthropic 官方的 Claude Code Plugin 数量持续增长，于是重新整理了现有的官方插件列表，并建议开发者**定期关注官方插件生态**。

## 插件分类总览

| 类别 | 插件 | 说明 |
|------|------|------|
| **UI/前端** | `frontend-design` | 生成独特的、生产级质量的界面，避免"AI味"的通用美学 |
| | `playground` | 可视化配置的 HTML 游乐场 |
| **开发流程** | `agent-sdk-dev` | Claude Agent SDK 开发工具包 |
| | `commit-commands` | Git 工作流自动化（commit、push、PR） |
| | `feature-dev` | 结构化功能开发（7 阶段流程） |
| | `ralph-loop` | 迭代式自引用循环 |
| **代码审查/质量** | `code-review` | 自动 PR 代码审查（5 个并行 Sonnet Agent） |
| | `code-simplifier` | 代码简化 |
| | `pr-review-toolkit` | 全面的 PR 分析（6 个专业 Agent） |
| | `security-guidance` | 安全问题实时监控（XSS、注入等 9 种模式） |
| **Claude Code 配置管理** | `claude-code-setup` | 分析代码库，自动推荐 Hooks/Skills/MCP 配置 |
| | `claude-md-management` | 维护 CLAUDE.md，补充缺失上下文 |
| | `hookify` | 自定义 Hook 创建 |
| **输出风格** | `learning-output-style` | 交互式学习模式，鼓励用户亲自编写代码 |
| | `explanatory-output-style` | 教育型输出，解释实现背后的选择 |
| **语言服务器 (LSP)** | 12 种语言 | Python、TypeScript、Go、Java、Rust、C/C++、C#、PHP、Ruby、Kotlin、Lua、Swift |

---

## 前置条件

- 已安装 Claude Code 并完成认证
- 版本 ≥ 1.0.33（运行 `claude --version` 检查）
- 如需升级：
  - Homebrew: `brew upgrade claude-code`
  - npm: `npm update -g @anthropic-ai/claude-code`

---

## 插件基础操作

### 浏览与安装

```bash
# 在 Claude Code 中打开插件管理器
/plugin

# ⚠️ 首次使用需先添加官方插件市场（只需执行一次）
/plugin marketplace add anthropics/claude-plugins-official

# 从官方市场安装插件
/plugin install <plugin-name>@claude-plugins-official

# 安装后重新加载插件
/reload-plugins
```

> **实测说明：** 官方市场**不会**自动可用，必须手动添加 `anthropics/claude-plugins-official` 仓库。
> 原文提到的 `anthropics/claude-code` 是 Claude Code 源码仓库，仅包含部分插件；
> 完整的官方插件目录在 `anthropics/claude-plugins-official`（含 32 个内部插件 + 16 个外部插件）。

### 管理插件

```bash
# 禁用插件（不卸载）
/plugin disable <plugin-name>@<marketplace-name>

# 重新启用
/plugin enable <plugin-name>@<marketplace-name>

# 完全卸载
/plugin uninstall <plugin-name>@<marketplace-name>
```

### 安装范围

| 范围 | 说明 |
|------|------|
| **User** | 为自己安装，全项目生效（默认） |
| **Project** | 为仓库所有协作者安装 |
| **Local** | 仅自己在当前仓库使用 |

---

## 推荐插件详解

以下是作者重点推荐的几个插件，附详细安装步骤和使用方法。

### 1. Claude Code Setup — 项目配置自动推荐

> 自动分析代码库，推荐适合的 Hooks、Skills、MCP 集成配置。

**安装：**

```bash
/plugin install claude-code-setup@claude-plugins-official
/reload-plugins
```

**使用方法：**

- 安装后在项目目录中启动 Claude Code，插件会自动分析当前代码库
- 根据项目技术栈和结构，推荐合适的配置方案
- 帮助新项目快速建立最佳实践的 Claude Code 配置

---

### 2. CLAUDE.md Management — 上下文文件维护

> 自动维护 `CLAUDE.md` 文件，回顾会话内容并补充缺失的上下文信息。

**安装：**

```bash
/plugin install claude-md-management@claude-plugins-official
/reload-plugins
```

**使用方法：**

- 插件会在会话过程中追踪哪些上下文信息是缺失的
- 自动建议更新 `CLAUDE.md`，确保项目指引保持完整和最新
- 适合团队协作场景，让新成员快速了解项目上下文

---

### 3. Learning Output Style — 交互式学习模式

> 当判断"这段代码值得人类亲自编写"时，会提示用户自己动手，培养编程能力。

**安装：**

```bash
/plugin install learning-output-style@claude-plugins-official
/reload-plugins
```

**使用方法：**

- 安装后通过 `SessionStart` Hook 自动激活，无需手动调用
- 在关键决策点，插件会要求用户编写 5-10 行代码
- 适合学习场景，帮助用户在 AI 辅助下仍能提升编码能力

---

### 4. Hookify — 自定义 Hook 创建

> 帮助创建和管理 Claude Code 的自定义 Hook。

**安装：**

```bash
/plugin install hookify@claude-plugins-official
/reload-plugins
```

**使用方法：**

```bash
# 交互式创建新 Hook
/hookify:hookify

# 查看已有 Hook 列表
/hookify:list
```

- 插件包含模式分析 Agent，能根据使用习惯推荐 Hook
- 支持 `PreToolUse`、`PostToolUse`、`SessionStart` 等事件类型

---

### 5. Security Guidance — 安全指导

> 实时监控代码安全问题，覆盖 XSS、注入、eval 等 9 种常见安全模式。

**安装：**

```bash
/plugin install security-guidance@claude-plugins-official
/reload-plugins
```

**使用方法：**

- 通过 `PreToolUse` Hook 自动生效，在 Claude 执行工具前进行安全检查
- 当检测到潜在安全风险时，会主动发出警告
- 无需手动调用，安装即生效

---

### 6. Code Review — 自动代码审查

> 使用 5 个并行 Sonnet Agent 进行全方位 PR 代码审查。

**安装：**

```bash
/plugin install code-review@claude-plugins-official
/reload-plugins
```

**使用方法：**

```bash
# 对当前变更进行代码审查
/code-review:code-review
```

- 5 个 Agent 分别负责：合规性、Bug 检测、上下文分析、历史对比、注释质量
- 并行执行，效率高

---

### 7. PR Review Toolkit — PR 全面分析

> 6 个专业 Agent 协同完成 PR 审查。

**安装：**

```bash
/plugin install pr-review-toolkit@claude-plugins-official
/reload-plugins
```

**使用方法：**

```bash
# 审查指定 PR
/pr-review-toolkit:review-pr
```

---

## LSP 语言服务器插件

LSP 插件为 Claude 提供实时代码智能（跳转定义、查找引用、类型检查等）。

### 安装示例（Go、Python、TypeScript、Rust）

```bash
# 1. 确保系统已安装对应的语言服务器二进制文件
npm install -g typescript-language-server typescript  # TypeScript
brew install pipx && pipx install pyright             # Python（macOS）
go install golang.org/x/tools/gopls@latest            # Go
brew install rust-analyzer                             # Rust（macOS，或用 rustup component add rust-analyzer）

# 2. 在 Claude Code 中安装 LSP 插件
/plugin install typescript-lsp@claude-plugins-official
/plugin install pyright-lsp@claude-plugins-official
/plugin install gopls-lsp@claude-plugins-official
/plugin install rust-analyzer-lsp@claude-plugins-official
/reload-plugins
```

### 支持的语言与所需二进制

| 语言 | 插件名 | 所需二进制 |
|------|--------|-----------|
| Python | `pyright-lsp` | `pyright-langserver` |
| TypeScript | `typescript-lsp` | `typescript-language-server` |
| Go | `gopls-lsp` | `gopls` |
| Java | `jdtls-lsp` | `jdtls` |
| Rust | `rust-analyzer-lsp` | `rust-analyzer` |
| C/C++ | `clangd-lsp` | `clangd` |
| C# | `csharp-lsp` | `csharp-ls` |
| PHP | `php-lsp` | `intelephense` |
| Ruby | — | — |
| Kotlin | `kotlin-lsp` | `kotlin-language-server` |
| Lua | `lua-lsp` | `lua-language-server` |
| Swift | `swift-lsp` | `sourcekit-lsp` |

### LSP 插件带来的能力

- **自动诊断**：每次文件编辑后，语言服务器自动报告错误和警告，Claude 能即时修复
- **代码导航**：跳转定义、查找引用、获取类型信息、列出符号、追踪调用链

---

## 常见问题

**Q: `/plugin` 命令无法识别？**
确认 Claude Code 版本 ≥ 1.0.33，升级后重启终端。

**Q: 插件安装后 Skill 没有出现？**
清除缓存后重装：
```bash
rm -rf ~/.claude/plugins/cache
# 重启 Claude Code，然后重新安装插件
```

**Q: LSP 报 "Executable not found in $PATH"？**
确保对应语言服务器的二进制文件已安装，并在 `$PATH` 中可用。

**Q: 语言服务器内存占用过高？**
`rust-analyzer` 和 `pyright` 在大型项目中可能消耗较多内存，可用 `/plugin disable <name>` 临时禁用。

---

## 参考链接

- [官方插件文档](https://code.claude.com/docs/en/plugins)
- [发现与安装插件](https://code.claude.com/docs/en/discover-plugins)
- [在线浏览官方插件](https://claude.com/plugins)
- [插件提交入口 (Claude.ai)](https://claude.ai/settings/plugins/submit)
- [插件提交入口 (Console)](https://platform.claude.com/plugins/submit)
