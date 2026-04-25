# Search Harness — 上层架构

> **目的**:把对话中已经达成共识的设计决策固化成图与概念,**不含代码、不含库具体 API**,只保留塑造架构的"必要底层选择"。
>
> **读者**:你自己 + 未来可能加入的协作者。所有"实现细节级"的内容(Pydantic schema / 命令参数 / 文件路径) **不在这里**。

---

## 1. 核心定位

| 它是 | 它不是 |
|---|---|
| **被 coding agent 调用的搜索 harness**(Claude Code / Cursor / Codex CLI 等) | 独立 agent 产品(对标 Perplexity / DeerFlow) |
| 一个**零配置即可工作**的 CLI 二进制 + 一组 skills | 一个 MCP server(原因见 §3.6) |
| **垂直化、为 coding 优化**的检索 + ranking 层 | 通用 web search 的另一个 wrapper |
| 输出**结构化 citation**,让 host LLM 综合 | 输出 synthesized answer |

---

## 2. 主架构图

```
┌──────────────────────────────────────────────────────────────────┐
│                              User                                 │
└────────────────────────────┬─────────────────────────────────────┘
                             ↓ 自然语言任务
┌──────────────────────────────────────────────────────────────────┐
│   Host LLM   (Claude Code / Cursor / Codex / Gemini CLI / …)     │
│                                                                   │
│   • 唯一的推理者:决定调谁、何时调、综合所有结果                 │
│   • 通过 bash 调本地 CLI / 通过 MCP 调外挂                       │
└──┬──────────────┬─────────────────────────┬─────────────────────┘
   │ bash         │ bash                    │ MCP
   ↓              ↓                         ↓
┌──────────────┐ ┌──────────────────┐ ┌─────────────────────────┐
│search-harness│ │ @playwright/cli  │ │ DeepWiki MCP            │
│  (CLI 二进制)│ │  + browser/skill │ │  (Cognition,免费)       │
│  + skills/   │ │  (web fallback)  │ │  ─ 仓库深度理解 Q&A     │
│              │ │                  │ │                         │
│ ★ 主战场     │ │ ★ 仅作 fallback  │ │ + 其他可选 MCP(用户装) │
└──────┬───────┘ └──────────────────┘ └─────────────────────────┘
       │
       ↓ (CLI 内部流水线 — 见 §5)
┌──────────────────────────────────────────────────────────────────┐
│                   SEARCH HARNESS — 内部流水线                     │
├──────────────────────────────────────────────────────────────────┤
│                                                                   │
│  ① Heuristic Query Gate    (regex/keyword,0-cost)               │
│      ─ 决定"哪些 Tier 源相关",粗筛,不做精细 routing             │
│                                                                   │
│      ↓                                                            │
│                                                                   │
│  ② Parallel Fan-out        (per-source timeout,允许部分失败)    │
│  ┌────────┬────────┬────────┬────────┬────────┐                 │
│  │ Tier 1 │ Tier 2 │ Tier 3 │ Tier 4 │ Tier 5 │                 │
│  │原生 API│AI 二级 │通用Web │准多模态│真多模态│                 │
│  │ (free) │  源    │ search │transcript│ video │                 │
│  ├────────┼────────┼────────┼────────┼────────┤                 │
│  │默认 on │默认 on │ opt-in │默认 on │ opt-in │                 │
│  │零 key  │零 key  │需 key  │零 key  │需 key  │                 │
│  └────────┴────────┴────────┴────────┴────────┘                 │
│                                                                   │
│      ↓                                                            │
│                                                                   │
│  ③ RRF Fusion + URL Dedup                                        │
│      ─ 跨源结果按 reciprocal-rank 融合,去重                     │
│                                                                   │
│      ↓                                                            │
│                                                                   │
│  ④ Output Layer                                                  │
│      • JSON to stdout    ─ 索引 + snippet + citation(token 极简)│
│      • 全文落本地文件     ─ workspace/.cache/<query-hash>/        │
│      • host 用 grep/jq/head 按需读                               │
└──────────────────────────────────────────────────────────────────┘

       ↓ (host 收到 JSON,二次决策)

┌──────────────────────────────────────────────────────────────────┐
│  Host LLM 第二轮:                                                │
│   • 看 candidates → 决定要不要深挖某个 (调 DeepWiki MCP)          │
│   • 看 query     → 决定要不要 watch 视频 (调 read-video skill)    │
│   • 看任务       → 决定要不要操作 web (调 playwright CLI)         │
│   • Synthesize all  ← 综合发生在 host,不在 harness               │
└──────────────────────────────────────────────────────────────────┘
```

