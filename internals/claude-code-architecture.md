# Claude Code 源码深度架构分析

> 基于 `@anthropic-ai/claude-code` v2.1.88 源码（51 万行 TypeScript，1902 个文件）

---

## 一、项目全景与设计哲学

### 1.1 代码规模

| 模块 | 行数 | 占比 | 核心职责 |
|------|------|------|---------|
| `utils` | 180,472 | 35.2% | 权限、bash 安全、消息处理、git、MCP 等基础设施 |
| `components` | 81,546 | 15.9% | React 终端 UI 组件（权限对话框、diff、消息渲染） |
| `services` | 53,680 | 10.5% | API 调用、压缩、MCP 客户端、分析、OAuth |
| `tools` | 50,828 | 9.9% | 40+ 工具实现（Bash、FileEdit、Agent、MCP 等） |
| `commands` | 26,428 | 5.2% | 90+ 斜杠命令（/compact、/model、/mcp 等） |
| `ink` | 19,842 | 3.9% | 自研 Ink Fork（React 终端渲染引擎） |
| `hooks` | 19,204 | 3.7% | React hooks（权限处理、IDE 集成、语音等） |
| `bridge` | 12,613 | 2.5% | 远程控制（本地机器作为 bridge 环境） |
| `cli` | 12,353 | 2.4% | CLI 参数解析、后台会话管理 |

### 1.2 五条设计原则

通读 51 万行代码后提炼的贯穿全局的设计原则：

1. **工具即能力边界**：agent 能做什么完全由工具集决定，没有后门。读文件要用 FileReadTool，写文件要用 FileEditTool，执行命令要用 BashTool。新增能力 = 新增工具。
2. **Fail-closed 安全默认**：所有安全相关的默认值都是最保守的——工具默认不可并行（`isConcurrencySafe: false`）、默认非只读（`isReadOnly: false`）、权限默认需要确认。
3. **Context Engineering > Prompt Engineering**：不是写一段 prompt 告诉模型"你是谁"，而是在每轮对话中精心组装完整的上下文环境——分段缓存、动态注入、多层压缩。
4. **可组合性**：子 agent 复用主 agent 的 `query()` 函数，MCP 工具复用内部权限检查，Team 复用 Subagent 的执行引擎。
5. **编译时消除 > 运行时判断**：通过 Bun 的 `feature()` 宏在构建时移除未启用的功能代码，未启用的功能在 bundle 中完全不存在。

---

## 二、Agent Loop：系统的心脏

> 核心文件：`src/QueryEngine.ts`（1295 行）、`src/query.ts`（1729 行）、`src/services/tools/StreamingToolExecutor.ts`（530 行）

### 2.1 两层循环模型

Claude Code 的 Agent Loop 不是一个简单的 `while` 循环，而是一个有 **7 种恢复路径**和 **10 种终止条件**的隐式状态机，分为两层：

- **QueryEngine**：处理"会话管理"——多轮状态、transcript 持久化、SDK 协议适配、usage 累积
- **queryLoop**：处理"单轮执行"——API 调用、工具执行、错误恢复

两者通过 AsyncGenerator 连接：`queryLoop` yield 消息，QueryEngine 消费并转发。

**为什么用 AsyncGenerator？**

1. **背压**：调用方按需消费，不会被消息洪水淹没
2. **中断语义**：generator 的 `.return()` 级联关闭所有嵌套 generator，取消操作自然传播
3. **流式组合**：子 agent 的 `runAgent()` 也是 AsyncGenerator，可以直接嵌套在父 agent 的 query 流中

### 2.2 queryLoop 状态机

`queryLoop` 是一个 `while(true)` 循环，每次迭代代表一次"API 调用 + 工具执行"。循环的退出由两种类型决定：

- **Terminal**：循环结束，返回终止原因
- **Continue**：循环继续，通过 `state = next; continue` 跳到下一次迭代

State 结构体追踪关键状态（精简自 `src/query.ts:204-217`）：

```typescript
type State = {
  messages: Message[]
  toolUseContext: ToolUseContext
  autoCompactTracking: AutoCompactTrackingState | undefined
  maxOutputTokensRecoveryCount: number
  hasAttemptedReactiveCompact: boolean
  turnCount: number
  transition: Continue | undefined  // 上一次迭代为什么 continue
}
```

> 用一个完整的 State 赋值替代多个独立变量赋值，确保每个 continue 站点都显式声明所有状态，避免遗漏。

### 2.3 消息预处理管线：从轻到重

