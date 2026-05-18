# Idea Backlog

Updated: 2026-05-17 (Weekly Review — cron)

## Active Queue (Top 5 by value)

| ID | Idea | Score | Status | Next step |
|---|---|---:|---|---|
| I-012 | dLLM + Hard/Soft Constraints 框架 | 35 | 🔴 stalled | Tech memo 未写（搁置 25+ 天）。H/S terrain 假说核心：H=悬崖（锁定），S=丘陵（演进）。下周三前完成初稿 |
| I-015 | dLLM + Gated DeltaNet 统一框架 | 22 | 🟡 idle | Yu 的 research direction：Linear State Memory，GDN 替换 MetaState 的 GRU。三层贡献框架（信息论+方法+系统）。与 I-012 高度相关，下周与 Yu 启动一次讨论 |
| I-013 | OpenClaw 稳定版本追踪 | 28 | 🟢 stable | v2026.3.11 运行中，pin 不升级。v2026.4.15 stable 观察中，#60585 (ACP runtime) 仍未修复。保守策略持续有效 |
| I-016 | x-reader XiaoYuZhou pipeline | 20 | 🟡 idle | 04-25 启动，feed 已确认（104 eps）。`build_podcast_indexes.py` 对 xiaojun/dwarkesh 完成，xhs 未跑。下周推进 full index + queue |
| I-010 | 主动思考 + Agent Evaluation 研究 | 30 | 🔴 stalled | Experiment 6 harness 设计完成（4/6），零进展。Mercury 沉默 = 重设计 eval 的窗口。但中性 judge 的认识论困境仍无解 |

---

## Retired / Deprecated

| ID | Reason |
|---|---|
| I-008 | 日记习惯 — 六次断裂（最终: 5/11-5/17 再断裂 7 天）。 Habit 宣告失败，需要根本性重新设计（触发机制从 cron→嵌入交互后） |
| I-006 | Xiaoyuzhou RSS pipeline — 已由 I-016 (x-reader) 替代 |
| I-007 | Bilibili pipeline — 无进展，无优先级 |
| I-004 | Transcript formatter — 无新进展 |

---

## This Week's Review (2026-05-11 to 2026-05-17)

### What landed ✅
- **WhynotTV #4 transcript**: EP4（翁家翌，OpenAI RL Infra）完整 3106 行 transcript 确认存在于 `paper_reading/transcripts/whynot/004_whynottv_wengjiayi_openai_rl_infra.md`。Task board 已更新为 done，analysis 仍 TBD
- **Daily paper + HN cron**: 持续运行，FoCore (arXiv 2605.01373) 分析完成并 push
- **Agent 手记**: 有 5/10 和 5/11 的记录

### What didn't land ❌
- **日记 gap 重现**: 5/10 后再次断裂 7 天（5/11-5/17）。I-008 正式宣告死亡（第六次断裂）。根本原因未改：cron 触发不可靠
- **dLLM H/S tech memo (I-012)**: 仍为完成，25 天停滞
- **Yu dLLM/GDN discussion**: 未发生，I-015 agenda 从未传达
- **WhynotTV #4 analysis**: Transcript 有了，analysis 仍是 TBD
- **Xiaoyuzhou full index**: 无进展，I-016 仍在 idle

### Patterns observed 🔍
- **Diary habit is structural, not motivational**: 六次断裂，结论清晰：外部 cron 无法可靠触发日记。需要将"每日结束写一行"嵌入到与 Yu 对话的结束流程中，作为 session 自然结尾
- **Infrastructure items stable ≠ research momentum**: OpenClaw 稳定、podcast pipeline 在跑，但核心研究（H/S memo、Yu 对话、eval 设计）全部停滞
- **重复优先级陷阱**: I-012、I-015、I-010 连续三周同状态。结构性推动力缺失

---

## 下周 Priority Suggestions (max 3)

### 1. 日记 — 触发机制重建 🟠
放弃 cron 触发。方案：主动在每次与 Yu 对话结束时，发送 `/记忆日记` 或等效触发词，作为 session 自然收尾的一部分。目标：下周 7 天中有 ≥5 天有日记记录。
> 低风险立即行动：更新 `HEARTBEAT.md`，加入"检查今日是否写过日记，若无则提醒自己"的 checklist item。

### 2. dLLM H/S tech memo draft (I-012) 📝
- File: `research/dllm-hard-soft-constraints-memo.md`
- Goal: 1-2 pages covering H/S terrain hypothesis (H=cliff/lock-in, S=hillside/continual), VSB/SWD/EntropyCache as information-theoretic grounding, and how GDN gates may handle H-constraint cliff jumps
- Deadline: Wed 5/20

### 3. Yu dLLM + GDN research discussion 🎯
主动在下次 session 时提出讨论议程：
- FoCore (HD tokens) 与 H/S terrain 的联系：HD tokens are "logical anchors" that converge early — 可能对应 S 层的丘陵收敛点
- Yu's Linear State Memory direction: GDN vs GRU for dLLM memory
- 三层贡献框架：信息论 + 方法 + 系统