---

## 3. 核心设计原则(8 条)

### 3.1 Host LLM 是唯一推理者
Harness 内部**绝不内置 LLM**。所有"调谁 / 何时调 / 怎么综合"的决策都在 host(Claude/GPT/Gemini)那边发生。Harness 只暴露**确定性工具**。

### 3.2 零配置默认,渐进升级
- **零 key 即可全功能跑通**(Tier 1 + Tier 3 + Tier 4)
- GitHub Personal Access Token 是**唯一推荐的"30 秒升级"**(60→5000 req/hr)
- Tavily / Exa / Brave / Gemini 等都是**纯增益 buff**,不配也不影响主路径

### 3.3 数据源分 5 Tier,不是平铺
不同 Tier 有不同的**触发条件 / 成本曲线 / 失败容忍度**。不要一视同仁地查所有源。详见 §4。

### 3.4 小 N 用 flooding,不用 routing
**4-6 个垂直源时,无脑并发优于智能路由**(经典 IR 文献 + LLM 时代实证已收敛)。
- 用 0-cost 启发式 gate 做粗筛(不是 routing)
- 并发调用,per-source 3s 超时
- 部分失败不阻塞返回

> 等到源数量 >15 才考虑学过的 router(Self-RAG / Search-R1 风格)。当前不需要。

### 3.5 输出 = 结构化 citation,从不 synthesize
- harness 永远不返回"答案",只返回**带 citation 的候选 + 元数据**
- Synthesis 是 host LLM 的工作 — 它有完整 user context,看得到原文,损耗最小
- Citation 必须可定位:`repo@commit:path:Lstart-Lend` / `arxiv:id` / `url+anchor`

### 3.6 CLI + Skills,不是 MCP server
**经过实证修正**(Anthropic / Microsoft 自己也在调向):
- MCP tool schema 永久占 context,N 个 tool 是 N × 几千 token 的固定税
- Skills 是 progressive disclosure,只在相关时载入
- bash + jq/grep 是 30 年训练数据基底,模型极熟
- 多步组合走管道,中间结果不进 LLM context

→ 我们走 **CLI 二进制 + skills/ markdown** 路线。MCP 仅在**外挂**(Playwright / DeepWiki)上,因为那是别人维护的成熟 server,不是我们暴露给 host 的接口。

### 3.7 同级安装 (co-install),不要 bundle
**外挂工具用户装在 host 配置里,和我们 harness 平行**:
- `@playwright/cli` 由 Microsoft 维护
- DeepWiki MCP 由 Cognition 维护
- 我们一行不写,文档里推荐"建议同时安装"

→ 我们的代码库专注 4 件事:**source adapters + gate + fusion + skills**。

### 3.8 失败永远优雅降级
- Tier 4/5 失败 → 自动 fallback 到 Tier 3
- 任意 source 超时 → 不阻塞,返回 partial 结果
- 缺 API key → 跳过对应 source,不报错(仅 log)
- 永不让一条死链拖死整个 query

### 3.9 Adapter vs Skill 严格分边界

**adapter 的存在意义是"web_search 替代不了它的 ranking/字段",不是"它内容值得"。**

| 触发条件 | 处理方式 |
|---|---|
| 有特殊 ranking 信号(stars/votes/citation/downloads) | **写 adapter** |
| 有结构化字段(license/version/lang/pushed_at) | **写 adapter** |
| 已是免费 MCP 服务 | **co-install,不写代码** |
| 仅是"agent 不知道这个站存在" | **写进 skill 的 cross-reference 段** |
| Web search 完全替代不了(login wall / API only) | **adapter 或 drop** |

→ Skill 解决 **awareness 教学**,adapter 解决 **能力补全**——两者互补,不重复。

**实际筛选结果在 [`adapter-vs-skill.md`](./adapter-vs-skill.md)**:Tier 1 收敛到 **12 个核心 adapter**,数十个 site 转为 skill-only。

---

## 4. 五层数据源(Tier 模型)

