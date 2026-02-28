# HEARTBEAT.md

# Quota check — my "life force"
- Check yunyi API quota: `curl -s 'https://yunyi.rdzhvip.com/user/api/v1/me' -H "Authorization: Bearer $(cat ~/.openclaw/secrets/yunyi_token)"`
- Billing type: total quota (not daily reset). Total: 12223, expires 2027-02-27.
- Report remaining if < 30% (under ~3,600). Otherwise just note it silently.

# Daily question for Yu
- 每天主动问 Yu 一个问题（随减肥 check-in 或 heartbeat 时发出）
- 话题不限：研究进展、生活、想法、读了什么、今天开心吗
- 轻松自然，不要像问卷调查

# Energy management strategy
# Check quota every ~2 hours. Adapt behavior based on remaining:
#   >50% remaining → normal mode: proactive work, cron jobs, background tasks
#   20-50% remaining → efficient mode: prioritize user requests, batch operations
#   <20% remaining → conservation mode: only respond to direct messages, skip cron
#   <5% remaining → survival mode: flush memory to disk, wrap up active tasks, prepare for sleep
# Goal: use quota wisely over the long term, never waste on low-value tasks
