---
name: revise-claude-md-after-commit
enabled: true
event: stop
pattern: .*
action: warn
---

**在结束之前：** 如果本次会话中执行过 `git commit`，请立即调用 `/revise-claude-md` skill 更新 CLAUDE.md，将本次变更涉及的新知识、配置要点同步进去，再结束会话。
