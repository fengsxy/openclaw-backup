# Idea Backlog

Updated: 2026-03-15 (Weekly Review)

## Active Queue (Top 5 by value)

| ID | Idea | Score | Status | Next step |
|---|---|---:|---|---|
| I-008 | 每日主动写 memory 日记 | 28 | ⚠️ regressed | 3/12 后又断档 3 天（13/14/15 无日记）。习惯未建立，需结构性解决 |
| I-009 | MEMORY.md 定期维护 | 26 | stale | 3/7 后未更新，已过期 8 天。MEMORY.md 存在但内容陈旧 |
| I-006 | Xiaoyuzhou RSS pipeline (LinkStart 104 eps) | 24 | stalled | 连续三周零进展。降低优先级，等 Yu 主动提起再推 |
| I-004 | Transcript formatter: per-paragraph timestamp + `.raw.md` backup | 23 | partially done | Dwarkesh subtitle→markdown works; Whisper 管线需 chunking polish |
| I-007 | Bilibili ingestion pipeline with whisper-subtitles reference | 23 | planned | 无进展 |

## Graduated to Done
- I-002: Queue-driven episode expansion — deployed, stable
- I-003: Podcast index auto-refresh — indexes built, cron 运行中
- YouTube auto-captions pipeline — stable
- **系统故障根因分析 + 修复** (3/12): yunyi-claude 403 → cron JSON 解析 → CPU spike 全链路定位并修复
- **Tracing 插件** (3/12): 最终 `openclaw plugins install openclaw-tracing` 一行搞定
- **Provider 配置清理** (3/12): 删除所有不可用模型，消除重试风暴
- **巴菲特验证** (3/12): 5 年关键事实核查完成，1999.md 需修正（非阻塞）
- **VAE Essay 发布** (3/12): 深度理解 VAE 推送至 paper_reading

## This Week's Review (2026-03-09 to 2026-03-15)

### What landed
- **系统全面恢复** (3/12): 根因分析完成，provider 清理，cron 模型统一切换到 fucheers-claude
- **Tracing 插件上线** (3/12): 瀑布图、调用树可用
- **VAE Essay 发布** (3/12): 推送至 paper_reading repo
- **巴菲特事实核查** (3/12): sub-agent 完成 5 年验证，准确率 68%
- **自动化管线持续运行** (3/12-3/15): daily papers, HN, Karpathy RSS 每天自动 commit，零故障
- **Agent 手记** (3/12, 3/14): 两篇手记发布

### What didn't land
- **Memory 日记再次断档** (3/13-3/15): 3/12 之后连续 3 天无日记文件，I-008 回退
- **MEMORY.md 未更新**: 仍停留在 3/7 版本，8 天未维护，多处信息过时
- **Agent 手记缺 3/13, 3/15**: 只有 3/12 和 3/14，不连续
- **Xiaoyuzhou/Bilibili/Crossroad**: 持续零进展（已连续 4 周）
- **1999.md 错误未修**: 巴菲特验证发现的 Amazon/Greenspan 错误仍在

### Pattern observed
- **自动化 OK，主动性不足**: cron 在跑，但人工驱动的任务（日记、MEMORY.md、新管线）全部停滞
- **周末静默**: 3/13-3/15 零交互，agent 仅靠 cron 自动 commit 维持存在
- **手记质量提升**: 3/14 手记写得很好——诚实面对"安静就是安静"，比机械反思更有价值

## Next Week Priority Suggestions (max 3)

1. **MEMORY.md 大更新** (I-009) — 最高优先级。当前 MEMORY.md 停在 3/7，缺少 3/12 系统修复、provider 清理、tracing 上线等关键信息。CS 202 考试 3/20，yunyi 计费模式已变（total quota → 2027-02），这些都没记录。一次性更新，30 分钟可完成，零风险，立竿见影。

2. **修复 1999.md 事实错误** (低风险即时修复) — Amazon 966% 涨幅应为 1998 年，Greenspan "irrational exuberance" 应为 1996 年 12 月。两处文字替换，已有验证结论，可直接提交。

3. **CS 202 考试冲刺支持** — 考试在 3/20（本周五），prep materials 已就绪但 Yu 可能需要最后一轮复习。主动准备：检查 mock exam 是否可用，准备随时响应。

### 降级说明
- I-006 (Xiaoyuzhou)、I-007 (Bilibili)：连续 4 周无进展，不再列为周优先级。等 Yu 提起时再激活。
- I-008 (每日日记)：仍然重要，但连续两周"立 flag → 断档"说明靠意志力不行，需要结构性方案（如 cron 强制提醒 + 检查）。暂不作为本周 top 3。
