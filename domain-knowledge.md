# Domain Knowledge — Search Agent / Harness 领域

> **用途**:沉淀对话中已经积累的领域知识,作为后续决策、回顾、向他人介绍项目时的**参考手册**。
>
> **不是**:本项目的架构文档(看 `architecture.md`)、源清单(看 `sources.md`)、筛选后的 adapter 分类(看 `adapter-vs-skill.md`)。
>
> **结构**:产品 inventory → 概念与机制(含 DeerFlow 实证)→ 文献引用 → 商业/合规 → 我们的判断与翻车 → DeerFlow 工程框架交叉分析 → 元规则。
>
> **结构**:产品 inventory → 概念与机制 → 文献引用 → 商业/合规 → 我们的判断与翻车 → URL 索引。

---

## 1. 产品 / 项目 inventory

按"承担什么角色"分类。**不预判取舍,只描述客观事实**。

### 1.1 完整 Agent search / Research orchestrator(同类竞品)

这一类是**用户产品 / standalone agent**,不是给我们调的依赖。架构高度收敛于 **Planner → 并行 Executor → Compressor → Writer**。

| 项目 | 一句话定位 | 关键设计 | 链接 |
|---|---|---|---|
| **GPT Researcher**(assafelovic) | OSS deep research,**唯一完整封装成 MCP server** 的 | Planner-Executor-Publisher 三段;**`gptr-mcp` 暴露 `deep_research`/`quick_search`/`write_report`/`get_research_sources` 4 工具**——值得抄接口形态 | [GitHub](https://github.com/assafelovic/gpt-researcher), [MCP](https://github.com/assafelovic/gptr-mcp) |
| **Stanford STORM / Co-STORM** | 学术 deep research,sentence-level grounding | Perspective-Guided Question Asking + Simulated Conversation;严格句级引用 | [GitHub](https://github.com/stanford-oval/storm) |
| **MindSearch**(InternLM) | 经典 planner+并发 searcher 范式 | WebPlanner 拆 DAG + 并行 WebSearcher;3 分钟读 300+ 页 | [GitHub](https://github.com/InternLM/MindSearch), [paper](https://arxiv.org/abs/2407.20183) |
| **HuggingFace Open Deep Research** | smolagents 极简实现(~1000 行) | ReAct + code-as-action;参考价值在 minimalist agent loop | [smolagents](https://github.com/huggingface/smolagents/tree/main/examples/open_deep_research) |
| **LangChain Open Deep Research** | LangChain 旗舰示例,代码组织最干净 | Supervisor + parallel sub-agents,sub-agent context 隔离 + 压缩回传 | [GitHub](https://github.com/langchain-ai/open_deep_research) |
| **LangChain DeepAgents** | 通用长任务 agent 模板 | `write_todos` + 虚拟 filesystem + subagent + memory | [GitHub](https://github.com/langchain-ai/deepagents) |
| **Bytedance DeerFlow** | 通用 super agent harness(2.0)/ deep research(1.x) | 16+ middleware 链 / subagent / sandbox / skills(2.0);静态 graph(1.x) | [GitHub](https://github.com/bytedance/deer-flow) |
| **Vane**(ItzCrazyKns) | Perplexica 升级版,自托管 Perplexity 克隆 | SearxNG 后端 + Next.js 前端 + 任意 LLM provider;**search 部分是 wrapper,无垂直 ranking** | [GitHub](https://github.com/ItzCrazyKns/Vane) |
| **Perplexity** | 商业 answer engine | 自建爬虫 + 索引;Perplexity Computer 启用并发 sub-agent | [perplexity.ai](https://www.perplexity.ai) |

### 1.2 RL 训练的 search agent(学术热点,不直接借鉴)

| 项目 | 一句话 | 链接 |
|---|---|---|
| Search-R1 | veRL 上 RL 训 multi-turn search,7 datasets +41% over RAG (COLM 2025) | [GitHub](https://github.com/PeterGriffinJin/Search-R1) / [paper](https://arxiv.org/abs/2503.09516) |
| Search-O1 | 给 reasoning model 加 agentic RAG + Reason-in-Documents (EMNLP 2025) | [GitHub](https://github.com/RUC-NLPIR/Search-o1) |
| R1-Searcher | 两阶段 outcome-based RL,纯 RL 无 SFT | [GitHub](https://github.com/RUCAIBox/R1-Searcher) |

→ 都是**训练范式**,需要 RL pipeline,对"无训练、纯 prompt orchestration"路线参考度低。

### 1.3 Doc grounding(防 API 幻觉)

| 工具 | 模式 | 关键差异 |
|---|---|---|
| **Context7**(Upstash) | MCP server,**预索引**主流库 | 0 配置免费,主流库 80% 命中,小众/私有库不行 |
| **Ref.tools** | MCP server,**agentic search+read** | session-aware,**95% 省 token**,**私有 repo 索引免费**,带 web fallback |
| **docs-mcp-server**(arabold) | OSS 自托管 MCP | 完全可控覆盖范围,自己运维 |
| **Jina Reader**(`r.jina.ai`) | URL → markdown reader | 0 配置免费,无索引,按需抓 |
| **Firecrawl** | 商业 scraping + search | JS 渲染 + 结构化抽取,JS-heavy 站点必备 |

### 1.4 Browser automation

| 工具 | 形态 | 内部 LLM | 维护方 |
|---|---|---|---|
| **`@playwright/mcp`**(Microsoft) | MCP server,zero config | ❌ 纯工具 | Microsoft,31.4k stars |
| **`@playwright/cli`**(Microsoft 2026) | CLI(给 host 写 bash 调用),最 token 效率 | ❌ | Microsoft |
| **browser-use** library | Python lib,内置 agent loop | ✅ 需配 LLM key | 社区,~3k stars |
| **Vercel agent-browser** | Rust CLI + Node daemon hybrid | 部分 | Vercel |
| **Browser DevTools MCP** | MCP,声称比 Playwright MCP 省 78% token | ❌ | 社区 |
| **Claude in Chrome** | Chrome extension | host LLM | Anthropic |

**关键事实**:browser-use ≠ Playwright 的替代品,**它建在 Playwright 上**。它们之间的真正差异是"内置 LLM 决策" vs "host LLM 决策"。

### 1.5 商业 AI search API

| API | 索引来源 | 输出特点 | 编程友好度 |
|---|---|---|---|
| **Tavily** | 自家爬虫 | 清洗后内容,RAG-ready,有 answer 端点 | 一般 |
| **Exa** | 自家 neural index | 语义匹配,有 People/Company/**Code** vertical | **强**(Exa Code 端点) |
| **Brave Search API** | 自家完整索引(2025 已脱离 Bing) | 标准 SERP + LLM Context endpoint | 一般 |
| **Serper.dev** | Google SERP scraping | Google 真排序 JSON | 借 Google ranking,适合长尾 |
| **SerpAPI** | 多引擎 SERP scraping | 同上 + 多引擎 | 同 Serper 但贵 5-10× |
| **Linkup** | 爬虫 + 出版商授权 | `/fast` + `/deep`,EU 合规 | 复杂多跳强项 |
| **Perplexity Sonar** | 黑盒(疑似套 Brave/Serper) | synthesized answer | **不适合 harness**(替宿主 LLM 做了 synthesis) |
| **Kagi Search API** | 多源融合 | 标准 + FastGPT + Summarizer | **closed beta,不可商业依赖** |
| **You.com API** | 混合 | Web Search / Contents / Research 三档 | 无差异化 hook |
| **Google Programmable Search** | Google 索引(限站点) | 标准 SERP JSON | **2025 停止新签约,2027/01 全面下线** |
| **Bing Web Search API** | — | — | **2025-08-11 已退役** |

**第三方 benchmark**(AIMultiple 8-API 测试):
- Agent Score 排名:Brave 14.89 > Firecrawl 14.58 > Exa 14.39 > Tavily 13.67 > Perplexity 12.96 > SerpAPI 12.28
- Tavily 自报 SimpleQA 93.3% / Exa Research 94.9% / Linkup 91% — **几乎所有"vs Google"benchmark 都来自厂商自家 blog**,需打折看
- WebWalker 多跳:**Exa 81% > Tavily 71%**
- p95 延迟:**Exa 1.4-1.7s vs Tavily 3.8-4.5s**

### 1.6 AI-Augmented 二级源(白嫖别人 LLM 算力)

| 源 | 价值 | 关键 |
|---|---|---|
| **DeepWiki**(Cognition / Devin 团队) | **50k+ 主流 repo 已预生成 AI wiki + Q&A** | **官方 MCP,完全免费无 auth**:`ask_question`/`read_wiki_contents`/`read_wiki_structure` |
| **deepwiki-open**(AsyncFuncAI) | OSS 自托管 | 可指向**私有 repo / 内网 GitLab** |
| **OpenDeepWiki**(AIDotNet) | C#/TS 实现 | 模块化,适合企业内集成 |
| **Phind** | coding 垂直 AI 搜索 | Perplexity-for-devs |
| **gitingest** | repo→LLM-ready 文本 | URL 替换:`gitingest.com/<owner>/<repo>` |
| **Repomix** | CLI:打包 repo 成单文件 | 支持过滤/格式 |
| **GitMCP** | repo → MCP server | 暴露给 agent |
| **uithub** | URL trick:`uithub.com/x/y` 取全 repo Markdown | 0 配置 |

### 1.7 编程领域专用工具(supporting cast)

| 工具 | 角色 |
|---|---|
| **Aider repo map** | 本地 codebase grounding 金标准:**tree-sitter + symbol 图 + personalized PageRank** |
| **Sourcegraph Cody** | 跨仓库符号搜索,zoekt + 远端代码主机 |
| **Continue.dev** | 自定义 Context Providers,有 repo-map provider |
| **SWE-Search / Moatless** | MCTS + 双 agent (Value/Discriminator),SWE-bench +23% |
| **Serena MCP** | LSP-based symbolic code search,MCP 形态 |
| **Claude Context**(Zilliz) | 向量代码检索 MCP |
| **GitHub MCP server**(官方) | repo/issue/PR 操作 |

### 1.8 "Thin Harness, Fat Skills" 共同体(Garry Tan + Anthropic 系)

三个项目共享同一种哲学,且彼此设计成可组合:

| 项目 | 作者 | 一句话 | 形态 |
|---|---|---|---|
| **[gstack](https://github.com/garrytan/gstack)** | [Garry Tan](https://github.com/garrytan)(YC CEO) | "把 Claude Code 变成虚拟工程团队"——23 个专家角色 + 8 个工具(CEO/Eng/Designer/QA/Security/Release) | CLI + skills,装在 ~/.claude/skills/ |
| **[gbrain](https://github.com/garrytan/gbrain)** | Garry Tan | "Your AI agent is smart but forgetful. GBrain gives it a brain"——29 skills 的个人知识 brain + 自连知识图,PGLite/Postgres+pgvector | CLI + MCP server + skills |
| **[superpowers](https://github.com/obra/superpowers)** | [obra](https://github.com/obra) (Jesse Vincent, Anthropic) | "完整的 software development methodology for your coding agents",TDD + subagent-driven 开发 | Anthropic 官方 plugin marketplace |

**关联(三层)**:

1. **共同哲学**:三者都明确宣称 "**Thin harness, fat skills**" 原则——把智能放在 skills(markdown 工作流),runtime 极薄。gbrain 有专门一篇 [`docs/ethos/THIN_HARNESS_FAT_SKILLS.md`](https://github.com/garrytan/gbrain) 论述
2. **gstack ↔ gbrain 显式联动**:`gstack/setup-gbrain` 命令、`USING_GBRAIN_WITH_GSTACK.md` 文档、gstack 把 gbrain 当 host 注册——**gstack 管"如何 ship",gbrain 管"如何 remember"**
3. **多 host 兼容设计**:三者都通过 CLI + skills 形态自然兼容 Claude Code、Codex CLI、Cursor、OpenCode、OpenClaw 等多个 agent runtime

**对我们 harness 的直接意义**:
- 三者验证了我们最近的 "**CLI + skills**" 架构选择(§5.1 #4)
- gbrain 可以是我们 harness 的 **memory 层**——存用户搜索 candidates / 技术栈偏好 / 过往项目选择
- gstack 的 `/setup-X` 一键安装模式是 README 的样板
- superpowers 的 **94% PR rejection rate** + "**If You Are an AI Agent**" 反 slop 守则值得抄到我们 CONTRIBUTING

**关于 OpenClaw**:gstack/gbrain 重度提到 [OpenClaw](https://github.com/openclaw/openclaw)(247k stars,Peter Steinberger 主导)。这是开源 agent runtime 生态的核心节点之一。**我们 CLI + skills 形态自然兼容,不需要专门适配**。

---

## 2. 概念与机制

### 2.1 LLM Web Search 真实拓扑

**LLM 自己不会搜索**。任何号称"有 web search"的模型,本质都是三件东西的组合:

```
┌────────────────────────────────────────────┐
│  ① 后端搜索引擎(真正干活的)              │
│  Gemini → Google Search 自家索引(独占)   │
│  ChatGPT → Bing → SearchGPT 自建爬虫       │
│  Claude → Brave Search API                  │
│  Perplexity → 完全自建爬虫和索引            │
│  Qwen/MiniMax/DeepSeek → 多数租 Bing/百度   │
├────────────────────────────────────────────┤
│  ② 模型侧的三件后训练能力                  │
│  - 何时搜(知识截止 + 置信度判断)         │
│  - 怎么搜(query rewriting / decomposition)│
│  - 怎么读(snippet → grounded answer)     │
├────────────────────────────────────────────┤
│  ③ 工程胶水(决定体验上限)                │
│  snippet vs 全文抓取 / rerank / browse loop │
└────────────────────────────────────────────┘
```

**全球真正"自己爬网建索引"的引擎不超过 5 家**:Google、Bing、Brave、Perplexity、Yandex/Baidu。其他都是套壳或复用。**索引层是真正护城河**。

### 2.2 浏览器 vs 搜索引擎(易混淆)

```
┌─ AI 答题引擎  →  Perplexity / ChatGPT / Claude
├─ 浏览器(UI)→  Chrome / Firefox / Brave / Safari / Edge
└─ 搜索引擎(索引)→ Google / Bing / Brave Search / DuckDuckGo
```

- **浏览器 ≠ 搜索引擎**。Chrome 自己不搜索,只是把 query 发给 Google
- **Chromium 提供**:Blink 渲染引擎、V8 JS 引擎、Network stack、Web API 实现、沙箱、扩展 API、DevTools
- Chrome / Edge / Brave / Opera / Arc / Vivaldi 都是 Chromium 上加产品壳;Firefox(Gecko)/ Safari(WebKit)是另两个独立内核
- **DuckDuckGo 严格说是搜索引擎**(虽然主要复用 Bing 索引);**Brave 既是浏览器也有自家 Brave Search**

### 2.3 网站发布即被搜到的原理

**"被搜到"不是浏览器干的,是搜索引擎爬虫干的**。爬虫发现新站靠 5 条路径:
1. **Backlinks**(主流)— 已索引网站链向你
2. **Sitemap 主动提交**(最快,几小时到几天)
3. **Certificate Transparency 日志**(申请 HTTPS 证书会触发)
4. **WHOIS 监控**(部分爬虫)
5. **随机扫描**(极少)

只要你的站满足"公开 DNS + 可 HTTP 抓取 + 不在 robots 里禁",**世界上任何人都可以爬你**——爬虫不需要你授权。这就是"开放 Web"的基础。

### 2.4 多模态信噪比的 4 种细分

把"multimodal"拆开,4 种问题完全不同:

| 类型 | 例子 | 难度 | SNR |
|---|---|---|---|
| 多模态作为内容类型 | "找一个讲 React Server Components 的视频" | 中 | **高**(只检索) |
| 多模态作为消费内容 | "把这 30 分钟视频里讲 hooks 的部分总结" | 高 | **低**(全 transcribe) |
| 多模态作为输入理解 | "我截屏了报错,告诉我哪里不对" | 中 | **高**(单图 vision) |
| 多模态作为 ranking 信号 | "看页面截图判断结果相不相关" | 高 | **极低**(token 巨大) |

**关键 insight**:多模态的"高 SNR 用法"是**消费平台预处理过的文本元数据**(YouTube transcript / 播客文字版 / slides PDF),而不是自己跑 ASR/OCR/Vision。

**Long-tail 框架**:80% query 在 text 层够 / 15% 在准多模态(transcript)够 / 5% 真需要 raw video/image。**架构原则**:为 80% 优化默认路径,为 15% 加廉价层,为 5% 留昂贵但可达的 escape hatch。**不要把 5% 的成本均摊到所有 query**。

### 2.5 Skills vs MCP 的 Token 经济(2026 已收敛的共识)

[Anthropic 工程博客《Code execution with MCP》](https://www.anthropic.com/engineering/code-execution-with-mcp) 的核心发现:

- **150K tokens → 2K tokens**(同任务,代码执行 vs MCP 调用),**节省 98.7%**
- "Tool definitions 在 Anthropic 内部曾消耗 134K tokens"——光 schema 就把 context 烧了大半
- 简单任务上 **MCP 用 32× 于 Skills 的 token**

[Playwright 官方 benchmark](https://testcollab.com/blog/playwright-cli):
- **Playwright MCP 114K tokens vs Playwright CLI 27K tokens**,~4× 减少
- 关键机制:**完整 DOM 或图片二进制不进入 LLM context,除非 agent 显式 read 那些文件**

| 维度 | MCP | Skills + Bash CLI |
|---|---|---|
| Tool 描述驻留 context | 全部 schema 永久占,N×几千 token | **Progressive,按需载入** |
| 输出处理 | 全部塞回 LLM | **落文件,grep/jq/head 按需读** |
| 多步组合 | 每步 round-trip 给 host LLM | **管道直接 pipe**,中间不进 LLM |
| 训练分布 | MCP 是 2024 才有 | **bash/curl/jq/grep 是 30 年训练数据基底** |
| 状态管理 | server 自己存 state | **文件系统 = 0 cost state** |

**MCP 仍有价值的少数场景**:
- 需要 host **直接看到结构化 schema** 才能调对的工具
- **持久状态/会话**(如长 session 的 browser context)
- **跨 agent 标准化协议**(给别人用)

### 2.6 多源路由:Flooding vs Routing(20 年 IR 文献已收敛)

**经典 IR 文献**:
- **Si & Callan ReDDE (SIGIR 2003)**:routing 在源 >10 且大小不均时显著优于 flooding
- **Thomas & Shokouhi SUSHI (SIGIR 2009)**:同上
- **Shokouhi & Si 2011 综述**:**候选源 ≤ 6 且都是高质量精选源时,flooding 与 routing 几乎打平,且实现简单得多**(routing 的分类误差吃掉节省)

**LLM 时代实证**:
- **Toolformer / Gorilla**:工具空间 ≥1600 时,**学过 routing 远胜 flooding**
- **Self-RAG**:"按需 retrieve" 比 "always retrieve" 好——但前提是判得准
- **MindSearch**:planner 拆 DAG,**每个子问题内部并发 fan-out**(3 分钟读 300 页)
- **Anthropic Contextual Retrieval**:BM25+embedding+rerank **hybrid 把检索失败率砍 67%**

**业界共识(2024-2026)**:**没有任何严肃产品做"纯智能 routing 选一个源"或"无脑 flood 所有源"**。共识是 **三段式混合**:
```
Decompose query → Parallel fan-out within source-class
                → RRF fusion + dedup → LLM rerank top 20-30
```

**对小 N(4-6 源)场景**:**flood + 廉价启发式 gate + RRF + LLM rerank** 是当前最优。学过的 router 留到 >15 源再说。

### 2.7 AI 搜索为什么有时不如自己 Chrome

**多层有损变换**,每层都丢信息:

1. **Query 层**:LLM 改写后精度被稀释("OpenAI API rate limiting" 替代 "ChatGPT API 429 retry-after")
2. **检索层**:后端弱(Brave < Bing < Google)+ Top-K 太小(3-5 vs 10+)+ 抓不到 JS/login + 走通用而非垂直站
3. **综合层**:5 个结果合成成一段,失去源归属、冲突信号、原文结构、recency,可能 paraphrase 出错
4. **UX 层**:人扫 10 条标题 5 秒,自带"哪些站靠谱"的品味,LLM 没有
5. **框架层**:AI 给"答案",但很多任务实际要"地图"

→ **设计 harness 的反推原则**:pass-through 模式 / top-K 给大 / 结构保留 / 不做 synthesis / 垂直直连 / recency 显式化

### 2.8 垂直站的真正价值 = 它的 API,不是它的域名

每个垂直站有**两套东西**:
- 它的 **网页**(被 Google 索引的部分)
- 它**自己**的搜索/API(带原生 ranking 信号:vote/stars/citation/published_at)

**Google 索引只抓到第一列**。即使你 `site:stackoverflow.com xxx`,你也拿不到 SO 自己的 vote/accepted ranking。**真正"原生搜索"是用站方自己的 API**。

→ harness 价值 = "**绕开通用 web 索引,直接走垂直站的原生 API,拿到原生 ranking 信号和结构化数据**"。Chrome 能做这件事是因为人会手动切站;AI 默认做不到,harness 把这件事自动化。

### 2.9 Adapter vs Skill 边界测试(派生自 2.8)

**原则**:**adapter 的存在意义是"web_search 替代不了它的 ranking/字段",不是"它内容值得"**。

否则就只是 wrapper(给 web_search 套个 Pydantic schema 而已,无新增价值)。

**4 问决策树**:

```
Q1. Web search 能找到这个站的内容吗?
    ├─ 不能(login wall / API only)→ adapter 或 drop
    └─ 能 →  Q2

Q2. Web search 能拿到它的核心 ranking 信号吗?
    ├─ 不能(stars/votes/citation/downloads 等)→ adapter
    └─ 能 → Q3

Q3. 它已经有免费 MCP / API 暴露给 agent 了吗?
    ├─ 是 → co-install,不算 adapter
    └─ 否 → Q4

Q4. agent 知道这个站存在吗?
    ├─ 不知道 → 加进 skill 的 cross-reference 段(教 agent 用 web_search + `site:`)
    └─ 知道 → 不需要任何动作
```

**实际应用结果**:把 sources.md 数百个候选源筛到 12 个核心 adapter + 5-7 个可选 + 数十个 skill-only。详见 [`adapter-vs-skill.md`](./adapter-vs-skill.md)。

**Skill 双重职责**(派生洞察):
- 教 host 怎么调 adapter(路线图)
- **教 host 还应该 web_search 哪些 site**(awareness 教学)——这是 skill 真正的杠杆,把数十个 skill-only 站集中暴露

### 2.10 防幻觉 / 防偷懒 / 灵活切换 三套机制(DeerFlow 实证)

**对应 architecture.md §11 工程纪律**。这一节梳理"业界主流 agent harness 在三个研究关键问题上是怎么做的",作为我们设计自己机制时的参考库。

#### 2.10.1 防幻觉(anti-hallucination)

DeerFlow 的实际做法(以中间件为主):

| 机制 | 文件 | 原理 |
|---|---|---|
| **ClarificationMiddleware** | `agents/middlewares/clarification_middleware.py` | agent 调 `ask_clarification` 工具→直接 `Command(goto=END)` 中断,强制用户介入。**不让模型在信息不全时硬猜** |
| **ToolErrorHandlingMiddleware** | `agents/middlewares/tool_error_handling_middleware.py` | tool 抛 exception → 转成 ToolMessage 注入 context。**模型不会"以为成功了"** |
| **DanglingToolCallMiddleware** | `agents/middlewares/dangling_tool_call_middleware.py` | AI 发了 tool_call 但被打断没结果?注入占位 ToolMessage,**强制 thread 一致** |
| **MemoryMiddleware 的 prompt injection** | `agents/memory/` | 把已知事实灌进 system prompt 的 `<memory>` 标签,**让模型看到 ground truth 而非编造** |
| **Pydantic schema 全栈** | `app/gateway/routers/*.py` 的 response_model | 任何 API 返回必走 schema 校验,**字段不对就 fail loud** |
| **Skill 强制 citation** | 各 `skills/*/SKILL.md` | skill markdown 里写死"必须引用源,格式 `[file:line]`",**模型遵守 skill 就是遵守 citation 纪律** |

#### 2.10.2 防偷懒(anti-laziness)

| 机制 | 文件 | 原理 |
|---|---|---|
| **LoopDetectionMiddleware**(双层) | `agents/middlewares/loop_detection_middleware.py` | hash-based(同 tool calls 重复) + frequency-based(同 tool 类型 30 次警告/50 次硬停)。**catch 模型卡死或刷量** |
| **TodoListMiddleware** | `agents/middlewares/todo_middleware.py` | plan_mode 下,模型必须维护 todo list,**不能"忘记"中间步骤** |
| **SubagentLimitMiddleware** | `agents/middlewares/subagent_limit_middleware.py` | 截断超额并发 task 调用(MAX_CONCURRENT=3),**防止一次喷 20 个 subagent 然后躺平** |
| **recursion_limit + max_turns** | `subagents/executor.py:303` | 硬上限,模型刷再多步也被掐 |
| **SummarizationMiddleware** | `agents/middlewares/summarization_middleware.py` | context 接近上限时自动总结,**模型不能用"context 满了"当借口偷懒** |

**关键洞察**:DeerFlow 的 anti-laziness 主要是"防过度行动"(loop detection),**对"做得不够"的防御主要靠 skill 文件里的 imperative prompt**。这点 gstack/superpowers 做得更狠——命令式 skill 语气、verifiable 步骤。**我们综合吸收**(见 architecture.md §11.2)。

#### 2.10.3 灵活切换(flexible switching)

| 机制 | 文件 | 原理 |
|---|---|---|
| **Reflection-based class resolution** | `reflection/` (resolve_variable, resolve_class) | config.yaml 写 `use: langchain_openai:ChatOpenAI`,代码 `resolve_class(path, BaseModel)` 动态加载。**换 backend 改 1 行 yaml** |
| **Provider pattern** | `sandbox/sandbox_provider.py`、`models/factory.py` | 抽象接口 + 多实现(LocalSandbox/AioSandbox / 多 model)。**新 provider 只实现接口** |
| **`get_available_tools()` 动态组合** | `tools/__init__.py` | 启动时根据 config + MCP + builtins + subagent 配置组合工具集。**enable/disable 单 tool 不重启** |
| **MCP 外挂** | `mcp/` + `extensions_config.json` | 外部工具一律 MCP 接,**新工具不改主代码** |
| **Skills as markdown** | `skills/{public,custom}/` | 加 skill = 加 markdown 文件 + 一行 enabled state,**0 代码扩展** |
| **Config 自动 reload** | `config/app_config.py` 的 mtime 检查 | 改 yaml 不重启 server,**热切换** |
| **Lazy init** | 所有 middleware 默认 `lazy_init=True` | 不用就不构造,**启动快** |

#### 2.10.4 轻量化的工程约束

| 约束 | 实施方式 |
|---|---|
| Harness/App 边界 | CI grep 测试:`tests/test_harness_boundary.py` 检查 `packages/harness/` 不出现 `import app.*` |
| TDD | CLAUDE.md 第一条强制每个 feature 必有测试,~110 个测试文件实测 |
| Trace ID 贯穿 | 每个 SubagentExecutor 一个 8-hex trace_id,所有 log 带它 |
| 配置版本化 | `config_version: 8` 字段 + `make config-upgrade` 自动迁移 |
| 类型严格 | 全栈 Pydantic + Python 3.12 type hints + ruff 240 字符 |

---

## 3. 关键论文 / 文献(可直接 cite)

### 3.1 经典 IR(federated search / source selection)

| 文献 | 链接 |
|---|---|
| ReDDE — Si & Callan SIGIR 2003 | http://www.cs.cmu.edu/~lsi/Sigir_2003.pdf |
| SUSHI — Thomas & Shokouhi 2009 | https://www.researchgate.net/publication/221299793_SUSHI |
| Federated Search 综述 — Shokouhi & Si 2011 | https://www.microsoft.com/en-us/research/wp-content/uploads/2011/01/now.pdf |
| Resource Selection on the Web — Dai et al. 2016 | https://arxiv.org/abs/1609.04556 |

### 3.2 LLM 时代 search agent

| 论文 | 链接 |
|---|---|
| WebGPT (2021) | https://arxiv.org/abs/2112.09332 |
| ReAct (2022) | https://arxiv.org/abs/2210.03629 |
| Toolformer (2023) | https://arxiv.org/abs/2302.04761 |
| Gorilla (2023) | https://arxiv.org/abs/2305.15334 |
| ToolLLM (2023) | https://arxiv.org/abs/2307.16789 |
| Self-RAG (2023) | https://arxiv.org/abs/2310.11511 |
| FreshLLMs (2023) | https://arxiv.org/abs/2310.03214 |
| Self-Discover (2024) | https://arxiv.org/abs/2402.03620 |
| MindSearch (2024) | https://arxiv.org/abs/2407.20183 |
| Search-R1 (2025) | https://arxiv.org/abs/2503.09516 |
| SWE-Search / Moatless (2024) | https://arxiv.org/abs/2410.20285 |

### 3.3 工程实践博客 / 经验贴

| 来源 | 链接 |
|---|---|
| Anthropic — Code execution with MCP | https://www.anthropic.com/engineering/code-execution-with-mcp |
| Anthropic — Contextual Retrieval(-67% failure) | https://www.anthropic.com/news/contextual-retrieval |
| Anthropic — Advanced tool use | https://www.anthropic.com/engineering/advanced-tool-use |
| LlamaIndex — Router Query Engine | https://developers.llamaindex.ai/python/framework/module_guides/querying/router/ |
| Tavily — WebLangChain post-mortem | https://blog.tavily.com/building-and-breaking-weblangchain/ |
| Perplexity Computer 介绍 | https://www.perplexity.ai/hub/blog/introducing-perplexity-computer |
| AIMultiple — 8-API Agentic Search benchmark | https://aimultiple.com/agentic-search |

---

## 4. 商业 / 合规

### 4.1 API key 经济(免费 vs 付费源)

**对 coding 场景,80% 的核心源完全不需要 key**:

| 门槛 | 源 |
|---|---|
| **零 key,直接用** | GitHub(匿名 60/hr)/ arXiv / HN Algolia / Wikipedia / npm/PyPI/crates / Stack Exchange / DuckDuckGo 爬虫库 / Hugging Face Hub(只读)/ DeepWiki MCP |
| **免费 key,30 秒注册** | GitHub PAT(60→5000/hr)/ Brave(2k/月免费)/ Tavily(1k/月免费)/ Reddit OAuth / Semantic Scholar(可选) |
| **付费才好用** | Exa / Serper / Linkup / Firecrawl |

→ "需要配 N 个 key" 的担忧被高估。**默认零 key 完整能跑,GitHub PAT 是唯一推荐升级**。

### 4.2 爬虫合法性(给选型用的判断)

**通常合法的**:
- 爬公开页面、不绕过登录
- 遵守 robots.txt(礼仪非法律)
- 合理 rate limit(不构成 DOS)
- 在 User-Agent 注明身份

**通常不合法**:
- 绕过认证 / 付费墙(违反 CFAA 或类似)
- 违反 ToS 用于商业竞争(灰色)
- 转发盗用版权内容
- 极高频率致服务降级

**各源风险等级**:

| 源 | 风险 | 建议 |
|---|---|---|
| GitHub / SO / arXiv / HN | 低 | 用 API,SO 内容是 CC-BY-SA 可大量使用 |
| Reddit | 中 | API,非要爬就低速 |
| DuckDuckGo | 低-中 | 社区一直爬,不商用大规模 |
| Google / Bing SERP | **高** | **别自己爬,用 Serper/SerpAPI 转嫁** |
| X/Twitter / LinkedIn | **极高** | **别碰** |
| 各官方 docs / engineering blog | 极低 | 随便爬 |

**OSS 项目模板免责声明**:"This tool fetches public web content. Users are responsible for compliance with target sites' terms of service."

### 4.3 开源 vs 闭源 SaaS(对此项目)

**结论:开源压倒性合适**。理由:
1. 一个人扛不住 SaaS 的销售/客服/合规/SLA
2. 潜在用户(coding agent 开发者)是技术人群,**喜欢能看代码的工具**
3. 开源项目可以"半死不活"还有价值;闭源 SaaS 半死就死
4. 现在 dev tool 高 stars 几乎都开源(Aider/Continue/Cline/browser-use/LangChain)
5. 后期可走 **Plausible / Sentry / Supabase 模式**——开源核心 + hosted 服务变现

---

## 5. 我们做出的判断 + 反例

### 5.1 已经验证的判断(可作为后续决策锚点)

1. **不内置 LLM**——harness 是宿主 LLM 的工具,不要重复 LLM 能力
2. **Output 只给 citation,不做 synthesis**——synthesis 在 host(完整 user context)发生,损耗最小
3. **Flood + RRF 而非 routing**(at 4-6 源)——20 年 IR + 2024-2026 业界实证支持
4. **CLI + skills 而非 MCP server**(对我们 harness 自身)——Anthropic / Microsoft 自己的转向
5. **零 key 默认 + 渐进升级**——80% coding 场景免费源够用
6. **Browser 用 Playwright CLI 而非 browser-use**——4× token + Microsoft 维护 + host 主导决策
7. **Browser 不集成进 harness,co-install 推荐**——Microsoft 帮我们维护
8. **DeepWiki 等 AI 二级源直接白嫖**——别自己跑别人已经做过的 LLM 处理
9. **多模态分 5 Tier,buff 思路**——长尾价值不能用 ROI 均摊算
10. **开源,不闭源**——见 §4.3
11. **Adapter vs Skill 严格分边界**——adapter 只在 web_search 替代不了 ranking/字段时才写;只是"agent 不知道有这站"的问题用 skill 的 cross-reference 段解决。**结果**:Tier 1 收敛到 12 个核心 adapter,数十个 site 转 skill-only,见 [adapter-vs-skill.md](./adapter-vs-skill.md)
12. **三源吸收工程纪律**——DeerFlow 提供工程骨架(provider 抽象、reflection、boundary、lazy init、Pydantic 全栈、trace_id),gstack/superpowers 提供 skill 写作纪律(命令式语气、verifiable 步骤、reject slop 文化),gbrain 提供 "fat skills" prompt 强度。**三者综合,而非单一参考**(见 §1.8 + architecture.md §11)
13. **防偷懒主要靠 skill prompt**——DeerFlow 实证:防过度行动靠 LoopDetection 中间件,**防"做得不够"主要靠 skill 文件里的 imperative prompt**。研究质量分水岭在 skill 文本如何写,不是代码逻辑(见 §2.10.2)

### 5.2 我们一开始想错被纠正的(避免重蹈)

| 早期判断 | 纠正 | 触发 |
|---|---|---|
| "1.x 的 graph 比 2.0 简单,从 1.x 起步" | 改:2.0 的 skill 模型更适合多 vertical;但**两者 search 部分都是 wrapper,不能直接抄** | 用户清晰说"是 harness for agents",且实读 DeerFlow 代码后 |
| "用 MCP server 暴露 harness 给 host" | 改:**CLI + skills 更好**(Anthropic 自己证明 98% token 减少) | 用户 push,实证查了 Anthropic / Microsoft 转向 |
| "browser-use vs Playwright 二选一" | 纠正:**browser-use 建在 Playwright 上**,真正选择是 zero-config 偏好 | 实读 browser-use 文档 |
| "推荐 browser-use" | 改:**Playwright MCP 更适合 zero-config + no internal LLM 原则** | 用户偏好梳理后 |
| 之后 "推荐 Playwright MCP" | 再改:**Playwright CLI + skills 更好**(用户实证 push) | 查 Anthropic Code-execution + Playwright 自家 benchmark |
| "做 MVP 就拼 Tavily + Exa + Serper + Brave 几个 web search" | 纠正:**这是 wrapper 思路**,真正价值在 ranking / fit / 深度处理 / 验证 | 用户明确 push 反 wrapper |
| 漏了 AI-augmented 二级源类(DeepWiki 等) | 加 §0 进 sources.md | 用户 catch |

**反思的 meta-pattern**:每次"按业界主流推"的判断都被实证 push 反了。**这个领域是被旧范式的惯性绑架的**——"MCP 才现代"、"agent loop 是趋势"、"自托管 search engine 是 cool"——但实证一查,几乎都在被新模式替代。**保持反 anchoring 警觉**。

### 5.3 留作未决的开放问题

1. **Router 设计**——gate 层精确逻辑、决策粒度(用户明确说"我还需要时间思考")
2. **Long-tail 任务路由**——多模态(Tier 4-5)、browser(Tier 5)的触发条件、降级策略(用户明确说"我还需要时间思考")
3. **Adapter / skill 的精细化平衡**——边界已定(§2.9 决策树),但具体 case 还需 case-by-case(用户明确说"我还需要时间思考")
4. **Skill 设计粒度**——每个 vertical 一个,还是按"任务模式"分?skill 文件的篇幅、imperative tone 力度(用户明确说"我还需要时间思考")
5. Cache TTL 策略(各 source 时效性差异大)
6. Citation schema 在不同 Tier 间如何统一最小公共集
7. Tier 3 web search backend 默认选哪个
8. ~~sources.md 用户筛选保留哪些~~ → **已通过 [adapter-vs-skill.md](./adapter-vs-skill.md) 解决**
9. 可选 adapter(libraries.io / OpenReview / HF Hub / CVE 等)是否进 Phase 1
10. 是否需要 result memory(同 query 第二次触发不重复 fan-out)—— **gbrain 是天然候选**(见 §1.8)
11. 中日社区站的 web_search 命中率到底如何(需要实测验证 skill-only 假设)

---

## 6. URL 索引(快速跳转)

### 我们项目的同类参考
- [GPT Researcher](https://github.com/assafelovic/gpt-researcher) | [GPTR MCP](https://github.com/assafelovic/gptr-mcp)
- [Stanford STORM](https://github.com/stanford-oval/storm)
- [MindSearch](https://github.com/InternLM/MindSearch)
- [LangChain Open Deep Research](https://github.com/langchain-ai/open_deep_research) | [DeepAgents](https://github.com/langchain-ai/deepagents)
- [Bytedance DeerFlow](https://github.com/bytedance/deer-flow)
- [Vane](https://github.com/ItzCrazyKns/Vane) | [Perplexica](https://github.com/ItzCrazyKns/Perplexica)
- [HuggingFace smolagents](https://github.com/huggingface/smolagents)

### Thin Harness Fat Skills 共同体
- [gstack](https://github.com/garrytan/gstack)(Garry Tan / YC CEO)
- [gbrain](https://github.com/garrytan/gbrain)(Garry Tan)
- [superpowers](https://github.com/obra/superpowers)(Jesse Vincent / Anthropic)
- [OpenClaw](https://github.com/openclaw/openclaw)(Peter Steinberger,247k stars 开源 agent runtime)

### Doc grounding
- [Context7](https://context7.com)
- [Ref.tools](https://ref.tools) | [GitHub](https://github.com/ref-tools/ref-tools-mcp)
- [docs-mcp-server](https://github.com/arabold/docs-mcp-server)
- [Jina Reader](https://jina.ai/reader/)
- [Firecrawl](https://firecrawl.dev/)

### Browser automation
- [Microsoft Playwright MCP](https://github.com/microsoft/playwright-mcp)
- [Playwright CLI 介绍](https://testcollab.com/blog/playwright-cli)
- [browser-use](https://github.com/browser-use/browser-use)

### AI 二级源
- [DeepWiki](https://deepwiki.com) | [MCP Server](https://cognition.ai/blog/deepwiki-mcp-server)
- [deepwiki-open](https://github.com/AsyncFuncAI/deepwiki-open)
- [OpenDeepWiki](https://github.com/AIDotNet/OpenDeepWiki)
- [gitingest](https://gitingest.com) | [Repomix](https://github.com/yamadashy/repomix) | [GitMCP](https://github.com/idosal/git-mcp) | [uithub](https://uithub.com)

### 编程领域
- [Aider repo map](https://aider.chat/docs/repomap.html)
- [Sourcegraph Cody](https://sourcegraph.com/docs/cody)
- [Continue.dev](https://docs.continue.dev/customize/deep-dives/custom-providers)
- [Serena MCP](https://github.com/oraios/serena)
- [SWE-Search / Moatless](https://github.com/aorwall/moatless-tree-search)
- [GitHub MCP server](https://github.com/github/github-mcp-server)

### 关键工程博客
- [Anthropic — Code execution with MCP](https://www.anthropic.com/engineering/code-execution-with-mcp)
- [Anthropic — Contextual Retrieval](https://www.anthropic.com/news/contextual-retrieval)
- [LlamaIndex Routers](https://developers.llamaindex.ai/python/framework/module_guides/querying/router/)
- [AIMultiple agentic search benchmark](https://aimultiple.com/agentic-search)

### Search API 商业
- [Tavily](https://tavily.com) | [Exa](https://exa.ai) | [Brave Search API](https://brave.com/search/api/)
- [Serper](https://serper.dev) | [SerpAPI](https://serpapi.com) | [Linkup](https://linkup.so)
- [Perplexity Sonar](https://docs.perplexity.ai/) | [Kagi API](https://help.kagi.com/kagi/api/search.html)

---

## 7. DeerFlow 工程框架交叉分析(详细)

> **§2.10 给的是 DeerFlow 的具体机制**(防幻觉/防偷懒/灵活切换)。本节给"我们应该抄什么、明确不抄什么、缺什么"的判断。**作为 architecture.md §11 的依据材料**。

### 7.1 直接可用的工程模式(应该抄)

#### Harness/App 边界防火墙(最值钱)

DeerFlow 把代码严格分两层:
- `packages/harness/deerflow/` — 可发布核心(纯库,import 前缀 `deerflow.*`)
- `app/` — 应用层(FastAPI gateway / IM channels,import 前缀 `app.*`)

规则:`app` 可 import `deerflow`,但 `deerflow` **绝不能** import `app`。**用 CI 测试强制**(`tests/test_harness_boundary.py`)。

→ 我们的等价:
```
search-harness/
├── core/        ← source adapters / gate / fusion(纯库,无 CLI/skill 知识)
├── cli/         ← argparse / 输出格式化 / 文件 IO(import core)
└── skills/      ← markdown 文件
```
+ CI test 检查 `core/*.py` 不出现 `import cli` / `import skills`。

**为什么值得**:让我们以后可以**给 core 加任意 surface**(Python lib / MCP server / 长驻 daemon)而无需重构。

#### LoopDetectionMiddleware 的"稳定键"思想(不是 loop detection 本身)

DeerFlow `_stable_tool_key()` 函数:**针对不同工具类型,用不同策略派生稳定 key**。

```python
# read_file: 把 line range 按 200 行分桶
#   → "src/foo.py:0-1" 而不是 "src/foo.py:1-37"
# write_file/str_replace: 内容敏感,hash 全 args
# 通用: 取 (path, url, query, command) 这些 salient 字段
```

**对我们 cache/dedup**:
- GitHub: `query + filters` 是 key,**不进 max_results**
- arXiv: `query + date_bucket(week)`,不要 exact timestamp
- web search: `query` only(snippet 在变,但同 query 拿同批 URL 是合理的)

→ **每个 source adapter 必须实现 `cache_key(query, filters) -> str` 策略函数,不能用通用 dict-hash**。

#### 工具错误 → 消息

DeerFlow 的 `ToolErrorHandlingMiddleware` 把 tool 抛出的 Exception 转换成 ToolMessage,让 agent loop 能继续。

**对我们的等价**:任何 source 的 exception **绝不能传播出 fan-out 阶段**。每个 source 的失败 → 一条带 `error` 字段的空 result,fusion 阶段感知到但不阻塞其他 source。

→ "**Adapter 强制契约**":每个 adapter 必须实现 `async def search(query) -> SearchResult | ErrorResult`,**不允许 raise**。CI 加测试验证。

#### Trace ID 贯穿全链路

DeerFlow 给每次调用生成 `trace_id = str(uuid.uuid4())[:8]`,**所有 log 都带这个 ID**。

**对我们**:
- CLI 入口生成 `query_id`(8 位 hex)
- 透传到每个 source adapter 的 logger
- Output JSON 也带 `query_id` 字段

**好处**:用户报 bug 时贴 `query_id`,你立刻能在 log 里筛出整条流水线。

### 7.2 自觉不抄的 over-engineering(明确不做)

| DeerFlow 有的 | 为什么我们不做 |
|---|---|
| **17 个 middleware 链** | 大部分是 agent loop 概念(Title/Memory/Sandbox/Subagent…),CLI 不需要 |
| **SubagentExecutor + 双 thread pool** | 我们不 spawn agent,只在 source 间并发,asyncio.gather 就够 |
| **ThreadState 自定义 reducer** | 无状态 |
| **Memory 系统 + queue debounce** | CLI 之间无 cross-call memory(host LLM 自己管)。**真要长期 memory → 集成 gbrain** |
| **Sandbox 双 provider(Local/Aio)** | 不执行代码 |
| **Channels(Slack/Feishu/Telegram…)** | 不在 search 范畴 |
| **LangGraph Server** | 不要 long-running server |
| **Streaming bridge** | CLI stdout 自带流式 |
| **Config versioning + 自动 reload** | MVP 阶段 yaml 改了就重启 CLI |

**总原则**:DeerFlow 解决的"agent 长跑生命周期"问题,我们大多数不存在。**借鉴的是底层工程纪律,不是上层 agent 框架**。

### 7.3 我们可能缺的(应该补)

#### TDD 文化

DeerFlow 的 CLAUDE.md 第一条:"**Every new feature or bug fix MUST be accompanied by unit tests. No exceptions.**"——~110 个测试文件实测。

→ 对我们:
- 每个 source adapter 必须有 mock-based 单元测试(模拟 API 响应)
- gate / fusion 必须有 property-based 测试(给定 N 个 source 输出,验证 RRF 性质)
- end-to-end 测试用 VCR/cassette 录制真实 API 响应

#### Adapter 的"契约测试"

DeerFlow `TestGatewayConformance`:**用 Pydantic model 验证每个 client 方法的返回**。Gateway 加新字段而 client 没跟上,CI 立刻 fail。

→ 对我们:每个 source adapter 应该返回统一的 `SearchCandidate` schema(pydantic),CI 跑契约测试确认所有 adapter 都满足。

#### Lazy initialization

DeerFlow 的 middlewares 默认 `lazy_init=True`——只在被实际用到时才构造。

→ 对我们:source adapter 应该**懒加载**——不调 GitHub 就不 import `httpx + GitHub SDK`。**这对零配置体验很关键**(`search-harness --help` 不应该花 2 秒加载所有依赖)。

### 7.4 关键文件参考(DeerFlow)

| 文件 | 作用 |
|---|---|
| `backend/packages/harness/deerflow/agents/lead_agent/agent.py:241` | `_build_middlewares()` — 中间件组装顺序 |
| `backend/packages/harness/deerflow/agents/middlewares/loop_detection_middleware.py` | LoopDetection 完整实现 + stable key 思想 |
| `backend/packages/harness/deerflow/subagents/executor.py:128` | SubagentExecutor — 后台并发任务 + cancellation |
| `backend/packages/harness/deerflow/agents/middlewares/tool_error_handling_middleware.py` | 错误 → message 转换 |
| `backend/tests/test_harness_boundary.py` | CI 边界防火墙测试 |
| `backend/CLAUDE.md` | 工程规范全集(TDD + 边界规则) |

### 7.5 一句话总结

> **DeerFlow 解决的是"agent 长跑生命周期"问题,我们解决的是"垂直检索 + 高质量 ranking"问题。两者完全不同,但 DeerFlow 在工程纪律上有几个真正值得抄的细节(harness/app 防火墙、stable key 派生、错误转消息、trace_id),其余 17 middleware 大部分对我们是 over-engineering。**

---

## 8. 元规则(给未来的自己)

1. **任何"业界主流"判断都要实证 push**(我们已经翻车 5 次,见 §5.2)
2. **Wrapper 思路不会创造价值**(差异化在 ranking / fit / 深度处理 / 验证 / 个性化)
3. **每次新需求出现,先问"这件事别人是不是已经替我做了 AI 处理"**(白嫖原则)
4. **保持工具表面薄,把复杂留在 Pipeline 层**——工具表面变了 = host 重学;Pipeline 内部变了 = 用户无感
5. **Token 经济 > 任何架构美感**——MCP schema 太重就不用 MCP,bash 老土但训练分布满
6. **加新源前先过 §2.9 的 4 问决策树**——不满足"web_search 替代不了"就不写 adapter,只是"agent 不知道"就 skill 解决
7. **三源吸收**——DeerFlow(工程骨架)+ gstack/superpowers(skill 写作纪律)+ gbrain(fat skills)三方综合,不押单一参考
