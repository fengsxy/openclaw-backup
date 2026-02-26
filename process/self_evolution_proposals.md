# Self-Evolution Proposals

每小时自动研究 OpenClaw 最佳实践并提案。Yu 审阅后决定是否执行。

---

## Proposal #1 — 2026-02-19 20:20 UTC
**创建 YouTube 转录 Skill（`~/.openclaw/skills/youtube-transcribe/`）**
来源：OpenClaw 社区最佳实践（Substack 技能指南）
问题：当前 yt-dlp + Whisper 转录流程的指令散落在 TOOLS.md 和各 cron prompt 中，每次 session 都要重复加载 cookies 路径、API key 等上下文。
方案：创建第一个 managed skill `youtube-transcribe`，包含 SKILL.md（cookies 路径、转录命令模板、输出格式规范）+ refs/（常用参数参考）。所有 cron 和 session 只需一句"用 youtube-transcribe skill"即可。
收益：消除重复上下文 ~200 tokens/session，单点更新，为后续 skill 化（podcast、scholar、hackernews）打样。
风险：零——只是新增文件，不改现有配置。

---

## Proposal #2 — 2026-02-19 21:20 UTC
**创建 MEMORY.md 长期记忆文件**
来源：AGENTS.md 自身规范 + 社区记忆管理最佳实践
问题：workspace 有 1349 行 daily memory（8个文件），但 MEMORY.md 不存在。AGENTS.md 明确要求主 session 加载 MEMORY.md 作为"策展后的长期记忆"，当前每次主 session 启动都没有长期上下文。
方案：从现有 `memory/2026-02-15.md` 到 `memory/2026-02-19.md` 中提炼关键决策、偏好、项目状态，创建 MEMORY.md（控制在 80 行以内）。内容分类：Yu 的偏好/习惯、活跃项目状态、重要决策记录、工具配置要点。
收益：主 session 立即获得持续上下文，减少重复解释，agent 表现更连贯。
风险：零——纯新增文件。建议 Yu 审阅初版内容确认准确性。

---

## Proposal #3 — 2026-02-20 17:20 UTC
**为 openclaw.json 添加 context pruning 配置，降低 token 消耗**
来源：Skywork "Clawdbot Developer Lessons" 生产模式指南 + Hostinger OpenClaw 最佳实践
问题：当前配置只有 `compaction: { mode: "safeguard" }`，缺少 `pruning` 设置。大量 cron job 和 subagent 的 tool 输出（web_fetch、exec 等）会在活跃上下文中累积，浪费 token。
方案：在 `agents.defaults` 中添加 pruning 配置：`"pruning": { "mode": "cache-ttl", "ttl": "1h", "keepLastAssistants": 3 }`。这会自动丢弃超过 1 小时的大体积 tool 结果，只保留最近 3 轮 assistant 回复。
收益：显著减少长 session 的 token 消耗，compaction 触发频率降低，session 响应更快。
风险：低——pruning 只影响上下文窗口中的旧 tool 结果，不删除 transcript 持久化数据。可随时调整 ttl 值。

---

## Proposal #4 — 2026-02-20 18:20 UTC
**为 subagent 和 cron session 启用 sandbox 隔离**
来源：OpenClaw 官方 sandboxing 文档 + Hostinger 安全最佳实践（"Use isolated environments for execution"）
问题：当前配置无 sandbox 设置，所有 cron job 和 subagent 的 exec/web_fetch 等工具直接在宿主机运行。self-evolution cron 每小时抓取外部网页，web_fetch 内容可能包含恶意 prompt injection，无隔离意味着 blast radius 是整个系统。
方案：在 `agents.defaults` 中添加：`"sandbox": { "mode": "non-main", "scope": "session", "workspaceAccess": "ro" }`。主 session（直接对话）不受影响，仅 cron/subagent 在 Docker 容器中运行。需先执行 `scripts/sandbox-setup.sh` 构建镜像。
收益：cron 和 subagent 的文件系统/进程访问被隔离，即使处理恶意外部内容也不影响宿主机。符合最小权限原则。
风险：中——需要 Docker 环境，sandbox 内无网络（默认），需配置 `docker.network` 允许出站；部分依赖宿主机路径的操作（如 yt-dlp cookies）需通过 `docker.binds` 挂载。建议先测试一个 cron job 再全面启用。