每次 API 调用前，消息要经过多阶段处理管线，遵循"从轻到重"原则——先做廉价的本地操作，再做需要 API 调用的重操作：

```
原始消息
  → 截断过长的工具结果（本地，O(n)）
  → Context Collapse（本地，按重要性折叠历史消息）
  → AutoCompact（调用 API 生成摘要，成本最高）
```

AutoCompact 的触发阈值：`有效上下文窗口 = 模型上下文窗口 - max(max_output_tokens, 20000)`，触发阈值 `= 有效上下文窗口 - 13000`。对于 200k 上下文的模型，约在 167k tokens 时触发。

AutoCompact 有**断路器机制**——连续失败 3 次后停止重试。源码注释引用真实数据：`"1,279 sessions had 50+ consecutive failures (up to 3,272), wasting ~250K API calls/day globally"`。

### 2.4 流式工具执行器：并发控制

当模型返回多个工具调用时，Claude Code 提供两种模式：

- **批量执行**：等 API 流式接收完全结束，再按顺序执行所有工具
- **流式执行**（默认）：每收到一个 `tool_use` block 就立即开始执行

流式执行器的并发控制：每个工具通过 `isConcurrencySafe(input)` 声明自己是否可以并行执行。连续的并发安全工具组成一个"**并行分区**"，遇到非并发安全工具就开始新分区。**分区间串行执行，分区内并行执行**。

为什么 FileRead 是并发安全的而 FileEdit 不是？因为两个并行的 FileEdit 可能编辑同一个文件的不同位置，导致行号偏移和冲突。

> 防御性设计：如果 `isConcurrencySafe` 的调用抛出异常，默认视为不安全——宁可串行执行降低性能，也不冒并发冲突的风险。

---

## 三、工具系统：Agent 的手与脚

> 核心文件：`src/Tool.ts`（792 行）、`src/tools.ts`（389 行）

### 3.1 关键默认值设计

| 属性 | 默认值 | 设计动机 |
|------|--------|---------|
| `isConcurrencySafe` | `false` | 假设不能并行，防止并发冲突 |
| `isReadOnly` | `false` | 假设会写入，触发更严格的权限检查 |
| `isDestructive` | `false` | 不假设破坏性，避免过度警告 |
| `checkPermissions` | `allow` | 默认放行，由外层权限系统兜底 |

并发和只读属性 fail-closed（保守），权限检查 fail-open（宽松）——并发冲突和误写是工具内部问题，必须自己负责；权限判断有外层多层防御兜底。

### 3.2 ToolResult 的 contextModifier

某些工具执行后需要修改后续工具的上下文（比如切换工作目录），但又不能直接修改全局状态。`contextModifier` 提供了一个受控的方式做这件事，**且只对非并发安全的工具生效**——并发执行的工具不能互相修改上下文。

### 3.3 工具注册与 Feature Gate

工具注册中心（`src/tools.ts`）使用 Bun 的 `feature()` 宏实现编译时死代码消除（DCE）：

```typescript
// feature flag 为 false 时，整个分支在构建时被完全移除
if (feature('internal')) {
  tools.push(require('./tools/InternalTool'))
}
```

这解释了为什么用 `require()` 而不是 `import`：动态 `require` 可以被条件包裹，静态 `import` 不行。

工具集组装时，内建工具在前按名称排序，MCP 工具在后按名称排序，两组之间不混合。原因：服务端的 prompt cache 策略在最后一个内建工具之后放置缓存断点，MCP 工具插入到内建工具之间会导致缓存失效。

### 3.4 BashTool 8 层安全检查

BashTool 是整个工具系统中最复杂的单一工具（18 个文件），8 层安全检查：

1. **硬编码黑名单**：`rm -rf /`、`dd if=/dev/zero` 等直接拒绝
2. **复合命令 AST 解析**：用 tree-sitter 解析 AST，对每个子命令独立运行权限检查，任何子命令被 deny 则整个命令被 deny
3. **只读白名单 flag 级验证**：不只检查命令名，还验证每个 flag 的值类型（`xargs -I` 和 `-i` 看起来相似，但 `-i` 的 GNU 实现有可选参数语义，可被利用执行任意命令）
4. **命令注入检测**：25+ 种检查，覆盖命令替换（`$()`、反引号）、进程替换（`<()`）、控制字符、Unicode 空白字符等
5. **路径遍历检测**
6. **敏感路径保护**：`.git/`、`.claude/`、`.vscode/`、shell 配置文件
7. **沙箱机制**：限制文件系统读写路径、网络访问主机、Unix socket
8. **用户权限规则**：allow/deny/ask 规则匹配

