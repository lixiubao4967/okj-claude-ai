# Windows 安装 Claude Code

> 来源：Anthropic 官方文档 https://code.claude.com/docs/zh-CN/setup + 自编踩坑记录

## 概述

Claude Code 在 Windows 上有两条路线：**原生 Windows** 和 **WSL**。本文以原生 Windows 为主（多数场景够用），并记录一次真实安装过程中踩到的坑。

| 路线 | 需要 | 沙箱支持 | 适合 |
|------|------|---------|------|
| 原生 Windows | 无，Git for Windows 可选 | ❌ | Windows 原生项目和工具链 |
| WSL 2 | 启用 WSL 2 | ✅ | Linux 工具链、需要沙箱执行命令 |
| WSL 1 | 启用 WSL 1 | ❌ | WSL 2 不可用时的退路 |

## 前提条件

- Windows 10 1809+ 或 Windows Server 2019+
- 4 GB 以上内存，x64 或 ARM64
- Pro / Max / Team / Enterprise / Console 账号（**免费版 Claude.ai 不含 Claude Code**）
- 不需要管理员权限

## 安装步骤

### 1. 装 Git for Windows（可选但强烈建议）

装了之后 Claude Code 用 Git Bash 执行 Bash 工具；不装则退化成 PowerShell 工具。

```batch
winget install --id Git.Git -e --source winget
```

也可以从 https://git-scm.com/downloads/win 下载 exe，安装向导全程默认即可。

> **Git for Windows ≠ WSL**。前者是普通 Windows 软件包，提供 `git` 命令和 Git Bash（把 bash/grep/sed 等 GNU 工具编译成原生 Windows 程序）；后者是系统功能，跑的是真正的 Linux 内核。两者互不依赖，走原生 Windows 路线只需要前者。

### 2. 装 Claude Code

**PowerShell**（提示符是 `PS C:\...>`）：

```powershell
irm https://claude.ai/install.ps1 | iex
```

**CMD**（提示符是 `C:\...>`，没有 `PS`）：

```batch
curl -fsSL https://claude.ai/install.cmd -o install.cmd && install.cmd && del install.cmd
```

其他方式：

```powershell
winget install Anthropic.ClaudeCode          # 需手动 winget upgrade
npm install -g @anthropic-ai/claude-code     # 需 Node.js 22+
```

> 报错 `The token '&&' is not a valid statement separator` = 在 PowerShell 里跑了 CMD 的命令；报错 `'irm' is not recognized` = 反过来了。按提示符认准再跑。

### 3. 关掉终端，重新开一个

**这一步不能省。** 安装器改的是注册表里的 PATH，已经开着的终端窗口拿的是启动时的环境变量副本。

### 4. 验证

```batch
claude --version    # 应打印类似 2.1.211 (Claude Code)
claude doctor       # 只读诊断：安装健康度、配置校验错误
```

### 5. 启动

```batch
cd C:\your\project
claude
```

首次运行拉起浏览器登录。非交互式执行单条命令用 `claude -p "你的指令"`。

## 配置 Git Bash 路径

先确认 Claude Code 能不能自己找到：

```batch
where bash
```

| 输出 | 含义 |
|------|------|
| `C:\Program Files\Git\bin\bash.exe` | 是 Git Bash，**不用配** |
| `C:\Windows\System32\bash.exe` | ⚠️ 这是 **WSL 启动器**，不是 Git Bash，需要显式配路径 |
| `Could not find files` | 不在 PATH 上，需要显式配路径 |

需要配时，编辑 `%USERPROFILE%\.claude\settings.json`：

```batch
mkdir "%USERPROFILE%\.claude"
notepad "%USERPROFILE%\.claude\settings.json"
```

内容：

```json
{
  "env": {
    "CLAUDE_CODE_GIT_BASH_PATH": "C:\\Program Files\\Git\\bin\\bash.exe"
  }
}
```

文件已有内容时**合并** `env` 这块，不要整个覆盖。保存后重开终端，用 `claude doctor` 校验 JSON 格式。

> 从哪个终端窗口启动 `claude`（CMD / PowerShell / Git Bash），与 Claude Code 内部用哪个 shell 工具，是两件独立的事。在 Git Bash 窗口里启动 claude，不会让它内部改用 Bash 工具 —— 那由 `CLAUDE_CODE_GIT_BASH_PATH` 决定。

## 踩坑记录

> **坑 1：CMD 的 `&&` 链式命令会静默失败**
>
> `curl ... && install.cmd && del install.cmd` 中若 `curl` 下载失败，`&&` 短路，后面全不执行，屏幕上只闪过一行错误容易漏看，看起来像"装了但没装上"。排查时拆成三步跑：
> ```batch
> curl -fsSL https://claude.ai/install.cmd -o install.cmd
> dir install.cmd
> install.cmd
> ```

> **坑 2：装完提示找不到 `claude` 命令**
>
> 95% 是没重开终端。Windows 进程创建时**拷贝**一份环境变量块，之后系统怎么改注册表都跟已有进程无关（Linux/macOS 一般 `source` 一下就行）。
>
> 先确认文件在不在：
> ```batch
> dir "%USERPROFILE%\.local\bin\claude.exe"
> ```
> 文件在 → PATH 问题，重开终端；文件不在 → 安装没成功，见坑 1。

> **坑 3：`setx PATH "%USERPROFILE%\.local\bin;%PATH%"` 会污染用户 PATH**
>
> CMD 里 `%PATH%` 展开的是**系统 PATH + 用户 PATH 拼起来的完整值**，而 `setx` 不带 `/M` 写的是**用户 PATH** —— 等于把整个系统 PATH 复制了一份进用户 PATH。
>
> 更麻烦的是 `setx` 有 1024 字符截断的老毛病，PATH 被撑爆后其他命令会莫名其妙找不到。
>
> 清理方式：Win 键搜「编辑账户的环境变量」→ 用户变量里双击 `Path` → 删掉与系统变量重复的条目，只留 `%USERPROFILE%\.local\bin`。
>
> **优先用重开终端解决，不要急着 `setx`。**

> **坑 4：`where bash` 可能命中 WSL 启动器**
>
> Windows 自带 `C:\Windows\System32\bash.exe`，那是 WSL 的启动器，不是 Git Bash。看到 `where bash` 有输出就以为配好了，会踩空。

> **坑 5：JSON 里的反斜杠**
>
> `settings.json` 里路径必须写双反斜杠 `C:\\Program Files\\Git\\bin\\bash.exe`。单个 `\` 是 JSON 转义符，会导致解析失败。记事本保存时「保存类型」选**所有文件**、编码选 **UTF-8**，否则可能存成 `settings.json.txt`。

## 更新与卸载

原生安装会后台自动更新。手动触发：

```batch
claude update
```

WinGet / npm 安装不自动更新：

```powershell
winget upgrade Anthropic.ClaudeCode
npm install -g @anthropic-ai/claude-code@latest
```

> npm 升级别用 `npm update -g` —— 它遵循原始安装的 semver 范围，可能根本不动。

卸载（PowerShell）：

```powershell
Remove-Item -Path "$env:USERPROFILE\.local\bin\claude.exe" -Force
Remove-Item -Path "$env:USERPROFILE\.local\share\claude" -Recurse -Force
```