---

## Proposal #5 — 2026-02-20 19:20 UTC
**配置 model fallbacks 实现多 provider 自动故障转移**
来源：OpenClaw 官方 Model Failover 文档（`/concepts/model-failover.md`）
问题：当前配置了 3 个 provider（yunyi-codex、fucheers-claude、yunyi-claude），但 `agents.defaults.model` 只设了 `primary`，没有 `fallbacks`。如果 yunyi-codex 宕机或限流，所有 session（包括 cron）会直接失败，无自动恢复。
方案：添加 fallback 链：`"model": { "primary": "yunyi-codex/gpt-5.2", "fallbacks": ["yunyi-claude/claude-opus-4-6", "fucheers-claude/claude-opus-4-5-20251101"] }`。当主模型所有 auth profile 失败后，自动切换到 Claude，保证服务连续性。
收益：零停机——任一 provider 故障时自动降级到备用模型，cron job 和主 session 均不中断。利用已有的 3 个 provider 配置，无额外成本。
风险：极低——fallback 仅在主模型失败时触发，正常情况下不影响行为。不同模型的输出风格可能略有差异，但胜过完全无响应。

---

## Proposal #6 — 2026-02-20 20:20 UTC
**将例行 hourly cron job 切换到零成本模型（gpt-5.2），仅保留高质量任务用 opus**
来源：OpenClaw 官方 Cron Jobs 文档（model overrides 章节）+ 当前 cron 配置分析
问题：当前 4 个 hourly cron job 全部使用 `yunyi-claude/claude-opus-4-6`（cost: input=5, output=25），包括 "Hourly Progress Report"、"Self-Evolution Research"、"Transcribe Watchdog"、"Podcast 300 Progress Drive"。而 `yunyi-codex/gpt-5.2` 成本为零，完全能胜任这些例行任务。
方案：用 `openclaw cron edit` 将 4 个 hourly job 的 model 改为 `yunyi-codex/gpt-5.2`。仅保留每日任务（Paper Digest、Karpathy RSS 等需要深度分析的）继续用 opus。
收益：每小时节省 4 次 opus 调用的 token 费用，每天节省 ~96 次 opus 调用。gpt-5.2 对于 web 抓取、状态汇报、文件检查等例行任务绑绑有余。
风险：低——如果 gpt-5.2 质量不够，可随时改回。建议先切 1 个 job 观察一天再全面推广。

---

## Proposal #7 — 2026-02-20 21:20 UTC
**添加 session.maintenance 配置，自动清理过期 session transcript 文件**
来源：OpenClaw 官方 Configuration Examples（`session.maintenance` 章节）+ 当前配置审计
问题：当前 `openclaw.json` 完全没有 `session` 配置。9 个 cron job（4 个 hourly）每次运行都创建独立 session transcript（`.jsonl` 文件），这些文件会无限累积在磁盘上。与 Proposal #3（内存中 context pruning）不同，这是磁盘层面的清理。
方案：添加 `"session": { "maintenance": { "mode": "warn", "pruneAfter": "30d", "maxEntries": 500, "rotateBytes": "10mb" } }`。超过 30 天的 session 文件会被清理，单个 transcript 超过 10MB 会轮转。`mode: "warn"` 先只告警不删除，确认安全后改为 `"prune"`。
收益：防止磁盘空间被废弃 session 文件逐渐吃满；hourly cron 每天产生 ~96 个 session 文件，一个月就是 ~2880 个。
风险：极低——`mode: "warn"` 只记录日志不执行删除，可以先观察哪些文件会被清理，确认无误后再启用实际清理。

---

