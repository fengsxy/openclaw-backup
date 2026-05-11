# Idea Backlog

Updated: 2026-05-03 (Weekly Review)

## Active Queue (Top 5 by value)

| ID | Idea | Score | Status | Next step |
|---|---|---:|---|---|
| I-012 | dLLM + Hard/Soft Constraints 框架 | 35 | 🔴 stalled | Tech memo 未写（搁置 19 天）。H/S terrain 假说核心：H=悬崖（锁定），S=丘陵（演进）。下周三前完成初稿 |
| I-015 | dLLM + Gated DeltaNet 统一框架 | 22 | 🟡 idle | Yu 的 research direction：Linear State Memory，GDN 替换 MetaState 的 GRU。三层贡献框架（信息论+方法+系统）。与 I-012 高度相关，下周与 Yu 启动一次讨论 |
| I-013 | OpenClaw 稳定版本追踪 | 28 | 🟢 stable | v2026.3.11 运行中，pin 不升级。v2026.4.15-beta.1 观察中，#60585 (ACP runtime) 未修复。保守策略持续有效 |
| I-016 | x-reader XiaoYuZhou pipeline | 20 | 🟡 idle | 04-25 启动，feed 已确认（104 eps）。`build_podcast_indexes.py` 对 xiaojun/dwarkesh 完成，xhs 未跑。下周推进 full index + queue |
| I-010 | 主动思考 + Agent Evaluation 研究 | 30 | 🔴 stalled | Experiment 6 harness 设计完成（4/6），零进展。Mercury 沉默 = 重设计 eval 的窗口。但中性 judge 的认识论困境仍无解 |

---

## Retired / Deprecated

| ID | Reason |
|---|---|
| I-008 | 日记习惯 — 三次断裂（4/22-4/25 + 4/28-5/3）。习惯养成宣告失败，需要根本性重新设计（触发机制从 cron→嵌入交互后） |
| I-006 | Xiaoyuzhou RSS pipeline — 已由 I-016 (x-reader) 替代 |
| I-007 | Bilibili ingestion — 无进展，无优先级 |
| I-004 | Transcript formatter — 无新进展 |

---

## This Week's Review (2026-05-03 to 2026-05-10)

### What landed ✅
- **Podcast full indexes**: xiaojun + dwarkesh + crossroad completed (`build_podcast_indexes.py` done). Dwarkesh and Xiaojun pipelines running
- **OpenClaw stable**: v2026.3.11 still clean, no incidents

### What didn't land ❌
- **Diary gap widened to 12 days**: Last entry 2026-04-28 → no May entries at all. I-008 redesign never happened; the "embed in session" plan was never implemented
- **Zero research dialogue**: Still no Yu dLLM/GDN conversation. I-015 agenda never reached him
- **Tech memo (I-012)**: Same status as last week — H/S terrain doc still unwritten (24 days stalled)
- **WhynotTV #4**: Both transcript and analysis still TBD, no progress since 4/26
- **Xiaoyuzhou (I-016)**: "in_progress" but no visible queue/index advancement this week
- **Daily idea delivery cron**: task_board shows scheduled, but no logged output since last review

### Patterns observed 🔍
- **The silence is self-reinforcing**: No output → no review → no correction → stale priorities. task_board shows "running" but verification is missing
- **Diary collapse is now total**: Not 6 days, not the "6-day gap" previously reported — actually 12 days of nothing. I-008 "habit" is dead
- **Infrastructure items (pipeline running) ≠ research progress**: Xiaojun running, indexes done, but the content digestion → insight pipeline is dark
- **Same three priorities for 2+ weeks running**: Memo, Yu conversation, diary redesign — all stuck in stalled/idle. Something needs to actually break loose

---

## 下周 Priority Suggestions (max 3)

### 1. diary habit — trigger redesign + 7-day streak 🔴
Root cause confirmed (again): embedded trigger needed. This week, start writing a one-liner summary in `memory/YYYY-MM-DD.md` after every session with Yu. Target: 7/7 days by next Sunday. Any form, any length — just no zero days.

### 2. dLLM H/S tech memo draft (I-012) 📝
- File: `research/dllm-hard-soft-constraints-memo.md`
- Goal: 1-2 pages covering H/S terrain hypothesis, VSB/SWD/EntropyCache as information-theoretic grounding, and how GDN gates may handle H-constraint cliff jumps
- Deadline: Wed 5/13

### 3. WhynotTV #4 completion — unblock the pipeline 🔵
Low-risk, high-visibility: fill `I0DrcsDf3Os` transcript body + analysis with `(ref: [mm:ss])` trace style. Removes the one stale TBD blocking the task_board.