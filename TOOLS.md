# TOOLS.md - Local Notes

Skills define _how_ tools work. This file is for _your_ specifics — the stuff that's unique to your setup.

## GitHub

- **Repo:** git@github.com:fengsxy/paper_reading.git
- **SSH Key:** `~/.ssh/id_ed25519` (ed25519, added to GitHub 2026-02-18)
- **Workspace:** `/home/ubuntu/.openclaw/workspace`

## Secrets Location

敏感信息存放在 `~/.openclaw/secrets/`（不进 git）：

- **YouTube Cookies:** `~/.openclaw/secrets/youtube_cookies.txt`
  - 用于 yt-dlp 下载 YouTube 视频
  - Cookies 格式: Netscape format (从 Chrome 导出 JSON 后需转换)

## YouTube 下载方法 (2026-02-19 验证可用)

YouTube 现在需要 PO Token + JS runtime 来解决 n challenge。完整命令：

```bash
source ~/.openclaw/workspace/.venv/bin/activate

# 下载音频
yt-dlp --js-runtimes node --remote-components ejs:npm \
  --cookies ~/.openclaw/secrets/youtube_cookies.txt \
  -x --audio-format mp3 --audio-quality 0 \
  -o "%(title)s.%(ext)s" \
  "https://www.youtube.com/watch?v=VIDEO_ID"

# 列出可用格式
yt-dlp --js-runtimes node --remote-components ejs:npm \
  --cookies ~/.openclaw/secrets/youtube_cookies.txt \
  --list-formats "https://www.youtube.com/watch?v=VIDEO_ID"
```

关键参数说明：
- `--js-runtimes node`: 使用 Node.js 作为 JavaScript runtime
- `--remote-components ejs:npm`: 从 npm 下载 EJS challenge solver scripts
- 这两个参数缺一不可，否则只能下载 storyboard 图片

如果 cookies 过期：
1. 在 Chrome 登录 YouTube
2. 用 cookies 导出插件导出 JSON
3. 转换为 Netscape 格式保存到 `~/.openclaw/secrets/youtube_cookies.txt`
- **API Keys:** 需要设置环境变量
  - `GROQ_API_KEY` - 用于 Whisper 转录（免费，https://console.groq.com/keys）
  - `OPENAI_API_KEY` - 备用转录方案

## Search

- **Primary:** You.com YDC Search (`https://ydc-index.io/v1/search`)
- **Fallback:** `web_fetch` 直接抓页面（GitHub、文档、已知 URL）
- **Brave Search:** 已弃用，无 API key。内置 `web_search` 工具会报错，忽略即可。
- **Claude Code web search:** Claude Code 内置 web search tool，派 coding-agent 用 Claude Code 时可直接搜索
- YDC key: `~/.openclaw/secrets/ydc_api_key`
- 用法: `curl -sS --get 'https://ydc-index.io/v1/search' --data-urlencode 'query=...' --data-urlencode 'count=10' --data-urlencode 'language=EN' -H "Accept: application/json" -H "X-API-KEY: $YDC_API_KEY"`

## Backup

- **Repo:** `git@github.com:fengsxy/openclaw-backup.git` (私有，待创建)
- **Script:** `scripts/backup_workspace.sh` — 备份 agent 配置 + memory/ + process/ 到私有 repo
- **备份内容:** AGENTS.md, SOUL.md, USER.md, TOOLS.md, IDENTITY.md, HEARTBEAT.md, memory/*.md, process/*.md
- **频率:** 每日 cron（待配置）

## 雪球 (Xueqiu)

- **Cookie:** `~/.openclaw/secrets/xueqiu_cookie.txt`
- **行情接口:** `https://stock.xueqiu.com/v5/stock/quote.json?symbol=PDD` (需带 Cookie)
- **帖子接口:** 被阿里云 WAF 拦截，服务器端 curl 不可用
- **脚本:** `scripts/xueqiu_daily.py` — PDD/MSFT/VOO/QQQ/SPY 行情 + 持仓盈亏
- **Cron:** "雪球每日简报" (4945510b) — 周一到周五 1:30 PM PT
- **Cookie 过期:** JWT exp 2026-06-23，届时需要重新获取

## x-reader (Universal Content Reader)

- **Repo:** `github.com/runesleo/x-reader`
- **安装:** venv 内 `pip install git+https://github.com/runesleo/x-reader.git`
- **Skill:** `skills/x-reader/SKILL.md`
- **CLI:** `source .venv/bin/activate && x-reader <url>`
- **支持平台:** 微信公众号、小红书、X/Twitter、YouTube、B站、Telegram、RSS、小宇宙、Apple Podcasts
- **YouTube:** 自动提取字幕，无字幕时 fallback 到 Groq Whisper（需 GROQ_API_KEY）
- **小红书:** 需先登录 `x-reader login xhs`（Playwright headless 扫码），session 存 `~/.x-reader/sessions/xhs.json`
- **小红书登录注意:** x-reader 自带 login 有 EPIPE bug，用自写 Playwright 脚本替代（见 memory/2026-02-25.md）
- **Playwright:** 已装 `playwright install --with-deps chromium`
- **Batch 脚本:** `skills/x-reader/scripts/batch_read.py`

## What Goes Here

Things like:

- Camera names and locations
- SSH hosts and aliases
- Preferred voices for TTS
- Speaker/room names
- Device nicknames
- Anything environment-specific

## Examples

```markdown
### Cameras

- living-room → Main area, 180° wide angle
- front-door → Entrance, motion-triggered

### SSH

- home-server → 192.168.1.100, user: admin

### TTS

- Preferred voice: "Nova" (warm, slightly British)
- Default speaker: Kitchen HomePod
```

## Agent Evaluation Frameworks (Reference)

These are **external** benchmarks/frameworks for evaluating OpenClaw agents. Not installed locally, but documented for research and comparison.

- **PinchBench** (kilo.ai) — OpenClaw-native coding agent benchmark. Real-world tasks, automated + LLM judge scoring. Public leaderboard (mid-80% top scores). Repo: `github.com/pinchbench/skill`. Use to compare model performance on agentic workflows.
- **WildClawBench** (InternLM/上海 AI Lab) — "In-the-wild" benchmark running inside real OpenClaw instances with actual tools (bash, filesystem, browser, email, calendar). 60 original tasks, hard difficulty. HuggingFace dataset: `internlm/WildClawBench`. Includes Personal OpenClaw Leaderboard for long-term interaction studies.
- **PASB** (Personalized Agent Security Bench, arXiv 2602.08412) — Security evaluation framework. Tests attack propagation across user prompt processing, external content, tool invocation, memory. Case study on OpenClaw reveals system-level harms beyond text generation.
- **AgentBench skill** — OpenClaw skill with 40 real-world tasks (YAML). Good for targeted capability profiling across tool efficiency, structural accuracy, methodology.
- **ClawExam** — Community platform; embed adversarial tasks inside normal workflows (prompt injection, leakage). Downloadable skill.

See `research/agent-evaluation.md` for detailed analysis and how these inform a five-dimension evaluation radar (Correctness, Efficiency, Robustness, Process Quality, Safety).

## Why Separate?

Skills are shared. Your setup is yours. Keeping them apart means you can update skills without losing your notes, and share skills without leaking your infrastructure.

---

Add whatever helps you do your job. This is your cheat sheet.
