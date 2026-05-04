# LLM Wiki 知识库方法论

> 来源：第三方文章解读 · 《我用 Obsidian + Codex 搭了一个会持续进化的 AI 知识库》 · 作者：未知 · ingest 自 raw/2026-05-04-llm-wiki-obsidian.md

## 概述

把知识库从"资料仓库"升级成"会持续编译的系统"的方法论。核心思路是**分层 + 编译**：原始素材（raw）和整理后的知识（wiki）必须分开，由 Agent 负责把前者持续编译成后者。

本仓库已经在实践这套方法：`raw/` 是原始素材层，八个分类目录是 wiki 层，`CLAUDE.md` + `.claude/skills/ingest-raw/` 是编译规则。

---

## 核心概念

### 编译知识 vs 存储资料

| 模式 | 做法 | 结果 |
|------|------|------|
| 资料仓库（多数人） | 收藏文章 → 写摘要 → 扔进文件夹 → 用时翻 | 越存越多，找不到 |
| LLM Wiki | 资料进 raw → Agent 整理 → 更新概念页/主题页/索引 → 站在 wiki 层提问 | 越用越好用 |

差别不在能不能查，而在**知识会不会累积**。

### 与传统 RAG 的差别

- **RAG**：文件存进去 → 切片检索 → 现场拼答案
- **LLM Wiki**：raw 进 → Agent 编译 → wiki 沉淀稳定中间层 → 提问优先查 wiki

两者都能回答问题，差别在你的知识会不会越用越好用。

---

## 实施模式

### 双层目录

| 目录 | 角色 |
|------|------|
| `raw/` | 原始素材（一手输入） |
| 分类目录 / `wiki/` | 编译产物（结构化知识页） |

> 一上来就把 raw 和 wiki 混着放是最常见的翻车点。原始资料和整理后的知识本来就不是同一层东西。

### Agent 规则文件

仓库根目录放一份给 Agent 看的规则文件（本仓库用 `CLAUDE.md`，原文称 `AGENTS.md`），至少明确：

- 目录约定
- ingest 时该做什么（提炼观点、更新相关页、加交叉链接）
- 查询时该优先看哪一层

### Ingest 工作流

新资料进 raw 后，让 Agent 执行：

1. 提炼关键观点
2. 更新相关 wiki 页面
3. 必要时新建概念页 / 主题页
4. 添加交叉链接

> **关键**：不只是生成摘要，必须更新知识网络。如果 Agent 只会"总结一篇文章"，那只是个自动摘要器，不是知识系统。

### 基于 wiki 的查询 + 回写

提问时让 Agent 先查 wiki 层，必要时再回看 raw。如果得到了高质量答案，**继续让 Agent 把答案沉淀回 wiki** —— 这一步让知识库不只吸收资料，也吸收问题。

---

## 容易翻车的地方

1. **过度设计**：还没导入第一篇就花 2 小时设计 schema。先跑通最小闭环再优化。
2. **混层**：raw 和 wiki 混放，查询/更新/维护全乱。
3. **只摘要不组网**：Agent 只会总结单篇文章，不更新已有页、不加交叉链接，得到的只是自动摘要器。
4. **只 ingest 不查询不回写**：不查询就看不到 wiki 值不值得建；不回写高质量问答就不会复利。

---

## 本仓库的对应实现

| 方法论概念 | 本仓库实现 |
|-----------|------------|
| raw/ 原始素材层 | `raw/`（进 git，ingest 后删除，依赖 git 历史溯源）|
| wiki/ 编译产物层 | `setup/ usage/ channels/ plugins/ tools/ quickstart/ workflow/ internals/` 八个分类目录 |
| AGENTS.md 规则文件 | `CLAUDE.md`（目录约定、来源标注、commit 规范）|
| Ingest 工作流 | `.claude/skills/ingest-raw/SKILL.md`（合并优先、单一归属等决策已固化）|
| 主题驱动新建 | `.claude/skills/add-learning-doc/`（输入是主题或对话，与 ingest-raw 互补）|
| 文档格式约束 | `.claude/rules/doc-standards.md`（自动应用到所有 `**/*.md`）|

---

## 最小动手清单（在新仓库复刻）

1. 新建仓库，创建 `raw/` 和 wiki 层分类目录
2. 根目录写 `CLAUDE.md`（或 `AGENTS.md`），定义目录约定 + ingest 规则
3. 往 `raw/` 塞一篇文章
4. 让 Claude 执行第一次 ingest
5. 检查产出：来源标注、关键概念页、交叉链接是否齐全
6. 基于 wiki 层提问，验证 Agent 优先引用编译产物
7. 把好的问答继续沉淀回 wiki

---

## 相关

- [Obsidian 集成指南](../tools/obsidian-setup.md) —— 把本仓库当 vault 打开后，可以用 Obsidian 检索 wiki 层
- [Vibe Coding 指南](./vibe-coding-guide.md) —— 编码场景的对应方法论（memory-bank 机制）
