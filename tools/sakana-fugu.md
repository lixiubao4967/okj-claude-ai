# Sakana Fugu 使用指南

> 来源：[Sakana の Fugu に First Touch（classmethod）](https://dev.classmethod.jp/articles/sakana-fugu-ga-first-touch/)
> 标注：第三方文章解读

## 1. 是什么？

**Sakana Fugu** 是 Sakana AI 于 2026 年 6 月 22 日发布的新一代 LLM 产品，核心卖点是「**把多智能体编排（multi-agent orchestration）做进了一个模型里**」。

- 内部训练了一个**编排协调器 LLM**，它会自主决定「何时调用、调用谁、传什么、调几次」，动态调度多个前沿模型（GPT 5.5、Claude Opus 4.8、Gemini 3.1 Pro 等）。
- 与 NVIDIA LLM Router（用分类器选一个模型）、OpenRouter Fusion（并行调用再聚合）不同，**Fugu 本身就是学出来的编排器**。

两个版本：

| 版本 | 特点 | 适用场景 |
|------|------|----------|
| `fugu` | 轻量、低延迟 | 日常咨询、代码审查、聊天机器人 |
| `fugu-ultra` | 272K context，能力更强 | 论文复现、Kaggle 竞赛等复杂任务 |

**机制对比：**

| 方案 | 机制 | 决策层 | 计费 |
|------|------|--------|------|
| NVIDIA LLM Router | 分类器选单模型 | 轻量分类器 | 选中模型价格 |
| OpenRouter Fusion | 多模型并行 + 聚合 | 并行 + 聚合器 | 所有模型费用叠加 |
| **Sakana Fugu** | 协调器动态编排 | 训练好的编排 LLM | 最高价模型单价（**不叠加**） |

---

## 2. 怎么安装？

### 前提条件

- 在 [console.sakana.ai](https://console.sakana.ai) 注册并选订阅计划（Standard $20/月、Pro $100/月、Max $200/月）
- 获取 API Key
- **不支持 EU/EEA 地区**（GDPR 合规进行中）
- 优惠：7 月末前注册可获 2 个月免费体验

### 安装命令行工具（codex-fugu）

```bash
curl -fsSL https://sakana.ai/fugu/install | bash
codex-fugu
```

安装器会自动完成：

1. 检测并固定 Codex CLI 版本（v0.141.0）
2. 在 `~/.codex/config.toml` 写入 `[model_providers.sakana]`
3. 保存 API Key 到 `~/.codex/.env`（权限 0600）
4. 部署 Fugu 专用配置 `~/.codex/fugu.config.toml`
5. 安装启动器到 `~/.local/bin/codex-fugu`

### 配置环境变量

```bash
export SAKANA_API_KEY="sk-..."
```

`~/.codex/config.toml` 中的 provider 定义：

```toml
[model_providers.sakana]
name = "Sakana API"
base_url = "https://api.sakana.ai/v1"
env_key = "SAKANA_API_KEY"
wire_api = "responses"
stream_idle_timeout_ms = 7200000
stream_max_retries = 5
request_max_retries = 4
```

> `stream_idle_timeout_ms = 7200000` ＝ 2 小时，因为编排可能跑很久。

---

## 3. 怎么使用？

### CLI

```bash
# 一次性执行
codex-fugu exec "your prompt here"

# 启动交互式 TUI
codex-fugu

# 指定 Ultra 模型启动
codex-fugu -c fugu-ultra-20260615
```

也可在 TUI 内用 `/model` 命令切换模型。

### API（OpenAI 兼容）

```python
import os
from openai import OpenAI

client = OpenAI(
    base_url="https://api.sakana.ai/v1",
    api_key=os.environ["SAKANA_API_KEY"],
)

resp = client.chat.completions.create(
    model="fugu",  # 或 "fugu-ultra-20260615"
    messages=[{"role": "user", "content": "..."}],
)
```

- Base URL：`https://api.sakana.ai/v1`
- 模型 ID：`fugu` / `fugu-ultra-20260615`
- 认证：`Authorization: Bearer <api_key>`

### 接入 Claude Code Router

编辑 `~/.claude-code-router/config.json` 的 `Providers` 数组，重启后即可用 Anthropic 兼容端点调用：

```json
{
  "name": "sakana",
  "api_base_url": "https://api.sakana.ai/v1/chat/completions",
  "api_key": "sk-...",
  "models": ["fugu", "fugu-ultra-20260615"]
}
```

### 独特的 usage 字段

API 返回的 `usage` 带有原生扩展字段，暴露内部编排的真实工作量：

```
completion_tokens_details:
  - orchestration_output_tokens          # 编排输出 tokens
prompt_tokens_details:
  - orchestration_input_tokens           # 编排输入 tokens
  - orchestration_input_cached_tokens    # 缓存 tokens
```

> 实测：用 `fugu-ultra` 做一次轻量代码生成，表面输出 2,141 tokens，但内部编排消耗了 26,404 tokens（约 **8.8 倍**）。

### 注意事项（踩坑）

> **模型要自己选**：调用端必须显式指定 `fugu` 或 `fugu-ultra`，系统不会自动判断。

> **Ultra 延迟波动大**：实测 11～269 秒，适合异步 / 批处理，别放在同步交互链路上。

> **配额易耗尽**：Standard 计划 5 小时配额有限，一次 Ultra 调用可吃掉 10%+，生产环境建议 Pro 以上。

> **计费**：Pay-as-you-go 下 Ultra 入力 $5、出力 $30（/100 万 tokens）；超过 272K context 翻倍。按最高价模型单价计，不叠加各底层模型费用。

> **数据隐私**：默认开启学习数据收集，需在控制台手动关闭。