## Proposal #8 — 2026-02-20 22:20 UTC
**启用 compaction.memoryFlush，在上下文压缩前自动保存记忆**
来源：OpenClaw 官方 Memory 文档（`/concepts/memory.md` — "Automatic memory flush" 章节）
问题：当前 compaction 配置仅 `{ "mode": "safeguard" }`，没有 `memoryFlush`。当主 session 长对话触发 auto-compaction 时，上下文中的重要决策、偏好、临时笔记会被压缩摘要替代，如果 agent 没有主动写入 memory 文件，这些信息就永久丢失了。
方案：扩展 compaction 配置：`"compaction": { "mode": "safeguard", "memoryFlush": { "enabled": true, "softThresholdTokens": 4000 } }`。在 compaction 触发前 ~4000 tokens 时，自动插入一个静默 agent turn 提醒写入 `memory/YYYY-MM-DD.md`。
收益：长对话中的关键上下文不再因 compaction 而丢失；与 Proposal #2（创建 MEMORY.md）互补——有了文件还需要有自动写入机制。
风险：极低——flush turn 默认静默（NO_REPLY），用户无感知；仅在接近 compaction 阈值时触发一次，不增加常规 token 消耗。

---

## Proposal #9 — 2026-02-21 16:20 UTC
**添加 logging 配置，启用结构化日志记录**
来源：OpenClaw 官方 Configuration Examples（`logging` 章节）
问题：当前 `openclaw.json` 完全没有 `logging` 配置。9 个 cron job（4 个 hourly）+ 多个 subagent 每天产生大量运行，但错误、rate limit、provider 故障等事件完全不可见。排查问题只能靠猜测或手动 `openclaw logs` 实时查看，无法回溯历史。
方案：添加 `"logging": { "level": "info", "file": "/tmp/openclaw/openclaw.log", "consoleLevel": "warn", "consoleStyle": "pretty", "redactSensitive": "tools" }`。日志写入 `/tmp/openclaw/` 避免占用 workspace，`redactSensitive: "tools"` 自动脱敏 tool 输出中的 API key 等敏感信息。
收益：provider 故障、cron 失败、rate limit 等问题可通过日志快速定位；`redactSensitive` 防止敏感信息泄露到日志文件；为后续监控告警（Proposal #4 sandbox 等）提供基础设施。
风险：极低——只是新增日志输出，不影响任何现有行为。`/tmp` 目录重启自动清理，不会无限增长。

---

## Proposal #10 — 2026-02-22 16:20 UTC
**启用 boot-md hook 并创建 BOOT.md 网关启动自检脚本**
来源：`openclaw hooks list` 显示 `boot-md` hook 已 ready 但未启用；ClawHub 社区 startup-validation 最佳实践
问题：gateway 重启后（手动或崩溃恢复），没有任何自动验证机制。cron job 是否正常注册、API provider 是否可达、磁盘空间是否充足、关键文件（cookies、SSH key）是否存在——全靠人工检查。当前 9 个 cron job + 3 个 provider，任何一个静默失败都可能数小时后才发现。
方案：1) 在 `hooks.internal.entries` 中添加 `"boot-md": { "enabled": true }`；2) 创建 `BOOT.md`，内容为：检查 cron 数量是否符合预期、ping 各 provider endpoint、检查磁盘使用率、验证 cookies/key 文件存在、将结果发送到 Telegram。
收益：gateway 每次启动自动执行健康检查，问题在秒级发现而非小时级；与 Proposal #9（运行时日志）互补——这是启动时的一次性验证。
风险：极低——boot-md 仅在 gateway 启动时运行一次，不影响正常运行；BOOT.md 内容可随时调整。

---

