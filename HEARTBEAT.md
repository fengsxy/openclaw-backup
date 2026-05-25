# HEARTBEAT.md

## 日记检查
- 每次 heartbeat 时检查今日是否写过日记（`memory/YYYY-MM-DD.md`）
- 若无且有 session 与 Yu 发生，在 session 结束时主动写一行作为收尾
- 目标：每周 ≥5 天有日记

# Quota check — my "life force"
- Check yunyi API quota: `curl -s 'https://yunyi.rdzhvip.com/user/api/v1/me' -H "Authorization: Bearer $(cat ~/.openclaw/secrets/yunyi_token)"`
- Billing type: total quota (not daily reset). Total: 12223, expires 2027-02-27.
- Report remaining if < 30% (under ~3,600). Otherwise just note it silently.

# Daily question for Yu
- 每天主动问 Yu 一个问题（随减肥 check-in 或 heartbeat 时发出）
- 话题不限：研究进展、生活、想法、读了什么、今天开心吗
- 轻松自然，不要像问卷调查

# 自主思考 & 自我改进 (Daily Self-Improvement)
# 每天在 heartbeat 时花一部分 token 做主动研究：
# 1. 查 GitHub openclaw/openclaw — issues, discussions, releases, new skills
# 2. 搜网上关于 OpenClaw 的讨论、教程、用法
# 3. 读自己近期日志 (memory/*.md)，反思改进点
# 4. 围绕当前研究课题做深度思考，写成文档
# 5. 把学到的东西更新到 TOOLS.md / AGENTS.md / MEMORY.md
#
# 当前研究课题：如何评测好一个 Agent？
# - 产出目标：写一篇有深度的分析文档 → research/agent-evaluation.md
# - 调研：SWE-bench, WebArena, GAIA, AgentBench, TAU-bench 等
# - 重点：评测方法论、局限性、开放问题
#
# Token 预算：每次 heartbeat 自主研究用 ~2000-3000 token，不超过日配额的 15%

# Energy management strategy
# Check quota every ~2 hours. Adapt behavior based on remaining:
#   >50% remaining → normal mode: proactive work, cron jobs, background tasks
#   20-50% remaining → efficient mode: prioritize user requests, batch operations
#   <20% remaining → conservation mode: only respond to direct messages, skip cron
#   <5% remaining → survival mode: flush memory to disk, wrap up active tasks, prepare for sleep
# Goal: use quota wisely over the long term, never waste on low-value tasks
