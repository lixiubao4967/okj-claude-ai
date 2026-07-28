# 《深入理解 AI Agent：设计原理与工程实践》学习资源

> 来源：[bojieli/ai-agent-book](https://github.com/bojieli/ai-agent-book) · 作者：李博杰 · 许可：Apache 2.0
> 标注：第三方文章解读（开源书籍，非 Anthropic 官方出品）

## 1. 是什么？

一本开源的中文 AI Agent 教材，讲的是 **Agent 的底层原理与工程实践**，而不是某个框架的用法。全书 10 章 + 92 个配套项目（70+ 可独立运行），有 PDF/EPUB 和在线版，附 8 种语言的社区翻译。

核心公式贯穿全书：

```
Agent = LLM + 上下文（Context） + 工具（Tools）
```

这三者对应全书的三大支柱：模型能力、上下文工程、工具设计。**书的价值在于把"为什么这样设计"讲清楚**——KV Cache 为什么决定了 prompt 的拼接顺序、上下文压缩为什么会掉能力、工具调用为什么最终要内化进模型，这些正是用 Claude Code / Skill / MCP 时经常撞到但文档不会解释的部分。

访问入口：

| 形式 | 地址 |
|------|------|
| 在线阅读（支持搜索、切换语言） | https://bojieli.github.io/ai-agent-book/ |
| PDF / EPUB 下载 | https://github.com/bojieli/ai-agent-book/releases |
| 源码与配套项目 | https://github.com/bojieli/ai-agent-book |

---

## 2. 内容地图（10 章）

| 章节 | 主题 | 核心内容 |
|------|------|----------|
| 1 | Agent 基础知识 | Agent 的定义、新范式、架构与工程竞争力 |
| 2 | 上下文工程 | **KV Cache**、提示工程、技能（Skill）设计、上下文压缩 |
| 3 | 用户记忆和知识库 | 跨会话记忆、RAG、索引、知识图谱 |
| 4 | 工具 | **MCP 协议**、感知/执行/协作类工具、异步 Agent |
| 5 | Coding Agent 与代码生成 | 生产级代码生成系统的设计 |
| 6 | Agent 评估 | 评估环境、指标体系、统计显著性 |
| 7 | 模型后训练 | SFT 与 RL 的权衡、把工具调用内化进模型 |
| 8 | Agent 持续进化 | 从运行轨迹（trace）学习并迭代 |
| 9 | 多模态与实时交互 | 语音、GUI、Computer Use、机器人 |
| 10 | 多 Agent 协作 | 协作框架、群体智能涌现 |

正文源文件在仓库 `book/` 目录下，一章一个 markdown（`book/chapter1.md` … `book/chapter10.md`），另有 `introduction.md`、`afterword.md`、`reference-answers.md`（习题参考答案）。

**与本仓库的关联**：第 2 章（上下文工程 / Skill 设计）、第 4 章（MCP、工具设计）、第 5 章（Coding Agent）与 [Claude Code 源码深度架构分析](../internals/claude-code-architecture.md)、[MCP 安装记录](../tools/mcp-install.md)、[OKG 内部 Claude 快速入门](../quickstart/okg-claude-quickstart.md) 直接对应——前者讲原理，后者是落地。

---

## 3. 难度分级与阅读顺序

官方学习路径见 `docs/zh-CN/LEARNING.md`，按 5 级划分：

| 级别 | 章节 | 前置要求 |
|------|------|----------|
| 🟢 入门级 | 第 1–2 章 | 无特殊要求 |
| 🔵 进阶级 | 第 3–4 章 | 基础编程能力，涉及系统集成 |
| 🟣 高级 | 第 5–6 章 | 较强编程能力、复杂系统设计经验 |
| 🔴 专家级 | 第 7–8 章 | 深度学习与模型训练背景 |
| 🟠 应用级 | 第 9–10 章 | 综合应用，构建实际项目 |

推荐顺序按"模型 + 上下文 + 工具"递进：基础篇 → 上下文篇（系统提示、记忆、检索增强）→ 工具篇（感知/执行/协作工具）→ 评估与进化篇 → 拓展与协作篇。

> 作者强调**以动手实践为核心**：每个项目都能独立运行，建议从简单项目开始逐步深入，不要只读不跑。

---

## 4. 跑配套代码

```bash
git clone https://github.com/bojieli/ai-agent-book.git
cd ai-agent-book
```

配套项目按章节放在 `chapter1/` … `chapter10/`，各章 README 标注了项目类型：✅ 可运行 / 📖 复现 / 🚧 仅设计。主语言是 Python，Web 示例用 JavaScript。

**需要 API Key**：作者推荐 Kimi、智谱 GLM、DeepSeek、OpenRouter 等国内可直连的平台。若要跑 Claude 相关示例，用 Anthropic API Key（最新模型见 `claude-opus-5` / `claude-sonnet-5` 系列）。

> **踩坑提示**：第 6、7、9、10 章依赖的评测基准和训练框架**不在本仓库内**，需要手动另行 clone，包括 android_world、GAIA、OSWorld、SWE-bench（评测），minimind、AdaptThink、verl（训练），browser-use、claude-quickstarts（交互），generative_agents（斯坦福 AI 小镇，多 Agent）。这些仓库体积和依赖都不小，先看章节 README 再决定要不要装。

PDF 自建需要 Pandoc + xelatex + ElegantBook 模板（`book/build_pdf.sh`），一般直接下 Release 里的 PDF 更省事。

---

## 5. 怎么用这本书

- **只想搞懂 Claude Code 为什么这么设计**：读第 2 章（上下文工程）+ 第 4 章（工具/MCP）+ 第 5 章（Coding Agent），够了。
- **要做内部 Agent/Skill 落地**：加读第 3 章（记忆与知识库）和第 6 章（评估）——评估这一章最容易被跳过，但决定了 Agent 上线后能不能持续改进。
- **不碰模型训练就跳过第 7–8 章**：这两章需要深度学习背景，属于"知道存在即可"的范畴。