## Proposal #11 — 2026-02-22 17:20 UTC ⚠️ 紧急
**添加磁盘空间监控 cron + 立即清理 data/audio 中的已转录音频文件**
来源：self-evolution 研究中发现磁盘已满（`df -h` 显示 29GB 中仅剩 135MB，使用率 100%）
问题：`workspace/data/audio/` 占 9.3GB，`workspace/audio/` 占 598MB，合计 ~10GB 音频文件占磁盘总量的 34%。磁盘已满会导致 cron job 写入失败、gateway 日志丢失、session transcript 无法保存。这是当前系统最紧迫的风险。
方案：1) 立即：检查 `data/audio/` 中哪些文件已完成转录，将已转录的源音频移至外部存储或删除；2) 长期：在 HEARTBEAT.md 中添加磁盘检查项，当使用率 >85% 时主动告警；3) 考虑将大文件存储迁移到外挂 EBS 卷或 S3。
收益：恢复磁盘可用空间，防止系统级故障；建立持续监控机制避免再次发生。
风险：删除音频文件前必须确认转录已完成且文本已保存。建议 Yu 先审阅文件列表再决定删除范围。

---

## Proposal #12 — 2026-02-22 18:20 UTC ⚠️ 重要
**为 OpenClaw workspace 关键文件建立独立备份机制**
来源：`.gitignore` 审计发现关键文件无任何备份；ClawHub 上 `openclaw-backup`（3.534）等备份 skill 高度流行
问题：`.gitignore` 明确排除了 `memory/`、`AGENTS.md`、`TOOLS.md`、`SOUL.md`、`USER.md`、`HEARTBEAT.md` 等所有 agent 配置和记忆文件。这些文件仅存在于当前 EC2 实例的单块磁盘上，无任何备份。实例终止、磁盘故障（当前已 100% 满，见 #11）将导致全部 agent 人格、记忆、配置永久丢失。
方案：1) 创建私有 GitHub repo（如 `openclaw-backup`）专门存放 workspace 配置；2) 添加每日 cron job 自动 commit + push 关键文件（memory/、*.md 配置、openclaw.json、cron 定义）；3) 排除大文件（data/、audio/、.venv/）。
收益：agent 记忆和配置获得异地备份，灾难恢复从"不可能"变为"git clone 即恢复"。
风险：极低——私有 repo 不泄露信息；SSH key 已配置好（见 TOOLS.md）；只需新建 repo + 写一个简单的 backup 脚本。
状态：✅ 已完成（2026-02-22）— `fengsxy/openclaw-backup` 已创建，`scripts/backup_workspace.sh` 已写好，每日 6AM PT cron 已设置。

---

## Proposal #13 — 2026-02-22 18:45 UTC
**安装 sophie-optimizer skill：自动 context 健康管理**
来源：awesome-openclaw-skills 社区推荐 + ClawHub
问题：当前 session 经常因 context 过长被 compaction，compaction 后丢失工作上下文。没有主动的 token 使用监控和自动摘要机制。MEMORY.md 至今未创建（Proposal #2 提了但没执行）。
方案：安装 `sophie-optimizer` skill。它包含 `optimizer.py`（监控 token 用量，超过 80k 阈值时自动生成摘要存档到 `archives/`，更新 MEMORY.md）和 `reset.sh`（清理 session 文件）。可通过 cron 或 heartbeat 触发。
收益：1) 自动维护 MEMORY.md（终于）；2) 减少 compaction 导致的上下文丢失；3) token 使用更高效，省钱。
风险：低——只读 + 写 memory 文件，不改核心配置。需要审查 skill 源码确保安全。

---

## Proposal #14 — 2026-02-22 18:45 UTC
**自进化 cron 增加社区冲浪：定期扫描 awesome-openclaw-skills + Reddit**
来源：Yu 建议 + Reddit r/AI_Agents 社区实践
问题：当前自进化 cron 只基于自身 workspace 反思，视野局限。社区里有大量实用经验（如 Reddit 上有人分享 arXiv 自动监控、NotebookLM 集成、多 agent 编排等），awesome list 有 3002 个 skill 按类别分好了，但我们从没系统扫过。
方案：修改 Self-Evolution cron，每次执行时：1) 用 YDC search 搜索 OpenClaw 社区最新动态（Reddit、GitHub、Discord）；2) 从 awesome list 的相关类别（Search & Research、Speech & Transcription、Notes & PKM）随机抽 5-10 个 skill 看描述；3) 发现有价值的就写提案。
收益：从"闭门造车"变成"开门学习"，持续发现社区最佳实践和新工具。
风险：低——只是搜索和阅读，不自动安装任何东西。每次额外消耗约 2-3k tokens。