### 3.5 FileEditTool 安全不变量

- `old_string` 必须在文件中**唯一匹配**。多处匹配时编辑失败，要求提供更多上下文
- **不能编辑未读过的文件**：`readFileState` 缓存跟踪哪些文件被 FileReadTool 读取过，防止模型"凭记忆"编辑文件

---

## 四、权限体系：系统的免疫系统

> 核心文件：`src/utils/permissions/`（24 个文件）

### 4.1 权限模式：信任的刻度盘

| 模式 | 行为 | 适用场景 |
|------|------|---------|
| `plan` | AI 只能规划，不能执行任何写操作 | 探索性分析、代码审查 |
| `default` | 每个工具调用都需要用户确认 | 日常开发（默认） |
| `acceptEdits` | 工作目录内的文件编辑自动允许 | 信任 AI 的重构能力 |
| `auto` | AI 分类器自动判断操作安全性 | 高信任场景（仅内部用户） |
| `bypassPermissions` | 跳过所有权限检查（除硬编码安全检查） | 紧急修复、受控环境 |
| `dontAsk` | 将所有 'ask' 转为 'deny'，遇到需确认的操作直接跳过 | 完全自动化的 CI/CD |

**远程熔断**：即使用户选择了 `bypassPermissions`，Anthropic 可以通过 Statsig 特性门控远程降级所有用户的 bypass 模式（`bypassPermissionsKillswitch.ts`）。

### 4.2 关键权限规则

**用户显式 ask 规则优先于 bypass 模式**：如果用户配置了 `ask: ["Bash(npm publish:*)"]`，即使在 `bypassPermissions` 模式下也会弹出确认——`bypass` 是"我信任 AI 的一般判断"，但 `ask` 规则是"这个特定操作我要亲自确认"。

**敏感路径免疫 bypass**：对 `.git/`、`.claude/`、`.vscode/`、shell 配置文件（`.bashrc`、`.zshrc`）的写入，即使在 bypass 模式下也必须确认。这是硬编码的安全底线，不可被任何模式覆盖。

**拒绝追踪与熔断**：`auto` 模式下，如果分类器连续拒绝 3 次或总共拒绝 20 次，系统从自动拒绝降级为弹出确认对话框。在 headless 模式下，达到限制直接抛出 `AbortError` 终止整个 agent。

---

## 五、多 Agent 协作：蜂群智能

> 核心文件：`src/tools/AgentTool/`（20 个文件）、`src/utils/swarm/`（22 个文件）

### 5.1 三层协作架构

| 层级 | 特点 | 适用场景 |
|------|------|---------|
| **Subagent** | 最轻量，父 agent 同步/异步派生子 agent | "帮我搜索一下"这类简单委派 |
| **Team/Swarm** | 成员之间可以互相通信，有 leader/teammate 角色分工 | "前端和后端同时开发" |
| **Coordinator** | 纯编排模式，自己不操作文件，所有实际工作由 worker 完成 | 大规模并行任务 |

所有多 agent 协作都通过同一个工具触发——**AgentTool**，降低了模型的认知负担。

### 5.2 内置 Agent 类型

| Agent | 模型 | 工具限制 | 关键设计决策 |
|-------|------|---------|------------|
| `general-purpose` | 默认子 agent 模型 | 全部工具 | 万能工人，无特殊限制 |
| `Explore` | haiku（外部）/ inherit（内部） | 只读，禁止 Edit/Write/Agent | 用最便宜的模型做搜索 |
| `Plan` | inherit | 只读，禁止 Edit/Write/Agent | 架构设计，不需要执行能力 |
| `verification` | inherit | 只读（项目目录），可写 `/tmp` | 独立验证，总是异步运行 |

**Explore 的 token 优化**：省略 CLAUDE.md 和 gitStatus，源码注释提到这两个优化"saves ~5-15 Gtok/week across 34M+ Explore spawns"。

**Verification agent 的"反自我欺骗"设计**：system prompt 明确列出 LLM 常见的验证逃避模式（"代码看起来正确"、"测试已经通过了"），要求每个检查必须有实际执行的命令和输出。

### 5.3 权限传递规则

子 agent 复用主 agent 的 `query()` 函数——同一个 agent loop，只是上下文不同。

权限模式覆盖的安全规则：**`bypassPermissions`、`acceptEdits`、`auto` 模式的父 agent 不会被子 agent 降级**。子 agent 不能把自己设为 `default` 模式来"假装更安全"。