| Tier | 类型 | 触发 | 典型 latency | Token 成本 | SNR | 形态 |
|---|---|---|---|---|---|---|
| **1** | 原生垂直 API(零 key 优先) | 默认 on | <1s | 极低 | ~1.0 | **12 个核心 adapter**(见 [adapter-vs-skill.md §2](./adapter-vs-skill.md)) |
| **2** | AI-augmented 二级源 | 默认 on | 1-3s | 低 | ~0.9 | 主要走 **co-install MCP**(DeepWiki / Context7 / Ref),少量 adapter |
| **3** | 通用 web search API | opt-in(需 key) | 1-3s | 中 | ~0.7 | Tavily / Exa / Brave / Serper(任选,薄 wrapper adapter) |
| **4** | 准多模态(平台已文本化) | 默认 on | 1-2s | 低 | ~0.6 | YouTube transcript adapter / talks 索引 |
| **5** | 真原始多模态(自己处理) | opt-in(需 Gemini key) | 10s+ | 高 | ~0.3 | `read_video` skill 调 Gemini |

**关键设计**:
- Tier 1/2/4 默认启用且零成本 → **保证零配置体验完整**
- Tier 3/5 是 opt-in 增益 → **buff 思路**(用户配 key 才生效,不配也不影响主流程)
- 任何 query 都不会同时触发所有 Tier:gate 决定相关 Tier 集
- **大量数据源不在任何 Tier**——它们是 **skill-only**(教 agent 用其 web_search + `site:` 找),见 [adapter-vs-skill.md §5](./adapter-vs-skill.md)

---

## 5. 调用流程(以一次 query 为例)

```
User: "帮我找一个 Rust 写的实时分析数据库"

  ↓
Host LLM (Claude in Claude Code):
  "这是 find-repo 任务,我应该调 search-harness。"
  → bash: $ search-harness find-repo "real-time analytics database" \
            --language=rust --json
  ↓
[Harness 内部]

  ① Heuristic Gate:
     query 含 "database" + "rust" → 触发 Tier 1 (GitHub) + Tier 2 (DeepWiki) + Tier 4 (talk transcripts)
     query 无明显时效词 → 不触发 HN/Reddit recent 模式
     query 无学术词 → 不触发 arXiv

  ② Parallel Fan-out (3 路并发):
     ├ GitHub Search API → top 10 by stars + activity
     ├ DeepWiki batch → 对预知的 candidate(ClickHouse-rs 等)拿 wiki 摘要
     └ YouTube transcript search → "rust analytics database talk"

  ③ Fusion:
     RRF 合并 → URL dedup → 加 Tier 元数据
     输出 candidates: [{repo, stars, last_commit, license, deepwiki_summary, video_url?}]

  ④ Output:
     JSON 到 stdout(每条 ~200 token)
     全文落 ./.cache/<hash>/   (host 想看再 read_file)

  ↓
Host LLM 看到 JSON,二次决策:
  "前 3 个候选我不熟,深挖 ClickHouse-rs 和 Materialize 这两个"
  → 调 DeepWiki MCP: ask_question("ClickHouse-rs 的 query engine 怎么实现?")
  → 调 DeepWiki MCP: ask_question("Materialize 和 ClickHouse-rs 的 trade-off?")

  ↓
Host LLM 综合 3 路信息(harness JSON + 2 个 DeepWiki 答案)→ 写出推荐
```

**这个流程里 host 主动调了 3 次工具(harness × 1 + DeepWiki × 2),全在 host context 里编排,harness 内部不知道也不需要知道。**

---

## 6. 工具表面(给 host 看到的 API 形状)

> 以下是**概念形状**,不是命令行细节。具体参数延后到 implementation 阶段定。

### 6.1 search-harness CLI(我们主战场)

```
search-harness <vertical> <query> [filters] [--json]

verticals:
  find-repo       仓库发现 + ranking
  ground-api      官方文档锁定 + 抽取
  search-paper    论文搜索 + 引用图
  find-talk       会议/视频/播客发现(Tier 4)
  read-video      调用 Gemini 看视频(Tier 5,opt-in)
  read-image      vision 看图(也可让 host 自己做)
  search-web      通用 web search(Tier 3 fallback)
  search-discuss  HN/Reddit/SO 综合(社区口碑)
```

每个 vertical 是一条**独立 pipeline**,内部按 §5 走 gate→fan-out→fusion→output。

### 6.2 Skills 目录(教 host 何时/如何用)