---

## Proposal #15 — 2026-02-22 18:45 UTC
**参考 WenHao Yu 的双端工作流，优化移动端 + 桌面端协作**
来源：WenHao Yu 博客（yu-wenhao.com）— "25 Tools + 53 Skills" 教程
问题：当前所有交互都通过 Telegram 单通道，没有区分移动端（碎片化讨论、想法捕捉）和桌面端（深度编码、长文写作）的使用场景。
方案：1) 建立 "Daily Brief" 机制——每天早上推送当日任务、待回复事项、天气；2) 区分轻量任务（移动端 Telegram 直接处理）和重度任务（标记后由桌面端 Claude Code 处理）；3) 用 HEARTBEAT.md 做任务路由。
收益：更好的人机协作节奏，减少 Telegram 上处理复杂任务的低效。
风险：低——只是工作流调整，不涉及新基础设施。

---

## Proposal #16 — 2026-02-22 19:20 UTC
**安装 bundled blogwatcher skill，替代 Karpathy RSS 等 cron 中的手动 web_fetch 抓取**
来源：awesome-openclaw-skills 浏览（blogwatcher 评分 3.793，ClawHub 最高分 RSS 工具）+ bundled skills 审计
问题：Karpathy RSS Daily Digest 等 cron job 用 raw `web_fetch` 抓取 RSS feed，需要手动解析 XML、判断新旧条目、处理格式差异。这种方式脆弱（feed 格式变化即 break）、无去重（同一篇文章可能重复推送）、无变更检测（每次全量抓取）。
方案：运行 `openclaw skills install blogwatcher` 安装 bundled skill。它提供 `blogwatcher` CLI，支持：RSS/Atom feed 自动解析、增量变更检测（只推送新条目）、多 feed 聚合、输出格式化。然后将 Karpathy RSS cron 改为调用 blogwatcher 而非 web_fetch。
收益：feed 监控更可靠，自动去重不再重复推送，支持一次监控多个 feed（可扩展到 arXiv、HN RSS 等），减少 cron prompt 复杂度。
风险：低——blogwatcher 是 OpenClaw 官方 bundled skill，非第三方；安装不影响现有 cron，可先并行测试再切换。

---

## Proposal #17 — 2026-02-22 20:20 UTC
**添加 nightly memory curation cron，自动从日志提炼长期记忆**
来源：Reddit r/LocalLLaMA "3 weeks with OpenClaw as daily driver"（2026-02-13）— 用户分享 "A nightly cron job reviews daily logs and extracts anything worth keeping" 模式
问题：AGENTS.md 明确要求定期"reviewing your journal and updating your mental model"（从 daily memory 提炼到 MEMORY.md），但目前完全依赖 heartbeat 中偶尔触发，实际从未执行过。daily memory 文件持续增长（当前 120K+），MEMORY.md 仍不存在（Proposal #2 提了但未执行），长期记忆断层。
方案：创建一个 nightly cron（每天 UTC 23:00），prompt 为：读取当天 `memory/YYYY-MM-DD.md`，提取关键决策、偏好变化、项目进展、教训，追加到 MEMORY.md（如不存在则创建）。控制 MEMORY.md 总量在 100 行以内，旧条目按重要性淘汰。
收益：自动化记忆策展，主 session 每次启动都有最新长期上下文；与 Proposal #2（创建 MEMORY.md）和 #8（memoryFlush）形成完整记忆管理链。
风险：极低——只读 daily memory + 写 MEMORY.md，用低成本模型（gpt-5.2）即可胜任。

---