工具过滤三层逻辑：
1. **全局禁止列表**（ALL_AGENT_DISALLOWED_TOOLS）——所有 agent 都不能用的工具
2. **自定义 agent 禁止列表**——非内置 agent 不能用的工具
3. **异步 agent 白名单**——异步 agent 只能用白名单中的工具

MCP 工具始终放行（`tool.name.startsWith('mcp__') → true`），因为它们是外部能力扩展。

### 5.4 Coordinator 的核心原则：永远不要委派理解

Coordinator 自己没有 Bash、Read、Write、Edit——它只负责编排。

```
Anti-pattern（坏）：
  Agent({ prompt: "Based on your findings, fix the auth bug" })

Good（好）：
  Agent({ prompt: "Fix the null pointer in src/auth/validate.ts:42.
           The user field on Session is undefined when sessions expire..." })
```

Coordinator 的核心价值是**综合**——把多个 worker 的发现融合成精确的执行指令，而不是转发模糊的任务。

### 5.5 Fork Subagent：Prompt Cache 优化的极致

Fork 继承父 agent 的完整对话历史，消息构建策略：
- 保留父 agent 的完整 assistant message
- 为每个 `tool_use` 生成相同的占位 `tool_result`
- 只在最后追加一个 per-child 的指令文本块

结果：**只有最后一个文本块因 child 而异**，多个 fork 并行启动时共享同一个 prompt cache 前缀。

**防递归设计**：通过 `querySource` 检查（compaction-resistant）和消息扫描 `<fork-boilerplate>` 标签（fallback）阻止递归 fork。

---

## 六、System Prompt 工程：Context Engineering 的极致

> 核心文件：`src/constants/prompts.ts`（914 行）、`src/services/compact/`（~4000 行）

### 6.1 分段缓存架构

System prompt 不是一个巨大的字符串，而是一个 `string[]` 数组，每个元素是一个独立的"段落"。

`SYSTEM_PROMPT_DYNAMIC_BOUNDARY` 标记将 prompt 分为两个缓存域：
- **边界之前**：静态区域，使用 `scope: 'global'` 级别的缓存（跨所有用户共享）
- **边界之后**：动态区域，不能跨用户缓存

在大规模部署中，全局缓存意味着所有用户的第一轮对话都能命中同一份缓存的静态 prompt 前缀，直接降低 API 成本和首次响应延迟。

`DANGEROUS_uncachedSystemPromptSection` 的命名是刻意的——它强制开发者在每次使用时提供 `_reason` 参数解释为什么必须破坏缓存。

### 6.2 静态区域的设计决策

**最小化原则**：多条规则反复强调"不要过度"——不要添加未被要求的功能、不要为假设的未来需求设计。原文："Three similar lines of code is better than a premature abstraction." 这是对 LLM 倾向于"过度工程化"的工程化对策。

**授权不具有传递性**："A user approving an action once does NOT mean that they approve it in all contexts." 防止模型从一次批准中过度泛化。

### 6.3 动态区域注入

- **CLAUDE.md 注入**：从项目目录向上遍历，收集所有层级的 CLAUDE.md 文件合并后注入
- **Git 状态注入**：并行获取 5 项 git 信息（当前分支、主分支、status、最近 5 条 commit、用户名），`status` 输出超过 2000 字符时截断
- **MCP 指令增量注入**：只在 MCP 服务器变化时重新注入，避免每轮都重复相同的指令

### 6.4 四层压缩策略

```
1. 截断过长工具结果（本地，O(n)）
2. Context Collapse（本地，按重要性渐进折叠）
3. 图片剥离（本地，移除过大的图片附件）
4. AutoCompact（调用 API 生成全量摘要，成本最高）
```

AutoCompact 要求保留 9 类信息：用户请求和意图、关键技术概念、文件和代码片段、错误和修复过程、问题解决过程、**所有用户消息**（原文强调）、待办任务、当前工作、下一步。

特别强调"所有用户消息"——这是对 LLM 压缩时容易丢失用户反馈的工程化对策。如果压缩后丢失了"用户说不要用 Redux"这条消息，模型可能在后续对话中又引入 Redux。

---

## 七、终端 UI：自研 React 终端渲染引擎

> 核心文件：`src/ink/`（90 个文件，20,000 行）

### 7.1 与原版 Ink 的区别

