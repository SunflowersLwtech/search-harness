# Adapter vs Skill — 数据源分类

> **用途**:把 `sources.md` 里所有候选源按"是否值得为它写 adapter"分类,作为 Tier 1 实现 scope 的**单一真相源**。
>
> **关系**:
> - **派生自** `sources.md`(全集)
> - **作用于** `architecture.md` §4 Tier 1 的实际 adapter 列表
> - **跟随** sources.md 更新而更新

---

## 1. 核心规则:Adapter 必要性测试

**满足任一即建 adapter,否则 skill-only**:

1. **它有特殊 ranking 信号**(stars / votes / citation / 下载量 / 活跃度)是 web_search 拿不到的
2. **它有结构化字段**(license / language / pushed_at / version)需要精确过滤
3. **它已经是免费 MCP**——直接 co-install,不算 adapter
4. 如果只是"agent 不知道有这个站" → **skill 解决**,不要写 adapter

> **核心**:adapter 的存在意义是 "**web_search 替代不了它的 ranking/字段**",**不是"它内容值得"**。

**结论**:Skill 解决"awareness 教学",adapter 解决"能力补全"——**两者互补,不重复**。

---

## 2. Adapter scope — Phase 1 必做(★)16 个 + Phase 2 opt-in(◇)4 个