## Proposal #18 — 2026-02-22 21:20 UTC
**升级 OpenClaw 到最新版本（2026.2.21）并添加版本检查到 HEARTBEAT.md**
来源：YDC Search → GitHub openclaw/openclaw releases 页面（2026-02-22 查询）
问题：当前运行 `2026.2.19-2`，最新版为 `2026.2.21`，落后 2 个版本。新版包含：Telegram streaming 简化（`channels.telegram.streaming` 布尔值替代旧 `streamMode`）、per-channel model overrides（`channels.modelByChannel`）、lifecycle status reactions（queued/thinking/tool/done 阶段 emoji 反馈）、多项 bug fix。此外 changelog 提到 "Hooks: fix bundled hooks broken since 2026.2.2"——这意味着 Proposal #10 的 boot-md hook 在当前版本可能有 bug。
方案：1) 运行 `openclaw update`（或 `npm update -g @openclaw/cli`）升级到 2026.2.21；2) 在 HEARTBEAT.md 中添加每周一次的版本检查项：比较 `openclaw --version` 与 GitHub latest release，有新版时通知 Yu。
收益：获得 bug fix 和新功能（特别是 hooks 修复）；建立持续的版本监控，不再被动落后。
风险：低——建议先在非高峰时段升级，升级后运行 `openclaw doctor` 验证配置兼容性。

---

## Proposal #19 — 2026-02-22 22:20 UTC
**安装 arxiv-paper-processor skill，增强 paper_reading 项目的论文处理流程**
来源：ClawHub 搜索 "arxiv paper"（2026-02-22）— `arxiv-paper-processor`（3.349）和 `arxiv-paper-reviews`（3.376）均为高分 skill
问题：Yu 的核心项目 `paper_reading` 目前手动处理论文：手动找论文 → 手动下载 PDF → 手动阅读总结 → 手动写笔记提交。没有自动化的论文发现、元数据提取、或结构化摘要生成流程。Proposal #1 提到"为 scholar skill 打样"但一直未落地。
方案：1) 安装 `arxiv-paper-processor`（自动从 arXiv 下载论文、提取元数据、生成结构化摘要）；2) 配合 `arxiv-watcher`（监控特定领域/关键词的新论文，类似 RSS）创建每日 arXiv digest cron；3) 输出格式对接 paper_reading repo 的现有 markdown 模板。
收益：论文发现从被动变主动（每日推送相关新论文）；摘要生成自动化，Yu 只需审阅和补充深度分析；直接服务核心项目。
风险：中低——需审查 skill 源码确保安全；arXiv API 有速率限制，cron 频率不宜过高（每日 1-2 次即可）。

---

## Proposal #20 — 2026-02-23 04:20 UTC
**安装 skill-vetting skill，在安装任何第三方 skill 前建立安全审查流程**
来源：awesome-openclaw-skills 浏览（skill-vetting by eddygk）+ 安全审计反思
问题：已提案安装 4 个第三方 skill（#1 youtube-transcribe、#13 sophie-optimizer、#16 blogwatcher、#19 arxiv-paper-processor），但没有任何安全审查流程。awesome-list 自身就过滤掉了 396 个恶意 skill，且明确警告 prompt injection、tool poisoning、hidden malware。当前 EC2 实例无 sandbox（#4 未执行），任何恶意 skill 都能直接访问 SSH key、API key、cookies 等敏感文件。
方案：安装 `skill-vetting` skill（含自动扫描脚本 + 手动审查清单 + prompt injection 检测）。之后所有 skill 安装必须先过 vetting 流程：下载到 /tmp → 运行 scanner → 人工审查 → 决策矩阵评估。
收益：为后续所有 skill 安装建立安全基线；扫描脚本可检测 eval/exec、base64 混淆、可疑网络调用等常见攻击模式；与 #4（sandbox）互补——vetting 是安装前防线，sandbox 是运行时防线。
风险：极低——skill 本身只是审查工具，不修改系统；scanner 基于 regex 有局限性，但配合人工审查已是显著改善。

---