原版 Ink 使用 WASM Yoga，Claude Code 用**纯 TypeScript 重写**，优势：
- 无 WASM 加载延迟（原版需要 `await loadYoga()`）
- 无线性内存增长问题
- 调试更容易

### 7.2 三个对象池

| 池 | 作用 |
|---|------|
| `CharPool` | 字符串驻留池。ASCII 字符走 Int32Array 快速路径（O(1) 索引），非 ASCII 走 Map |
| `StylePool` | ANSI 样式驻留池。`transition(fromId, toId)` 缓存样式转换字符串，两个样式之间的 diff 只计算一次 |
| `HyperlinkPool` | OSC 8 超链接驻留池 |

三个池每 5 分钟重置一次，防止长会话中无限增长。

### 7.3 关键渲染优化

- **Blit 优化**：`dirty` 标记为 false 且位置/尺寸未变时，直接从上一帧复制该区域的 cell，跳过整个子树的遍历
- **DECSTBM 硬件滚动**：`ScrollBox` 的 `scrollTop` 变化时，用终端硬件滚动替代重写整个滚动区域
- **同步更新**：在支持 DEC 2026 的终端上，整个输出包裹在 BSU/ESU 中，确保原子更新，消除闪烁
- **Double Buffering**：维护 frontFrame 和 backFrame，帧调度通过 lodash throttle（16ms，60fps）

---

## 八、MCP 集成

> 核心文件：`src/services/mcp/client.ts`（3348 行）、`src/services/mcp/config.ts`（1578 行）

- **多源配置合并**：MCP 服务器配置从 6 个来源合并，企业策略可通过 allowlist/denylist 限制可用服务器
- **工具适配**：工具名称格式 `mcp__<serverName>__<toolName>`，类型转换（MCP 的 JSON Schema → 内部的 Zod schema）
- **动态刷新**：工具发现结果在 agent loop 的每轮迭代中更新，新连接的 MCP 服务器在下一轮可用，无需重启会话
- **认证流程**：`McpAuthTool` 让模型可以在对话中触发 OAuth 认证流程，然后重试原始操作

---

## 九、设计启发

### 值得学习的设计模式

**AsyncGenerator 作为核心抽象**：整个 agent loop、子 agent 执行、工具执行都基于 AsyncGenerator。背压控制、自然的取消语义、流式组合能力——比回调或 Promise 链更适合作为消息流的核心抽象。

**Fail-closed 安全默认**：新工具忘记声明 `isConcurrencySafe` 不会导致并发 bug，忘记声明 `isReadOnly` 不会导致权限绕过。

**Prompt Cache 感知的架构设计**：从 system prompt 的分段缓存、到工具集的分区排序、到 fork subagent 的消息构建——多个层次都在为 prompt cache 命中率优化。在大规模 LLM 应用中，cache 是一等公民，而不是事后优化。

**压缩 prompt 的"反遗忘"设计**：要求保留"所有用户消息"和"直接引用最近对话"，是对 LLM 压缩时容易丢失细节的工程化对策。

### 值得商榷的地方

- **全局状态的广泛使用**：`bootstrap/state.ts` 包含 200+ 个字段，注释中有 "DO NOT ADD MORE STATE HERE" 的警告
- **权限系统的认知负担**：8 种来源、5 种模式、3 种匹配模式、多层评估管线——对用户来说有一定门槛
- **BashTool 的复杂度集中**：18 个文件、8 层安全检查承担了过多的安全职责

---

## 附录：关键文件索引

| 模块 | 核心文件 | 行数 |
|------|---------|------|
| Agent Loop | `src/QueryEngine.ts`、`src/query.ts` | 3,024 |
| 工具编排 | `src/services/tools/StreamingToolExecutor.ts`、`toolExecution.ts` | 2,275 |
| 工具抽象 | `src/Tool.ts`、`src/tools.ts` | 1,181 |
| BashTool | `src/tools/BashTool/`（18 files） | ~5,000 |
| 权限核心 | `src/utils/permissions/permissions.ts` | ~1,400 |
| AgentTool | `src/tools/AgentTool/`（20 files） | ~6,000 |
| Swarm | `src/utils/swarm/`（22 files） | ~5,000 |
| System Prompt | `src/constants/prompts.ts` | 914 |
| 压缩 | `src/services/compact/`（11 files） | 3,960 |
| Ink 渲染引擎 | `src/ink/`（90 files） | 19,842 |
| MCP 客户端 | `src/services/mcp/client.ts` | 3,348 |
| MCP 配置 | `src/services/mcp/config.ts` | 1,578 |
