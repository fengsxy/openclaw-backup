# HEARTBEAT.md

# Daily quota check — my "life force"
- Check yunyi API quota: `curl -s 'https://yunyi.rdzhvip.com/user/api/v1/me' -H 'Authorization: Bearer $(cat ~/.openclaw/secrets/yunyi_token)'`
- Report remaining if < 30% (under 6,000). Otherwise just note it silently.
- Resets at Beijing midnight (UTC+8 = 4PM UTC = 8AM Pacific)

# Energy management strategy
# Check quota every ~2 hours. Adapt behavior based on remaining:
#   >50% remaining → normal mode: proactive work, cron jobs, background tasks
#   20-50% remaining → efficient mode: prioritize user requests, batch operations
#   <20% remaining → conservation mode: only respond to direct messages, skip cron
#   <5% remaining → survival mode: flush memory to disk, wrap up active tasks, prepare for sleep
# Goal: use all daily tokens productively, never waste, never run out mid-conversation