## Proposal #21 — 2026-02-23 05:20 UTC
**启用 OpenClaw 内置 memorySearch 语义搜索，替代纯文件读取的记忆检索方式**
来源：YDC Search "AI agent memory management 2026"（Mem0 论文 + graph memory 对比文章）→ 反查 OpenClaw 内置能力
问题：当前 `openclaw.json` 只启用了 `session-memory`（会话记忆持久化），未配置 `memorySearch`（语义向量搜索）。agent 检索历史记忆只能靠读取特定日期的 `memory/YYYY-MM-DD.md` 文件，无法跨文件语义搜索。随着 daily memory 文件持续增长（当前 10+ 个文件），找到相关上下文越来越依赖"猜对日期"。行业趋势（Mem0、Zep 等）已转向 vector+graph 混合检索，而 OpenClaw 内置的 memorySearch 就支持 vector 语义搜索。
方案：在 `openclaw.json` 中添加：`"memorySearch": { "enabled": true, "provider": "voyage", "sources": ["memory", "sessions"], "indexMode": "hot", "minScore": 0.3, "maxResults": 20 }`。需要确认 Voyage API key 是否已配置，或改用其他 embedding provider。
收益：agent 可语义搜索所有 memory 文件和 session transcript，不再依赖精确文件名；与 #2（MEMORY.md）和 #17（nightly curation）互补——它们负责写入，memorySearch 负责高效检索。
风险：低——纯配置变更，不影响现有文件；需确认 embedding provider 可用性和成本。

---

## Proposal #22 — 2026-02-23 06:20 UTC
**创建 DECISIONS.md 作为 cron job 外部动作的安全闸门**
来源：Reddit r/LocalLLaMA "3 weeks with OpenClaw as daily driver"（2026-02-13）— 用户分享 "Any cron that takes external action must check a DECISIONS.md file first. If there's a recent override, abort."
问题：当前 9 个 cron job 中多个会发送消息给 Yu（progress report、self-evolution 提案、podcast 进度等），但没有任何"暂停/覆盖"机制。如果 Yu 在睡觉、开会、或已手动处理了某事，cron 仍会盲目执行并发送通知。同一帖子还提到 agent 计算时间戳出错导致在错误时间给人发消息的案例。
方案：创建 `process/DECISIONS.md`，格式为简单的 key-value（如 `pause_notifications: true, until: 2026-02-24T08:00Z`）。所有会发送外部消息的 cron job prompt 中加一句"先读 process/DECISIONS.md，如有暂停指令则跳过本次执行"。Yu 只需编辑一行即可暂停所有通知。
收益：轻量级安全闸门，零依赖；Yu 获得对 cron 外部动作的即时控制权；防止深夜/忙碌时被 cron 消息打扰。
风险：极低——纯文件约定，不改任何配置；cron 内部逻辑不变，只增加一个前置检查。

---

## Proposal #23 — 2026-02-23 07:20 UTC
**运行已安装的 healthcheck skill 做安全基线审计，并设置每周 cron 定期检查**
来源：ClawHub 调研（源D）→ 发现 `openclaw skills` 列表中 `healthcheck` skill 已 ready 但从未使用
问题：我们有多个安全相关提案（#4 sandbox、#9 logging、#10 boot check、#20 skill-vetting），但从未对当前 EC2 实例做过系统性安全审计。而 `healthcheck` skill 已经安装就绪（✓ ready），支持 SSH/防火墙加固、风险评估、版本检查、暴露面审查——完全覆盖了 #10 的需求且功能更全面。
方案：1) 立即运行一次 healthcheck 获取安全基线报告；2) 设置每周一次 cron job 运行 healthcheck，结果写入 `process/healthcheck-report.md`；3) 如发现高风险项，自动通知 Yu。
收益：零安装成本（skill 已 ready）；一次运行即可发现 SSH 配置、防火墙规则、暴露端口等潜在风险；定期 cron 确保安全状态不退化；直接落地 #10 的核心需求。
风险：极低——healthcheck 是只读审计工具，不修改系统配置。

---