**Phase 1(★, 16 个)** 构成 architecture.md §4 Tier 1 默认实现 scope。
**Phase 2(◇, 4 个)** 是 niche 包管理器,config 写好留着,**默认不启用**——检测到对应语言 query 或用户 `--enable-phase2` 才加载。**libraries.io(#14)对这 4 类提供 SourceRank 兜底**,Phase 1 用户即便不开 Phase 2 也有基础覆盖。
**3 个原 ★ 已降级到 skill-only**(见 §2.3)。

**Auth tier 现实**(实测后,详见 [`adapter-empirical-test-report.md`](./adapter-empirical-test-report.md)):
- 完全 anon 跑得动:**13/16** Phase 1 adapter
- **强烈推荐 30 秒升级 key**(=零配置叙事的真实下限,**实质必需**):**GitHub PAT**(code search 401)+ **Semantic Scholar key**(burst 即 429)
- 推荐升级 key:Reddit OAuth、NVD key、libraries.io key

### 2.1 Phase 1 必做(16 个 ★)

| # | 类别 | Adapter | 独家信号 / 工程注 | API 文档 |
|---|---|---|---|---|
| 1 | Repo | **GitHub** | stars / forks / license / topic;含 Code Search;**PAT 实质必需**(匿名 401) | https://docs.github.com/rest |
| 2 | Repo | **GitLab**(多实例) | 同 GitHub 字段;**多实例必备**(.com / .gnome.org / .freedesktop.org);license 需 N+1 detail call | https://docs.gitlab.com/ee/api/ |
| 3 | Q&A | **Stack Exchange**(多站) | vote / accepted / tag;一份 adapter 5 站(SO/DBA/SF/CR/AI.SE) | https://api.stackexchange.com/docs |
| 4 | Q&A | **HN Algolia** | HN 唯一时间窗 + 票数过滤(`created_at_i>` × `points>`) | https://hn.algolia.com/api |
| 5 | Q&A | **Reddit** | subreddit + `t=year` + `upvote_ratio`;**UA 必填**;OAuth 兜底待规划 | https://www.reddit.com/dev/api |
| 6 | 学术 | **arXiv** | abstract / category / 时间窗;Atom XML;多词须 `%22quote%22` | https://info.arxiv.org/help/api |
| 7 | 学术 | **Semantic Scholar** | `influentialCitationCount` 实测有效;**key 强推**(burst 即 429) | https://api.semanticscholar.org |
| 8 | 学术 | **OpenReview** | 公开评审 / decision / 反驳;二段抓取(`notes/search` → `?forum=`) | https://docs.openreview.net |
| 9 | PM 原生 | **npm** | search 内联 downloads + dependents | https://registry.npmjs.org |
| 10 | PM via deps.dev | **PyPI** | version / classifiers;**JSON API 无 downloads,无搜索**;instant 经 deps.dev,长尾经 libraries.io | https://warehouse.pypa.io/api-reference/json.html |
| 11 | PM 原生 | **crates.io** | `recent_downloads`(90d) + `sort=downloads`;UA 必填 | https://crates.io/api/v1 |
| 12 | PM via deps.dev | **pkg.go.dev → deps.dev** | version / license / advisoryKeys;**pkg.go.dev `/api/v1/*` 全返 HTML**;Go 无 popularity,经 GitHub stars 兜底 | https://api.deps.dev |
| 13 | PM via deps.dev | **Maven Central** | version / GAV / timestamp;**Solr 无 downloads/license**;license 经 deps.dev,长尾经 libraries.io | https://central.sonatype.org/search/rest-api-guide |
| 14 | 跨 PM | **libraries.io** | SourceRank + dependent_repos_count;**结构性必需**(PyPI/Maven/Go 长尾兜底 + Phase 2 4 类基础覆盖);混合 auth | https://libraries.io/api |
| 15 | AI/ML | **Hugging Face Hub** | downloads / likes / pipeline_tag;子端点 `/api/daily_papers` 提供 PwC 部分继承 | https://huggingface.co/docs/hub/api |
| 16 | 安全 | **CVE / NVD** | CVSS / CWE / CPE 版本树 / references[];canonical 唯一源 | https://nvd.nist.gov/developers/vulnerabilities |

> **架构注**:#10 / #12 / #13 三个 PM via deps.dev 共享同一个 deps.dev HTTP client(见 [`domain-knowledge.md` §2.11](./domain-knowledge.md)),实质是 1 份代码 + 3 个 source config。

### 2.2 Phase 2 opt-in(4 个 ◇)— niche 包管理器

| # | 类别 | Adapter | 数据成色 | 降级理由 |
|---|---|---|---|---|
| 17 | PM 原生 | **RubyGems** | downloads + reverse_dependencies | Ruby/RoR 持续萎缩,coding agent 用例 <5%;libraries.io SourceRank 够 MVP |
| 18 | PM 原生 | **NuGet** | totalDownloads + verified flag | .NET 企业向、OSS coding-agent 频次低;**数据本身完美**,demote 是 demographics 不是技术 |
| 19 | PM 原生 | **Hex.pm** | 4 段下载粒度(unique) | Elixir 工艺一流但社区 <2%;config 写好留一键开 |
| 20 | PM 双服务 | **CRAN**(crandb + cranlogs) | DESCRIPTION 30+ 字段 + cranlogs 下载 | R 在 dev <2%;**双服务架构 = 2× 实施成本** |

### 2.3 历史 drop(已移到 §5 skill-only)

| 原 # | Adapter | drop 原因 | 替代路径 |
|---|---|---|---|
| 8(原) | **Papers with Code** | 域名 302 → huggingface.co/papers,API 死 [2026-04-29 一] | HF Papers 经 #15 HF Hub adapter,部分继承(daily_papers 子端点) |
| 18(原) | **Conda / Anaconda** | `/search` 充斥镜像 spam(`jjhelmus/_pytorch_select` 这种);conda 包 80% 也在 PyPI;single-package 直查 = 给 PyPI 加 fallback 1 行,不值独立 adapter [2026-04-29 二] | conda 用户路由到 #10 PyPI;libraries.io 兜底 conda-only 包 |
| 22(原) | **ProductHunt** | OAuth + GraphQL 实施成本 2-4×;coding agent 流市场扫描频次极低 [2026-04-29 二] | skill `site:producthunt.com` web_search |

### 2.4 变更记录(累积)

> **[2026-04-26]** 第 2、9、15-23 共 11 个原"可选 adapter"全部纳入 Phase 1 必做 scope(理由:覆盖面=零配置体验关键变量,先做后裁比先裁后补便宜)。
>
> **[2026-04-29 一]** Papers with Code 因域名死 drop(原 #8)。Phase 1 23 → 22。同时根据实测,行 12 `pkg.go.dev` → `pkg.go.dev → api.deps.dev`,行 17 CRAN → `crandb + cranlogs` 双服务,行 18 Conda 限定 `conda-forge/<pkg>` 直查。
>
> **[2026-04-29 二]** 重新审视 C/D 档价值后:**drop 2**(Conda、ProductHunt 移到 §2.3 + §5 skill-only)+ **demote 4**(RubyGems / NuGet / Hex.pm / CRAN 移到 Phase 2)+ **架构合并 3**(PyPI / pkg.go.dev / Maven 共享 deps.dev backend,见 [`domain-knowledge.md` §2.11](./domain-knowledge.md))。**Phase 1 22 → 16 必做(★)+ 4 opt-in(◇)**,实施工作量降至原计划 ~60%。

---

### 2.5 实测注解(2026-04-29,编号已与 §2.1/§2.2 对齐)

> 来自 [`adapter-empirical-test-report.md`](./adapter-empirical-test-report.md) 的关键工程现实,§2.1/§2.2 主表里没法塞下的全在这里。

**Phase 1(★)实测注解**

| # | Adapter | 实测发现 | 对实现的影响 |
|---|---|---|---|
| 1 | GitHub | 描述关键词 miss 仍然存在(meilisearch 实测不进 top 10);PAT 实质必需(code search 401) | adapter 层无解,需 skill 层 query rewriting + cross-ref |
| 2 | GitLab | GNOME / freedesktop / Inkscape / Mesa 不在 GitHub | 多实例必备;license 字段需 `?license=true` + N+1 detail 调用 |
| 3 | Stack Exchange | `is_accepted` + 答案级 `score` + 多站(SO/DBA/SF/CR/AI.SE)都用同一 API | 一份 adapter 5 站,正向规模效应 |
| 4 | HN Algolia | `created_at_i>` + `points>` 是 HN 唯一时间窗 + 票数过滤 | 长尾首选;HN 自家搜索都用它 |
| 5 | Reddit | 匿名(UA 必填)目前 100/10min/IP 可用 | 计划 OAuth 兜底,treat 403 as adapter 信号 |
| 6 | arXiv | 多词短语必须 `%22quote%22`,否则被改写 | adapter 内置 query escaping |
| 7 | S2 | `influentialCitationCount` 实测有意义(ratio 4.5-13.1%,非线性) | **S2 key 升为强推**,匿名烈性限流 |
| 8 | OpenReview | 评审 / decision / 反驳全在 JSON `content` 字段下 | 二段抓取:`notes/search` → `notes?forum=<id>` |
| 9 | npm | search 端点内联返回 downloads | 1 call 搞定长尾,最干净 adapter |
| 10 | PyPI | **JSON API 无 downloads(`{-1,-1,-1}`),无搜索 API** | instant 经 deps.dev;长尾经 libraries.io;单包下载量经 pypistats.org |
| 11 | crates.io | `recent_downloads`(90d 滚动)+ `sort=downloads` 原生 | 不需要兜底 |
| 12 | pkg.go.dev | `/api/v1/*` 全返 HTML,**根本没有真 API**;deps.dev 也无 popularity | adapter 实际指向 api.deps.dev;长尾经 GitHub stars |
| 13 | Maven Central | Solr 无 downloads / 无 license | license 经 deps.dev;长尾经 libraries.io |
| 14 | libraries.io | 2026-04 服务仍在线,数据新鲜;package-detail 匿名可用,`/search` 需 key | 双层 auth:无 key detail-by-name,有 key 全功能 |
| 15 | HF Hub | `downloads` + `likes` + `pipeline_tag` 三连;`/api/daily_papers` 部分继承 PwC | 教科书式 adapter,零摩擦 |
| 16 | NVD | CVSS+CWE+CPE+references 全在一次 `?cveId=<id>` 返回 | 慢但稳定;long-tail keyword + severity + 时间窗一次到位 |

**Phase 2(◇)实测注解**

| # | Adapter | 实测发现 | 对实现的影响 |
|---|---|---|---|
| 17 | RubyGems | `downloads` + `reverse_dependencies` 齐;search 不按 downloads 排序 | 客户端 jq sort;config 写好留 opt-in |
| 18 | NuGet | `totalDownloads` 内联 + `verified` flag,自动加权排序 | **数据最干净**,demote 是 demographics 不是技术 |
| 19 | Hex.pm | 4 段下载粒度(all/recent/week/day),Elixir 唯一 | 干净;config 写好留 opt-in |
| 20 | CRAN | crandb 零下载;cranlogs.r-pkg.org 是 Posit 伴生;无原生搜索 | adapter = crandb + cranlogs 双服务 + `cranlogs/top` 合成发现 |

**已 drop 的实测注解(供历史回溯)**

| 原 # | Adapter | 实测发现 | 现状 |
|---|---|---|---|
| 8(原) | Papers with Code | 域名 302 → huggingface.co/papers,API `/v1/*` 全 404 | drop;HF Papers 经 #15 部分继承 |
| 18(原) | Conda | `/search` 充斥用户镜像 spam(`jjhelmus/_pytorch_select` 这种) | drop;PyPI 兜底 + libraries.io fallback |
| 22(原) | ProductHunt | OAuth 强制,无 token 连 schema introspection 都 401 | drop → skill `site:producthunt.com` |

**衍生工程发现(超出单 adapter)**:

1. **api.deps.dev 浮现为隐藏主角** — Go / Maven / PyPI 的 license / version / advisory 元数据真路径,1 份 client 服务 #10 / #12 / #13 三个 adapter,详见 [`domain-knowledge.md` §2.11](./domain-knowledge.md)
2. **libraries.io 不是"兜底"是"结构性必需"** — Phase 1 给 PyPI/Maven/Go 长尾兜底,Phase 2 给 RubyGems/NuGet/Hex/CRAN 4 类基础覆盖
3. **Adapter 真价值在跨源中间层(fan-out / RRF / cache / dedup),不在 per-source 代码** — 推动从"19 Python 类"降级到"19 YAML config + 共享 engine",见 [`domain-knowledge.md` §2.11](./domain-knowledge.md)

---

## 3. (历史)可选 adapter — 已合并入 §2

原"可选 adapter"清单于 [2026-04-26] 全部纳入 §2 Phase 1。后续如出现新候选源,走 §7 决策树评估,达标即直接进 §2。

---

## 4. Co-install MCP(不写代码,文档推荐)

让用户在 Claude Code/Cursor 配置里和我们 harness 平行装,**Microsoft / Cognition 帮我们维护**:

| MCP / Service | 维护方 | 装它干嘛 |
|---|---|---|
| **DeepWiki MCP** | Cognition | 50k+ OSS repo 的 AI wiki + Q&A,免费无 auth |
| **Context7** | Upstash | 主流库 docs grounding |
| **Ref.tools** | Ref Tools 团队 | doc + web fallback,token 极简 |
| **`@playwright/cli`** | Microsoft | browser fallback(Tier 5) |
| **invidious + companion**(Power User self-host) | iv-org | Tier 4 find-talk 稳定后端;docker-compose 一行起;给愿花 1h 部署的高级用户。详见 [`domain-knowledge.md` §5.1 #17](./domain-knowledge.md) |

→ 我们 README 里写 "Recommended companions",一行 npx/docker 配置教学。**我们一行代码不写**。

---

## 5. Skill-only 类目(数十个站,几个 skill 全覆盖)

这些站点 web_search 都能找到,**只需要 skill 教 agent "在 X 场景下 also 搜索 Y 站"**。**不写 adapter**。

### 5.1 完全 skill 化的类目

| sources.md 章节 | 处理 | Skill 形式 |
|---|---|---|
| §0.2 Phind / Perplexity | Skill | "想要 dev-focused AI 视角时,跳到 Phind" |
| §0.3 gitingest / Repomix / uithub | Skill(URL trick) | "需要整 repo 文本时,用 `uithub.com/<owner>/<repo>`" |
| §1.1 GitLab / Bitbucket / SourceHut / Codeberg / Gitee | Skill | "GitHub 找不到时,web_search 加 `site:gitlab.com`" |
| §1.2 Sourcegraph / grep.app / searchcode | Skill | "需要跨仓库正则代码搜索时" |
| §2.1-2.2 各 SDK 官方 docs(MDN/AWS/Stripe/...) | Skill + Context7 兜底 | 用 Context7 MCP 即可,长尾走 web_search |
| §3.2 Quora / baeldung / Real Python / W3Schools | Skill | "深入某语言时,可 web_search 这些教学站" |
| §4.1 Google Scholar / DBLP / 付费库(IEEE/ACM/Elsevier) | Skill | 学术补充入口(OpenReview 已升级为 §2 #9 adapter) |
| §4.2 Connected Papers / Litmaps / Inciteful / Scite | Skill(URL trick) | "想看引用脉络时,跳 Connected Papers" |
| §4.3 Distill / The Gradient / LessWrong | Skill | 长文研究博客 |
| **§5 AI 实验室 / 厂商研究博客** | Skill ✅ | "想看 OpenAI/Anthropic/DeepMind 第一手公告时" |
| §6.1 Lobste.rs / Mastodon / Bluesky / LinkedIn | Skill | 小众或边缘 |
| §6.1 X/Twitter | Skill(标注付费/法律高风险) | "如果有 Twitter API key" |
| **§6.3 中文社区**(知乎/掘金/CSDN/V2EX/Bilibili 等) | Skill ✅ | "查中文上下文时,优先 site:zhihu.com / site:juejin.cn" |
| **§6.4 日本社区**(Qiita / Zenn / はてな) | Skill ✅ | "查日文上下文时,优先 site:qiita.com / site:zenn.dev" |
| **§6.5 Discord / Slack / Matrix / IRC** | **Drop** ✅ | bot 不可达,清单意义为零 |
| §7 YC companies / IndieHackers / BetaList / Crunchbase | Skill | 市场扫描补充 |
| §7.1 年度调研报告(SO Survey / Octoverse / State of X) | Skill | "想看年度技术格局时" |
| **§8 厂商工程博客** | Skill ✅ | "查架构最佳实践时" |
| §9 长文 / 教程 / 个人博客(DEV.to / Medium / Hashnode 等) | Skill | "查 how-to 时" |
| §10 Newsletter / 周报 | Skill | 时效性话题入口 |
| §11.1 Kaggle / Replicate / Modal / LMArena 等 | Skill | "查 LLM 排名 / 跑模型时" |
| §11.2 数据集站(data.gov / 世界银行) | Skill | 找数据集时 |
| §12 OWASP / Krebs / Schneier | Skill | 安全博客 |
| §13 数据库 / 性能(DB-Engines / Use the Index / Jepsen) | Skill | 数据库选型场景 |
| §14 会议主页(NeurIPS/KubeCon/USENIX/QCon) | Skill + 配 Tier 4 transcript | "找 talk 用 web_search,看视频用 transcript adapter" |
| §15 书 / 课程 | Skill / Drop | 不太是搜索场景 |
| §16 Curated lists / Awesome / public-apis | Skill | "找清单时,fetch awesome-X 的 README" |
| §17 招聘(HN Who is hiring / Levels.fyi) | Skill | 边缘 |

### 5.2 Skill 文件的样板形态

每个 skill 不只教 "如何用 adapter",还教 **"在 X 场景下,除我们 adapter,还应该 web_search 哪些站"**:

```markdown
# find-repo SKILL.md

When user wants to find an existing OSS solution to reuse, do:

1. **Primary**: Call `search-harness find-repo <query>`
   (uses GitHub API + arXiv + DeepWiki, with stars/活跃度/license ranking)

2. **Cross-reference for community sentiment**:
   - web_search with `site:news.ycombinator.com <query>` 
   - web_search with `site:reddit.com/r/programming <query>`
   - For Chinese ecosystem: web_search with `site:juejin.cn` or `site:zhihu.com`
   - For Japanese: web_search with `site:zenn.dev` or `site:qiita.com`

3. **For deep understanding** of any candidate repo, call DeepWiki MCP's `ask_question`.

4. **For market context** (similar startups / commercial alternatives):
   - web_search with `site:ycombinator.com/companies <query>`
   - web_search with `site:producthunt.com <query>`
```

→ 这样 agent 既用了 adapter 的精度优势,又自动补全 web_search 兜底,还有 site awareness 教学。**3 件事一份 skill**。

---

## 6. 实现优先级建议(MVP scope)

### Phase 1 — MVP(必做 ★ 16 个 + opt-in ◇ 4 个)

```
Phase 1 必做(★, 16 个)
  Repo:           GitHub(+ PAT 必需), GitLab(多实例)
  Q&A:            Stack Exchange(多站), HN Algolia, Reddit
  学术:           arXiv, Semantic Scholar(+ key 强推), OpenReview
  PM 原生:        npm, crates.io
  PM via deps.dev: PyPI, pkg.go.dev → deps.dev, Maven Central
                   (3 个共享 1 份 deps.dev client + libraries.io 兜底)
  跨 PM:          libraries.io(混合 auth)
  AI/ML:          Hugging Face Hub(+ /api/daily_papers PwC 部分继承)
  安全:           CVE / NVD

Phase 2 opt-in(◇, 4 个)— niche PM,默认不启
  RubyGems, NuGet, Hex.pm, CRAN(双服务)
  → 检测到对应语言 query 或 --enable-phase2 触发

Drop / skill-only(原 ★ 3 个)
  Papers with Code(域名死)→ HF Papers 经 #15
  Conda /search(spam channels)→ PyPI 兜底
  ProductHunt(OAuth 重)→ skill `site:producthunt.com`

实施次序(经 2026-04-29 二次实测重排):
  Wave A (零摩擦高 ROI,Phase 1):
    GitHub, arXiv, Stack Exchange, HN Algolia, npm, crates.io,
    NVD, Hugging Face Hub
  Wave B (Phase 1,要工程考量):
    Semantic Scholar(强推 key), Reddit(规划 OAuth),
    OpenReview(2-stage), libraries.io(双 auth), GitLab(多实例)
  Wave C (Phase 1,共享 deps.dev):
    deps.dev client(写一次)→ PyPI / pkg.go.dev / Maven 三 config
  Phase 2(◇ opt-in,实施时机视真实需求):
    RubyGems, NuGet, Hex.pm, CRAN

3-5 个 skill:
  find-repo / ground-api / search-paper / search-discuss

Co-install 推荐:
  DeepWiki MCP / Context7 / @playwright/cli
```

### Phase 2 — Power user(opt-in,需 key)

```
Tier 3-4 multimodal adapter:
  Tavily / Exa / Brave web_search wrapper(opt-in key)
  YouTube transcript
  Gemini video understanding
```

### Phase 3 — 长尾

```
继续根据真实使用反馈,把"频繁出现的 web_search site:" → 升级为 adapter
(数据驱动,不预测)
```

---

## 7. 决策原则的反向应用

**未来加新源时,先过这 4 个问题**:

```
Q1. Web search 能不能找到它?
    ├─ 不能(login wall / API only)→ adapter 或 drop
    └─ 能 →  Q2

Q2. Web search 能不能拿到它的核心 ranking 信号?
    ├─ 不能(stars / votes / citation 等)→ adapter
    └─ 能 → Q3

Q3. 它已经有免费 MCP / API 暴露给 agent 了吗?
    ├─ 是 → co-install,不算 adapter
    └─ 否 → Q4

Q4. agent 知道这个站存在吗?
    ├─ 不知道 → 加进 skill 的 "cross-reference" 段
    └─ 知道 → 不需要任何动作
```

**这个决策树挂在 sources.md 的"自查问题 §6"或这份文档,作为筛选 rubric。**

---

## 附录:与其他文档的关系

| 文档 | 关系 |
|---|---|
| `sources.md` | **全集**(枚举,不预判) |
| **本文档** | **筛过的子集 + 处理方式**(adapter/skill/co-install/drop) |
| `architecture.md` §4 | 引用本文档 §2 作为 Tier 1 实际 adapter 列表 |
| `domain-knowledge.md` §5.1 | "adapter vs skill 边界测试"作为已验证判断 |
