# Roadmap (长期 / 后期)

> 起草:2026-05-02  
> 状态:全部为 **future-track**,当前 MVP 不阻塞。这里只记方向,具体设计待启动各项时再展开。

---

## A. 长期路线(3 条)

### A1. Search-result 本地落盘 → coding agent 的 memory + LLM wiki

把每次有价值的搜索结果(snippet + URL + version + metadata)持久化为本地 store,供后续 coding agent 复用——既是 host LLM 的 long-term memory(按相关度召回),也是人能直接读的 wiki。

**未决**:何时落 / 谁去重 / TTL / 隐私边界(用户私域 vs 项目通用)。

### A2. Agent orchestration skills

当 query 复杂到需要"分组并行 + 中间裁判 + 二次回填"时,提供 orchestration skill 让 **host 编排** sub-agent,而不是 wrapper 内部硬编 graph。

- 借鉴 §1.1 / §7 DeerFlow 实证
- Wrapper 仍只负责单次 fan-out + RRF + schema 归一(§2.11 中间层职责锁定)
- 反例:graph framework 在 Claude Code 上过度工程(§2.12 已论证)

### A3. 原生多模态识别 plugin

让 host LLM 直接处理搜索结果里的图 / 视频 / 音频,不只看 alt / transcript。

**硬约束**:**plugin 传给 host 的信息不能丢失**——wrapper 不做 lossy 转码,直接 pass-through host 自带 vision API。多模态信号细分见 §2.4。

---

## B. 巨型 co-install 候选(难度高,不绑死 MVP)

两个独立项目级别的工程量,做成了价值极大,只作 co-install 选项。

### B1. Context search engine(代码层面)

为**本地 / 用户私有 codebase** 建立语义 + 符号 + 拓扑搜索——相当于"自家的小型 Sourcegraph"。

- 和 §9 SG 公网层互补:SG 只查 OSS,B1 才能触达用户自己的 monorepo
- 技术依赖:SCIP indexer / zoekt(都是 OSS,可自托管)
- 难点:各语言 indexer 成熟度参差(§9.2.4)、增量索引 / file watcher / IDE 集成、与 LSP / tree-sitter / ast-grep 的边界

### B2. CUA 端到端测试与内容获取(CLI + Browser + GUI)

**Computer Use Agent**——让 search-agent 直接驱动 OS 级交互来获取 / 验证内容,弥补 API + scraper 都打不进去的封闭系统。

| 层 | 现状 | 缺口 |
|---|---|---|
| **CLI** | §2 Playwright CLI / yt-dlp / gh / Context7 CLI 已部分覆盖 | 协调编排尚无 skill |
| **Browser** | Playwright / Firecrawl(§1.4)已有脚本式 | **真正的 CUA = 模型直接看截图 + 点鼠标**,不是预定义流程 |
| **GUI** | 空 | OS 原生窗口、登录态保持、需键鼠的私域交互(LinkedIn / 内部 wiki / 桌面 app) |

**核心场景**:端到端跑通一个 API 并截图验证 / 抓强反爬 / 验证 SDK example 真能运行。

**难点**:Vision + action loop 延迟和成本仍高(Claude Computer Use / OpenAI Operator 早期)、安全模型(全权 OS 控制 = 巨大 attack surface)、跨 OS / 分辨率 robustness 全靠 LLM 自己。

---

## 关联

- [`domain-knowledge.md`](./domain-knowledge.md) — 已锁定的所有判断 / 实证(§1-§9)
- [`sources.md`](./sources.md) — Phase 1 source 清单(20 ★ + co-install + ground-api)