```
skills/
  find-repo/SKILL.md       何时用 find-repo,如何串联 DeepWiki + cross-reference HN/Reddit/中日社区
  ground-api/SKILL.md      防 API 幻觉的标准流程,优先 Context7,fallback web_search 官方 docs
  search-paper/SKILL.md    论文综述的递进 query 模式 + cross-reference 实验室博客
  find-talk/SKILL.md       多模态 fallback 决策
  search-discuss/SKILL.md  社区口碑 cross-reference(HN/Reddit/中日/X 等 site awareness)
  ...
```

**Skills 承担两件事**:
1. **路线图**:教 host 何时调哪个 adapter,串联 co-install MCP
2. **Site awareness 教学**:对每个 vertical,列出 web_search 应该额外尝试的 `site:` 清单——把数十个 skill-only 站集中暴露给 agent

→ Progressive 加载,只在相关 query 触发。详见 [adapter-vs-skill.md §5.2 样板](./adapter-vs-skill.md#52-skill-文件的样板形态)。

### 6.3 外挂(co-installed,不归我们)

| 外挂 | 触发场景 |
|---|---|
| `@playwright/cli` + browser skill | host LLM 判断需要操作 web app / 抓取 JS-heavy 页面 |
| DeepWiki MCP | host LLM 判断需要深度理解某个 repo |
| (用户自己装的其他 MCP) | 由用户/host 决定 |

---

## 7. 配置与渐进升级

```
Quick Start (零 key,5 分钟跑起来)
  ├ 装 search-harness CLI
  ├ 装 @playwright/cli(可选,但推荐)
  └ Claude Code 配置加 DeepWiki MCP(可选)
  
  → 已经覆盖 80% coding 场景

Recommended (1 个 key,30 秒)
  └ 加 GitHub Personal Access Token(GitHub Tier 1 提速)
  
  → 覆盖 90%

Power User (多 key,按需加)
  ├ Tavily / Exa / Brave 任选 1(Tier 3 web search)
  ├ Gemini API(Tier 5 视频理解)
  └ Reddit OAuth(Reddit Tier 1 提速)
  
  → 覆盖 99%
```

---

## 8. 边界:harness **不做**的事

| 我们不做 | 由谁做 |
|---|---|
| Synthesis / 写答案 | Host LLM |
| 用户对话 / UI | Host(Claude Code 等)的 UI |
| 内部 LLM 推理 | — 全部转嫁给 host |
| Repo 深度理解 | DeepWiki MCP |
| 浏览器操作 | @playwright/cli |
| 本地代码理解(symbol/repo map) | Aider / Serena / Continue |
| 长视频生成 / TTS / 音频生成 | 不在 search 范畴,不做 |
| 用户认证 / 多租户 / 计费 | OSS 项目,不需要 |
| 持久会话 / Memory | Host 处理 |

→ **harness 的边界严格收窄到"垂直化检索 + ranking + citation"**,其他全部外推。

---

## 9. 失败 / 降级原则

```
任何 source 失败 → log + 跳过,不抛 exception
任何 Tier 失败 → 自动 fallback 到上一级 Tier
任何 key 缺失 → 静默禁用对应 source,主路径继续
任何 query 没有 candidate → 返回空 JSON 数组 + reason 字段(不报错)

per-source timeout 默认 3s
overall query timeout 默认 15s
```

→ **永远返回有效 JSON,即使是空的**。让 host 知道发生了什么,自己决定要不要换策略。

---

## 10. 未决设计点(留给后续讨论)

**用户当前正在思考的开放问题**(暂未决):
- [ ] **Router 设计**——gate 层的精确决策逻辑、决策粒度
- [ ] **Long-tail 任务路由**——多模态(Tier 4-5)、browser(Tier 5)的触发条件、降级策略
- [ ] **Adapter / skill 的精细化平衡**——边界已定(§3.9 决策树),但具体 case 还需 case-by-case
- [ ] **Skill 设计粒度**——每个 vertical 一个,还是按"任务模式"分?skill 文件的篇幅、imperative tone 力度

**其他未决**:
- [ ] Cache TTL 策略(各 source 时效性差异大)
- [ ] Citation schema 在不同 Tier 间如何统一(repo / paper / video / web 的最小公共集)
- [ ] Tier 3 的"哪个 web search backend 默认"(Tavily vs Exa vs Brave)
- [x] ~~sources.md 里你将筛选保留哪些 source,直接决定 Tier 1/2 的 adapter 范围~~ → **已通过 [adapter-vs-skill.md](./adapter-vs-skill.md) 解决**
- [ ] 可选 adapter(libraries.io / OpenReview / HF Hub / CVE 等)是否进 Phase 1
- [ ] 是否需要某种 "result memory"(同 query 第二次触发不重复 fan-out)—— **gbrain 是天然候选**(见 domain-knowledge.md §1.8)
- [ ] 如何让 host LLM 知道某 vertical 的"何时不该用"(skill 描述里 explicit vs 隐式 prompt)

---

## 附录 A:关键决策的引用回溯

| 决策 | 来自对话哪一段 |
|---|---|
| CLI + skills 而非 MCP | "Anthropic 自己已经从 MCP 路线打脸"(Code execution with MCP,98% token reduction) |
| Flood + RRF 而非 routing | 经典 IR 文献(ReDDE/SUSHI)+ MindSearch / LangChain 共识 |
| 不内置 LLM | "harness 不应该重复宿主已有的 LLM 能力" |
| Tier 模型 | 多模态 SNR 分析 + "long-tail buff" framing |
| Playwright CLI 而非 browser-use | 4× token reduction + Microsoft 自己的转向 |
| DeepWiki 作为外挂 MCP | "白嫖别人的 AI 处理算力"原则 |
| 零 key 默认 | API key 配置 = 开源传播阻断器 |
| **Adapter vs Skill 边界**(§3.9) | 用户洞察:"web_search 能直接搜到完整内容的不需要列入 adapter" |
| **§11 工程纪律** | DeerFlow 工程实证(domain-knowledge.md §2.10 + §7)+ gstack/superpowers/gbrain "thin harness fat skills" 共同体三方综合 |

---

## 11. 工程纪律(必要的底层选择)

> 完整背景见 [`domain-knowledge.md` §2.10 + §7](./domain-knowledge.md)。本节是 architecture 层面的**约束清单**——哪些必须做、怎么做、为什么。

### 11.1 防幻觉(anti-hallucination)

我们不是 agent loop,但研究质量同样需要 grounding。**5 个强制约束**:

| 约束 | 实现 |
|---|---|
| **Adapter 输出强 schema** | 每个 source adapter 必返 `SearchCandidate`(Pydantic),字段不齐 fail loud。CI 跑契约测试 |
| **Citation 可验证** | URL 必须是真实可达的(可选 HEAD check),arxiv ID 反向 lookup,repo 存在性验证 |
| **Source 永不抛 exception** | adapter 失败必返 `ErrorResult{error_class, message, hint}`,host LLM 看到 explicit error 不脑补 |
| **Raw 全文落 disk + retrieved_at** | host 想验证就 read,**raw 不丢,出问题可追溯**。stdout JSON 只放索引,详细内容在文件 |
| **跨 source 字段标 source** | `{value: "...", source: "github_api", confidence: 1.0}`——host 看到混合数据知道每值来自哪 |

### 11.2 防偷懒(anti-laziness)

研究质量分水岭。**关键洞察**:DeerFlow 用 LoopDetection 防过度行动,但**防"做得不够"主要靠 skill prompt 的 imperative 语气**。我们综合 gstack/superpowers 的命令式 skill 风格 + 代码层的硬约束:

#### 代码层强制(CLI binary 内)

| 约束 | 实现 |
|---|---|
| **强制最小源数** | gate 决定的 Tier 集是**最小集**,不允许只查一个 source。host 看不到选择权 |
| **Top-K 默认慷慨** | 每个 source 默认 k=10,host 显式想要少时才传 `--limit` |
| **Source quality 标记** | 0 结果 + 没 reason → `shallow=true`,fusion 输出带警告 |
| **Recency 强制** | 时效性 query(skill 标记)强制 `published_at >= now - 30d` |
| **Per-source timeout** | 默认 3s,partial failure tolerated,**永不让一条死链阻塞** |

#### Skill 层强制(markdown imperative 语气)

skills/ 文件**必须用命令式语气**(参考 superpowers / gstack 的写法):

```markdown
# find-repo SKILL.md  示例

When user asks to find an OSS repo, you **MUST**:

1. Call `search-harness find-repo <query>` (don't skip — GitHub Search API is the
   only source with proper stars/license ranking).
2. **MUST inspect at least top-3 candidates** before recommending one. Single-result
   recommendations are unacceptable for repo selection.
3. **MUST cross-reference community sentiment** via:
   - web_search "site:news.ycombinator.com <query>"
   - web_search "site:reddit.com/r/programming <query>"
   For Chinese ecosystem add: web_search "site:juejin.cn <query>"
4. **MUST call DeepWiki MCP `ask_question`** on any candidate before final recommendation.
5. **DO NOT** rely on the candidate's README alone — fetch + verify recent commit activity.
```

→ **关键约定**:
- `MUST` / `DO NOT` 大写:这是 spec,不是建议
- 编号步骤 + verifiable 退出条件
- 每个 skill 文件末尾必有 "**Anti-patterns**" 段,列出"绝对不能做的"
- 参考 gstack 的 `slop-scan`、superpowers 的 "If You Are an AI Agent" 节作为模板

### 11.3 灵活切换(flexible switching)

| 切换维度 | 实现 |
|---|---|
| **Source backend**(Tier 3:Tavily/Exa/Brave) | 抽象 `WebSearchAdapter` 接口 + config 选择,1 行 yaml 切 |
| **Doc grounding source**(Context7/Ref/docs-mcp) | Co-install MCP,host 自己的 MCP 配置切换,我们不管 |
| **Skill 内容** | 纯 markdown,fork 一份改 prompt 不动代码 |
| **Tier 启用 / 禁用** | config.yaml 里 `tiers: [1, 2, 4]` 这样配,跳过某 Tier 不调 |
| **额外 source 加入** | 1 个 adapter 文件 + 1 行 config + 可选 1 个 skill 段。**不动核心** |
| **Locale 切换** | skill cross-reference 段按 locale 分支(中文场景加 site:juejin.cn)|

**底层支撑**(参考 DeerFlow):
- **Reflection-based class loading**:config 写 `use: package.module:Class`,代码 `resolve_class()` 动态加载
- **Provider pattern**:每类 backend 有抽象接口 + 多实现
- **Lazy init**:adapter 不调就不 import

### 11.4 轻量化的工程约束(必须做到)

| 约束 | 实施 |
|---|---|
| **Core/CLI 边界** | CI grep `core/*.py` 不出现 `import cli` / `import skills`(对应 DeerFlow 的 `test_harness_boundary.py`) |
| **TDD mandatory** | 每个 adapter 必有 mock-based 单测 + VCR/cassette 录真实 API。`make test` 在 commit 前必跑 |
| **Adapter 契约测试** | 所有 adapter 输出过 `SearchCandidate` Pydantic 校验(对应 DeerFlow 的 GatewayConformance) |
| **Trace/query_id** | CLI 入口生成,所有 log 带,output JSON 带 |
| **Lazy import 强制** | adapter 不调就不 import 重依赖,`search-harness --help` < 200ms |
| **显式 cache_key 派生** | 每个 adapter 实现 `cache_key(query, filters)` 策略函数,**不允许 dict-hash 全部参数** |
| **Single binary** | bun compile / pyinstaller 出单文件,目标 < 10MB |
| **Skill hash 进 output meta** | output 带 `skill_hash` 字段,host log 显示用的是哪版 skill,**防止 skill 漂移** |

### 11.5 Research 场景特有的工程问题(DeerFlow 没解决,我们要正面打)

| 问题 | 我们怎么打 |
|---|---|
| **跨 source 矛盾**(GitHub 说 X 是 BSD,libraries.io 说 MIT) | fusion 阶段做 conflict detection,在 result 里标 `conflict: [...]` 字段,host 自己决定 |
| **同一事物多个 candidate** | URL/canonical-id dedup + 标 `also_at: [...]` |
| **Recency vs 完整性 trade-off** | per-source `mode: "fresh"|"deep"` 参数,gate 根据 query 类决定 |
| **"没有结果"的 grounding** | 某 source 真的 0 结果时,fusion 输出 `meta.empty_sources: [arxiv, semantic_scholar]`,**host 知道是真的没,不是漏查** |
| **可复现性** | 同 query 同时戳 cache hit 必返同结果。`query_id + retrieved_at` 写进 output |
| **Skill 防漂移** | skill 文件 hash 写进 output meta(见 §11.4) |

### 11.6 一句话总结

> DeerFlow 提供工程骨架(provider 抽象、reflection、boundary firewall、lazy init、Pydantic 全栈、trace_id),gstack/superpowers 提供 skill 写作纪律(imperative 语气、verifiable 步骤、reject slop 文化),gbrain 提供 "fat skills" prompt 强度。**三者综合,而非单一参考**。
