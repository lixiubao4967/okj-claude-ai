# CLAUDE.md

## 仓库性质

文档仓库，记录 Claude Code 的使用方法与实践经验。无可执行代码、无构建系统、无测试框架。

## 目录结构

```
├── README.md              ← 索引页（导航入口）
├── setup/                 ← 安装与初始配置
├── usage/                 ← 命令与快捷键
├── channels/              ← 远程控制（Telegram / Discord）
├── plugins/               ← 插件指南
├── tools/                 ← 工具（gstack / Obsidian 等）
├── quickstart/            ← 快速入门
├── workflow/              ← 工作流与方法论
├── internals/             ← 内部原理与源码分析
└── .claude/
    ├── rules/             ← 自动应用的上下文规则（doc-standards.md）
    ├── skills/            ← 项目专属 skill（add-learning-doc）
    └── hookify.*.local.md ← 事件触发规则
```

新文档必须放入对应分类目录。如果不属于现有分类，先建新目录再写文档。

## 文档语言

- 技术文档用**中文**撰写
- CLI 命令、配置片段保持原文（英文）
- 命令用 `代码块` 包裹

## Commit 规范

- 统一使用 `docs:` 前缀，例如：`docs: 添加 gstack 安装指南`

## 内容来源标注

| 标注 | 含义 |
|------|------|
| 自编 | 本仓库维护者原创 |
| 官方 | 翻译或搬运自 Anthropic 官方文档 |
| 第三方文章解读 | 基于第三方博客整理，非 Anthropic 官方出品 |

## 禁止操作

- 不要删除或重命名已有文档（除非明确要求）
- 不要把技术细节写进 CLAUDE.md，那些属于文档本身
