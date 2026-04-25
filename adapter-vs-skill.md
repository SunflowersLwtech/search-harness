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

## 2. 必做 adapter — 12 个核心(★)

这 12 个 adapter 构成 architecture.md §4 Tier 1 的实现 scope。**全部零 key 可用**(GitHub PAT 是唯一推荐升级)。

| # | Adapter | 独家信号 | API 文档 |
|---|---|---|---|
| 1 | **GitHub** | stars / forks / 活跃度 / language / license / topic;含 Code Search + Issues | https://docs.github.com/rest |
| 2 | **arXiv** | abstract / category / 时间窗 / 作者 | https://info.arxiv.org/help/api |
| 3 | **Stack Exchange**(SO + DBA + ServerFault + CodeReview + AI.SE 等) | vote / accepted / tag / score | https://api.stackexchange.com/docs |
| 4 | **HN Algolia** | HN 全文搜索 + 时间窗 + score | https://hn.algolia.com/api |
| 5 | **Reddit** | subreddit filter / time window / sort | https://www.reddit.com/dev/api |
| 6 | **Semantic Scholar** | 引用图 / 被引数 / related papers | https://api.semanticscholar.org |
| 7 | **Papers with Code** | 论文 ↔ 代码 ↔ benchmark 链接 | https://paperswithcode.com/api/v1/docs |
| 8 | **npm** | weekly downloads / version / dependents | https://registry.npmjs.org |
| 9 | **PyPI** | downloads / version / classifiers | https://warehouse.pypa.io/api-reference/json.html |
| 10 | **crates.io** | downloads / version / dependents | https://crates.io/api/v1 |
| 11 | **pkg.go.dev** | imported by / version / module path | https://pkg.go.dev/about#api |
| 12 | **Maven Central** | version / artifact metadata | https://central.sonatype.org/search/rest-api-guide |

---

## 3. 可选 adapter(留待筛选决定)

不在 MVP scope,待用户根据使用场景挑选:

| Adapter | 何时值得做 |
|---|---|
| **libraries.io** | 想"跨 28 包管理器统一搜索"而非分别接 4-5 个,可替代 #8-#12 |
| **OpenReview** | 学术深度用户,关心 ICLR/NeurIPS workshop 评审过程 |
| **ProductHunt** | 市场扫描场景需求强 |
| **Hugging Face Hub** | AI/ML 模型/数据集场景需求强 |
| **CVE/NVD/Snyk DB** | scope 含"安全审计 / 供应链漏洞" |
| **GitLab** | 用户重度依赖 GitLab 上的 OSS(GNOME/Inkscape/Tor 系) |
| **RubyGems / NuGet / Hex / CRAN / Conda** | 用户重度用对应语言时按需接 |

---

## 4. Co-install MCP(不写代码,文档推荐)

让用户在 Claude Code/Cursor 配置里和我们 harness 平行装,**Microsoft / Cognition 帮我们维护**:

| MCP | 维护方 | 装它干嘛 |
|---|---|---|
| **DeepWiki MCP** | Cognition | 50k+ OSS repo 的 AI wiki + Q&A,免费无 auth |
| **Context7** | Upstash | 主流库 docs grounding |
| **Ref.tools** | Ref Tools 团队 | doc + web fallback,token 极简 |
| **`@playwright/cli`** | Microsoft | browser fallback(Tier 5) |

→ 我们 README 里写 "Recommended companions",一行 npx/json 配置教学。**我们一行代码不写**。

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
| §4.1 Google Scholar / DBLP / OpenReview(可选)/ 付费库 | Skill | 学术补充入口 |
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

### Phase 1 — MVP(必做)

```
12 核心 adapter(零 key 优先实现):
  GitHub / arXiv / Stack Exchange / HN Algolia / Reddit
  Semantic Scholar / Papers with Code
  npm / PyPI / crates.io / pkg.go.dev / Maven Central

3-5 个 skill:
  find-repo / ground-api / search-paper / search-discuss

Co-install 推荐:
  DeepWiki MCP / Context7 / @playwright/cli
```

### Phase 2 — Power user(可选)

```
可选 adapter 按用户场景 demand 增量加:
  libraries.io / OpenReview / Hugging Face Hub / CVE-NVD …

Tier 3-4 multimodal adapter:
  Tavily/Exa/Brave web_search wrapper(opt-in key)
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
