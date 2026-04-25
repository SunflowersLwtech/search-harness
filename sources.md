# Coding-domain Search Sources — 枚举清单

按"实际查询场景"分类。**不考虑成本、不预判取舍**,你自己 ✅ 标关注的、❌ 划掉不要的。

每行字段:
- **源**:站点/服务名
- **URL**:主入口
- **用途**:你为什么会去查它(1 行)
- **API**:✅ 有干净公开 API / ⚠️ 受限/付费 / ❌ 仅 HTML

---

## 0. AI-Augmented 二级源(已预处理,白嫖别人的 LLM 算力)

**这一类不是原始数据源,而是别人已经用 AI 处理过的衍生视图**——你直接消费,省掉自己重复做 LLM 处理的 token + 时间。

### 0.1 仓库理解(repo wiki + Q&A)

| 源 | URL | 用途 | API/MCP |
|---|---|---|---|
| **DeepWiki**(Cognition / Devin 团队) | https://deepwiki.com | 已为 **50k+ 主流 OSS repo** 预生成 AI wiki + 架构图 + 模块级解释 + Q&A 接口 | ✅ **官方 MCP server,完全免费无 auth**:`ask_question` / `read_wiki_contents` / `read_wiki_structure` |
| DeepWiki MCP Server (官方) | https://cognition.ai/blog/deepwiki-mcp-server | DeepWiki 暴露给 Claude Code/Cursor 的 MCP 入口 | ✅ 免费 |
| deepwiki-open (AsyncFuncAI) | https://github.com/AsyncFuncAI/deepwiki-open | OSS 自托管版,可指向**私有 repo / 内网 GitLab** | OSS |
| OpenDeepWiki (AIDotNet) | https://github.com/AIDotNet/OpenDeepWiki | C#/TS 实现,模块化,适合企业内集成 | OSS |

### 0.2 开发者垂直 AI 搜索

| 源 | URL | 用途 |
|---|---|---|
| Phind | https://www.phind.com | 专为 coding query 优化的 AI 搜索(Perplexity-for-devs) |
| Perplexity (Coding mode) | https://www.perplexity.ai | 通用 AI 搜索,coding 类 query 命中率高 |

### 0.3 Repo→LLM 准备工具(预处理基础设施)

| 源 | URL | 用途 |
|---|---|---|
| gitingest | https://gitingest.com | URL 替换:`gitingest.com/<owner>/<repo>` → 整 repo 的 LLM-ready 文本 |
| Repomix | https://github.com/yamadashy/repomix | CLI:把整 repo 打包成 LLM 友好单文件,支持过滤 |
| GitMCP | https://github.com/idosal/git-mcp | 把任意 git repo 变成 MCP server 暴露给 agent |
| uithub.com | https://uithub.com | URL 替换技巧:`github.com/x/y` → `uithub.com/x/y` 取全 repo Markdown |

---

## 1. 代码 / Repo / SDK 发现

### 1.1 主流 git 托管平台

| 源 | URL | 用途 | API |
|---|---|---|---|
| GitHub | https://github.com | repo / code / issue / PR / discussion 全文搜索;主战场 | ✅ REST + GraphQL |
| GitLab | https://gitlab.com | 第二大托管平台;部分大型 OSS 在这 (GNOME / Inkscape / Tor) | ✅ REST |
| Bitbucket | https://bitbucket.org | Atlassian 系企业项目,Mercurial 历史 | ✅ |
| SourceHut (sr.ht) | https://sr.ht | 极简平台,Drew DeVault 系 OSS 在此 | ✅ |
| Codeberg | https://codeberg.org | 非营利 Gitea,FOSS 社区迁移目的地 | ✅ |
| Gitea (各实例) | https://gitea.com | 自托管 git 平台主流方案 | ✅ |
| Gitee | https://gitee.com | 中国国产 git 托管,部分国内项目仅在此 | ✅ |

### 1.2 跨仓库代码搜索

> **想要"理解"某个 repo 而不是搜代码?去看 [§0.1 DeepWiki](#01-仓库理解repo-wiki--qa)**——它已经预生成 AI wiki,免费 MCP 调用,比自己 fetch README + 解析强一个量级。

| 源 | URL | 用途 | API |
|---|---|---|---|
| Sourcegraph (public) | https://sourcegraph.com/search | 跨开源仓库符号级搜索 | ✅ GraphQL |
| grep.app | https://grep.app | 全 GitHub 正则代码搜索,极快 | ❌ 网页 |
| searchcode.com | https://searchcode.com | 老牌跨平台代码搜索 | ✅ |

### 1.3 包管理器 / 依赖注册中心

| 源 | URL | 用途 | API |
|---|---|---|---|
| npm | https://www.npmjs.com | JS/Node 包,下载量、版本、依赖 | ✅ |
| PyPI | https://pypi.org | Python 包 | ✅ |
| crates.io | https://crates.io | Rust 包 | ✅ |
| pkg.go.dev | https://pkg.go.dev | Go 模块,文档+源码 | ✅ |
| Maven Central | https://search.maven.org | Java/Kotlin/Scala | ✅ |
| RubyGems | https://rubygems.org | Ruby | ✅ |
| Packagist | https://packagist.org | PHP/Composer | ✅ |
| NuGet | https://www.nuget.org | .NET | ✅ |
| pub.dev | https://pub.dev | Dart/Flutter | ✅ |
| Hex.pm | https://hex.pm | Elixir/Erlang | ✅ |
| Hackage | https://hackage.haskell.org | Haskell | ✅ |
| CRAN | https://cran.r-project.org | R | ✅ |
| CPAN | https://metacpan.org | Perl | ✅ |
| Conda / Anaconda | https://anaconda.org | Python+科学计算 | ✅ |
| Homebrew formulae | https://formulae.brew.sh | macOS/Linux 包 | ✅ |
| AUR | https://aur.archlinux.org | Arch 用户仓库 | ✅ |
| Docker Hub | https://hub.docker.com | 容器镜像 | ✅ |
| Quay.io | https://quay.io | Red Hat 容器镜像 | ✅ |
| npm trends | https://npmtrends.com | npm 包热度趋势对比 | ❌ |
| bundlephobia | https://bundlephobia.com | npm 包 bundle 大小 | ✅ |
| star-history.com | https://star-history.com | GitHub stars 历史曲线 | ❌ |
| libraries.io | https://libraries.io | **跨 30+ 包管理器统一搜索 + 依赖图** | ✅ |
| Snyk Advisor | https://snyk.io/advisor | 包健康度评分(安全+维护) | ⚠️ 部分付费 |
| Socket.dev | https://socket.dev | npm/PyPI 供应链安全 | ⚠️ |

---

## 2. 文档 / API 参考

### 2.1 通用文档聚合

| 源 | URL | 用途 | API |
|---|---|---|---|
| DevDocs.io | https://devdocs.io | 离线聚合主流语言/框架文档 | ❌ 但开源 |
| ReadTheDocs | https://readthedocs.org | Python 系项目文档主托管 | ✅ |
| MDN Web Docs | https://developer.mozilla.org | Web 开发圣经 (HTML/CSS/JS/Web API) | ❌ 但页面规整 |
| Microsoft Learn | https://learn.microsoft.com | 微软全栈文档 (Azure/.NET/TypeScript) | ❌ |
| Apple Developer | https://developer.apple.com | iOS/macOS/watchOS/Swift | ❌ |
| Android Developer | https://developer.android.com | Android/Kotlin | ❌ |
| Google Developers | https://developers.google.com | Google APIs/SDKs | ❌ |
| Web.dev | https://web.dev | Google 现代 web 最佳实践 | ❌ |
| caniuse.com | https://caniuse.com | 浏览器特性兼容矩阵 | ✅ JSON dump |
| caniemail.com | https://www.caniemail.com | 邮件客户端兼容矩阵 | ✅ |

### 2.2 平台/SDK 官方 docs(示例,实际数百个)

| 源 | URL | 用途 |
|---|---|---|
| AWS docs | https://docs.aws.amazon.com | AWS 全服务 |
| GCP docs | https://cloud.google.com/docs | Google Cloud |
| Azure docs | https://learn.microsoft.com/azure | Azure |
| Cloudflare docs | https://developers.cloudflare.com | CF Workers / R2 / D1 |
| Vercel docs | https://vercel.com/docs | Vercel + Next.js 部署 |
| Stripe docs | https://docs.stripe.com | 支付 |
| Twilio docs | https://www.twilio.com/docs | 通信 |
| OpenAI docs | https://platform.openai.com/docs | OpenAI API |
| OpenAI cookbook | https://github.com/openai/openai-cookbook | 工作代码示例 |
| Anthropic docs | https://docs.anthropic.com | Claude API |
| Anthropic cookbook | https://github.com/anthropics/anthropic-cookbook | 工作代码示例 |
| Google AI docs | https://ai.google.dev | Gemini API |
| Hugging Face docs | https://huggingface.co/docs | Transformers / Datasets / Hub |
| Kubernetes docs | https://kubernetes.io/docs | K8s 全集 |
| Postman API Network | https://www.postman.com/explore | 公共 API 集合 + 文档 |
| RapidAPI Hub | https://rapidapi.com/hub | API 市场 |

### 2.3 文档 grounding 服务(MCP/SaaS)

| 源 | URL | 用途 | API |
|---|---|---|---|
| Context7 | https://context7.com | 主流库 docs 预索引 MCP | ✅ MCP |
| Ref.tools | https://ref.tools | docs + web fallback,token 极简 | ✅ MCP |
| docs-mcp-server | https://github.com/arabold/docs-mcp-server | OSS 自托管 docs MCP | ✅ MCP |

---

## 3. Q&A / Debug

### 3.1 Stack Exchange 全家桶

| 源 | URL | 用途 | API |
|---|---|---|---|
| Stack Overflow | https://stackoverflow.com | 编程 how-to / debug 第一站 | ✅ Stack Exchange API |
| Server Fault | https://serverfault.com | 系统/网络管理 | ✅ |
| Super User | https://superuser.com | 桌面/通用计算机问题 | ✅ |
| DBA Stack Exchange | https://dba.stackexchange.com | 数据库专家 | ✅ |
| Code Review | https://codereview.stackexchange.com | 代码评审 | ✅ |
| Software Engineering | https://softwareengineering.stackexchange.com | 设计/架构问题 | ✅ |
| Cross Validated | https://stats.stackexchange.com | 统计/ML 数学 | ✅ |
| AI Stack Exchange | https://ai.stackexchange.com | AI 技术问答 | ✅ |
| Data Science SE | https://datascience.stackexchange.com | 数据科学 | ✅ |
| Mathematics SE | https://math.stackexchange.com | 数学 | ✅ |

### 3.2 其他 Q&A / 知识

| 源 | URL | 用途 | API |
|---|---|---|---|
| GitHub Issues(各 repo) | (按 repo) | 调用方 bug 真实归宿,常比 SO 更新 | ✅ GitHub API |
| GitHub Discussions(各 repo) | (按 repo) | 项目方主导的 Q&A | ✅ |
| Quora | https://www.quora.com | 通用问答,有时有专家 | ❌ |
| baeldung.com | https://www.baeldung.com | Java 生态深度教程 | ❌ |
| GeeksforGeeks | https://www.geeksforgeeks.org | 算法/编程入门题解 | ❌ |
| Real Python | https://realpython.com | Python 教程站 | ❌ |
| W3Schools | https://www.w3schools.com | 入门级 web 文档 | ❌ |

---

## 4. 学术论文 / 研究

### 4.1 论文库与索引

| 源 | URL | 用途 | API |
|---|---|---|---|
| arXiv | https://arxiv.org | CS 系顶级 preprint(cs.LG/CL/SE/AI/IR) | ✅ |
| Papers with Code | https://paperswithcode.com | **论文 + 代码 + benchmark 排行榜** | ✅ |
| Semantic Scholar | https://www.semanticscholar.org | 引用图、被引数、related papers | ✅ S2 API |
| Google Scholar | https://scholar.google.com | 学术搜索默认入口 | ❌ 无开放 API |
| OpenReview | https://openreview.net | ICLR/NeurIPS workshop 公开评审 | ✅ |
| ACL Anthology | https://aclanthology.org | NLP 顶会论文 | ✅ |
| DBLP | https://dblp.org | CS 文献数据库 | ✅ |
| IEEE Xplore | https://ieeexplore.ieee.org | IEEE 会议/期刊 | ⚠️ 多数付费 |
| ACM Digital Library | https://dl.acm.org | ACM 会议/期刊 | ⚠️ 多数付费 |
| ScienceDirect (Elsevier) | https://www.sciencedirect.com | 期刊主站 | ⚠️ 付费 |
| SpringerLink | https://link.springer.com | Springer 期刊/书籍 | ⚠️ 部分付费 |
| bioRxiv | https://www.biorxiv.org | 生物 preprint(ML+生物有交叉) | ✅ |
| SSRN | https://www.ssrn.com | 经济/管理/金融 preprint | ✅ |
| HAL | https://hal.science | 法国开放学术仓库 | ✅ |
| Zenodo | https://zenodo.org | CERN 通用学术数据集 | ✅ |

### 4.2 引用关系可视化

| 源 | URL | 用途 | API |
|---|---|---|---|
| Connected Papers | https://www.connectedpapers.com | 论文关联图谱可视化 | ❌ |
| Litmaps | https://www.litmaps.com | 引用脉络追踪 | ⚠️ 付费 |
| Inciteful | https://inciteful.xyz | 论文图谱探索 | ❌ |
| Scite.ai | https://scite.ai | 引用上下文(支持/反对) | ⚠️ 付费 |
| ResearchRabbit | https://www.researchrabbit.ai | 论文推荐 + 时间线 | ❌ |
| ResearchGate | https://www.researchgate.net | 论文社交平台 | ❌ |

### 4.3 顶级研究博客 / 解读

| 源 | URL | 用途 | API |
|---|---|---|---|
| Distill.pub | https://distill.pub | 可视化机器学习论文 | ❌ |
| The Gradient | https://thegradient.pub | AI 长文解读 | ❌ |
| AI Alignment Forum | https://www.alignmentforum.org | AI 安全研究 | ❌ |
| LessWrong | https://www.lesswrong.com | 理性主义/AI 安全 | ✅ GraphQL |

---

## 5. AI 实验室 / 厂商研究博客

| 源 | URL | 用途 |
|---|---|---|
| OpenAI Blog | https://openai.com/research | 研究公告 |
| Anthropic Research | https://www.anthropic.com/research | Claude 研究 |
| DeepMind Blog | https://deepmind.google/discover/blog | 研究公告 |
| Google AI Blog | https://research.google/blog | Google 研究 |
| Meta AI Research | https://ai.meta.com/blog | Meta AI 研究 |
| Microsoft Research | https://www.microsoft.com/en-us/research/blog | MSR |
| NVIDIA Developer Blog | https://developer.nvidia.com/blog | GPU/CUDA/AI |
| Apple ML Research | https://machinelearning.apple.com | Apple ML |
| Salesforce Research | https://www.salesforce.com/blog/category/ai | Salesforce |
| Hugging Face Blog | https://huggingface.co/blog | OSS LLM 生态 |
| Cohere Blog | https://cohere.com/blog | LLM 厂商 |
| Mistral Blog | https://mistral.ai/news | 法国开源 LLM |
| DeepSeek 博客 | https://www.deepseek.com/blog | 国产 LLM |
| 通义千问博客 (Qwen) | https://qwenlm.github.io | 阿里 LLM |
| 智谱 AI | https://www.zhipuai.cn | 国产 LLM |
| MiniMax | https://www.minimax.io | 国产 LLM |
| Moonshot Kimi | https://www.moonshot.cn | 国产 LLM |

---

## 6. 社区讨论 / 口碑 / sentiment

### 6.1 英文主流社区

| 源 | URL | 用途 | API |
|---|---|---|---|
| Hacker News | https://news.ycombinator.com | Show HN / Ask HN / 技术讨论 | ✅ Algolia 全文搜索 |
| Reddit | https://www.reddit.com | 各 subreddit 极活跃 | ✅(2023 收紧但可用) |
| Lobste.rs | https://lobste.rs | 比 HN 小、技术含量高 | ✅ JSON feed |
| X / Twitter | https://x.com | Builder 社区实时讨论 | ⚠️ **付费/Nitter 镜像** |
| Mastodon (fosstodon) | https://fosstodon.org | FOSS 社区 | ✅ |
| Bluesky | https://bsky.app | 部分 dev 迁移目的地 | ✅ AT Protocol |
| LinkedIn | https://www.linkedin.com | 工程领导/招聘观察 | ⚠️ 受限 |

### 6.2 关键 subreddits(自查清单)

```
r/programming
r/MachineLearning
r/LocalLLaMA          ← LLM 自部署社区
r/LLMDevs
r/learnprogramming
r/ExperiencedDevs
r/cscareerquestions
r/webdev
r/javascript / r/typescript / r/reactjs / r/node
r/python
r/rust
r/golang
r/cpp / r/csharp / r/java / r/dotnet / r/kotlin / r/swift
r/devops / r/kubernetes / r/docker / r/sre
r/databases / r/PostgreSQL
r/sysadmin
r/sideproject / r/SaaS / r/Entrepreneur / r/startups
r/selfhosted
r/opensource
r/datascience / r/learnmachinelearning
r/coding
```

### 6.3 中文社区

| 源 | URL | 用途 |
|---|---|---|
| 知乎 | https://www.zhihu.com | 中文技术问答+长文 |
| 掘金 | https://juejin.cn | 国内开发者社区,前端尤其活跃 |
| CSDN | https://www.csdn.net | 国内最大技术博客(质量参差) |
| SegmentFault 思否 | https://segmentfault.com | 国内 Stack Overflow 仿 |
| V2EX | https://www.v2ex.com | 程序员社交 |
| 即刻 | https://web.okjike.com | 独立开发者/产品讨论 |
| 微博 | https://weibo.com | 行业实时讨论 |
| Bilibili | https://www.bilibili.com | 技术视频教程 |
| InfoQ 中文 | https://www.infoq.cn | 行业新闻深度报道 |
| 开源中国 | https://www.oschina.net | 国内 OSS 资讯 |
| 51CTO | https://www.51cto.com | 老牌 IT 站 |

### 6.4 日本社区

| 源 | URL | 用途 |
|---|---|---|
| Qiita | https://qiita.com | 日本最大开发者博客 |
| Zenn | https://zenn.dev | 新一代日本开发者社区 |
| はてなブックマーク | https://b.hatena.ne.jp | 日本 HN 类 |

### 6.5 实时/聊天(难爬,但内容极有价值)

| 源 | 用途 | 可达性 |
|---|---|---|
| Discord 各项目服务器 | 项目方+用户实时支持 | ❌ 需登录 |
| Slack 各项目 workspace(K8s/Rust/Kubernetes 等) | 同上 | ❌ |
| Matrix(matrix.org) | FOSS 通信网络 | ⚠️ |
| IRC(libera.chat) | 老牌 FOSS 通信 | ✅(纯文本,可桥接) |
| Discourse 论坛 | 项目专属论坛 | ✅ |

---

## 7. 创业 / 市场 / 竞品情报

| 源 | URL | 用途 | API |
|---|---|---|---|
| Y Combinator Companies | https://www.ycombinator.com/companies | YC 投资组合,看赛道 | ❌ |
| Hacker News(Show HN) | https://news.ycombinator.com/show | 新工具发布 | ✅ |
| ProductHunt | https://www.producthunt.com | 新产品发布投票 | ✅ |
| Indie Hackers | https://www.indiehackers.com | 独立开发者市场 + 收入披露 | ❌ |
| Crunchbase | https://www.crunchbase.com | 融资/公司数据 | ⚠️ 付费 |
| Wellfound (AngelList) | https://wellfound.com | 创业公司 + 招聘 | ⚠️ 受限 |
| BetaList | https://betalist.com | beta 阶段产品 | ❌ |
| Launching Next | https://www.launchingnext.com | 新产品聚合 | ❌ |
| Pitchbook | https://pitchbook.com | 私募/风投数据库 | ⚠️ 付费 |
| CB Insights | https://www.cbinsights.com | 行业洞察报告 | ⚠️ 付费 |
| TechCrunch | https://techcrunch.com | 创业新闻 | ❌ |
| The Information | https://www.theinformation.com | 高质量科技新闻 | ⚠️ 付费 |

### 7.1 年度调研报告

| 源 | URL | 用途 |
|---|---|---|
| Stack Overflow Developer Survey | https://survey.stackoverflow.co | 全球开发者年度调研 |
| GitHub Octoverse | https://octoverse.github.com | GitHub 生态年报 |
| State of JS / CSS / HTML / GraphQL | https://stateofjs.com | 前端各领域年度调研 |
| State of AI Report | https://www.stateof.ai | AI 行业年度报告 |
| JetBrains Developer Ecosystem | https://www.jetbrains.com/lp/devecosystem | JetBrains 年度调研 |
| ThoughtWorks Tech Radar | https://www.thoughtworks.com/radar | 技术雷达 |
| InfoQ Trends | https://www.infoq.com/articles/architecture-trends | 行业趋势 |

---

## 8. 厂商工程博客(架构/最佳实践)

| 公司 | URL |
|---|---|
| Stripe | https://stripe.com/blog/engineering |
| Vercel | https://vercel.com/blog |
| Cloudflare | https://blog.cloudflare.com |
| GitHub | https://github.blog/engineering |
| GitLab | https://about.gitlab.com/blog |
| Netflix Tech | https://netflixtechblog.com |
| Uber Engineering | https://www.uber.com/blog/engineering |
| Airbnb Engineering | https://medium.com/airbnb-engineering |
| LinkedIn Engineering | https://www.linkedin.com/blog/engineering |
| Meta Engineering | https://engineering.fb.com |
| Google Cloud Blog | https://cloud.google.com/blog |
| AWS Blog | https://aws.amazon.com/blogs |
| Microsoft Engineering | https://devblogs.microsoft.com |
| Shopify Engineering | https://shopify.engineering |
| Slack Engineering | https://slack.engineering |
| Notion Engineering | https://www.notion.so/blog/topic/eng |
| Figma Blog | https://www.figma.com/blog |
| Spotify R&D | https://engineering.atspotify.com |
| Dropbox Engineering | https://dropbox.tech |
| Discord Engineering | https://discord.com/blog/category/engineering |
| DoorDash Engineering | https://doordash.engineering |
| Lyft Engineering | https://eng.lyft.com |
| Twitch Engineering | https://blog.twitch.tv/en/tags/engineering |
| Pinterest Engineering | https://medium.com/pinterest-engineering |
| Reddit Engineering | https://www.redditinc.com/blog |
| Atlassian Engineering | https://www.atlassian.com/engineering |
| Datadog Engineering | https://www.datadoghq.com/blog/engineering |
| Honeycomb | https://www.honeycomb.io/blog |
| Mozilla Hacks | https://hacks.mozilla.org |
| Replit Blog | https://blog.replit.com |
| Linear Engineering | https://linear.app/blog |
| Sentry Blog | https://blog.sentry.io |

---

## 9. 长文章 / 教程 / 个人博客

### 9.1 聚合平台

| 源 | URL | 用途 |
|---|---|---|
| DEV.to | https://dev.to | 开发者博客平台 |
| Hashnode | https://hashnode.com | 同上,新一代 |
| Medium(技术) | https://medium.com | 仍有大量技术写作 |
| Substack(技术) | https://substack.com | newsletter 平台 |
| HackerNoon | https://hackernoon.com | 技术博客聚合 |
| Towards Data Science | https://towardsdatascience.com | DS/ML 长文 |
| KDnuggets | https://www.kdnuggets.com | DS/AI 老牌资讯 |
| Towards AI | https://towardsai.net | AI 教程 |
| freeCodeCamp | https://www.freecodecamp.org/news | 免费技术教程 |
| DigitalOcean Tutorials | https://www.digitalocean.com/community/tutorials | 系统/部署教程 |

### 9.2 经典个人博客 / 大牛站

| 源 | URL | 用途 |
|---|---|---|
| Martin Fowler | https://martinfowler.com | 架构/重构圣经 |
| Joel on Software | https://www.joelonsoftware.com | Joel Spolsky 历史文章 |
| Coding Horror | https://blog.codinghorror.com | Jeff Atwood (SO 创始人) |
| The Old New Thing | https://devblogs.microsoft.com/oldnewthing | Raymond Chen,Windows 内幕 |
| Daniel Stenberg | https://daniel.haxx.se | curl 作者 |
| LWN.net | https://lwn.net | Linux 内核新闻 |
| Phoronix | https://www.phoronix.com | Linux/性能跑分 |
| The Pragmatic Engineer | https://newsletter.pragmaticengineer.com | Gergely Orosz,工程文化/管理 |
| Bytebytego | https://bytebytego.com | 系统设计 |
| High Scalability | http://highscalability.com | 大规模架构 |
| Daring Fireball | https://daringfireball.net | Apple 生态 |
| Fabien Sanglard | https://fabiensanglard.net | 经典代码逆向解读 |

### 9.3 主流技术媒体

| 源 | URL | 用途 |
|---|---|---|
| Ars Technica | https://arstechnica.com | 高质量科技报道 |
| The Register | https://www.theregister.com | 英国技术新闻 |
| ZDNet | https://www.zdnet.com | 企业 IT |
| Wired | https://www.wired.com | 科技+文化 |
| MIT Tech Review | https://www.technologyreview.com | MIT 出品技术评论 |
| IEEE Spectrum | https://spectrum.ieee.org | IEEE 杂志 |

---

## 10. Newsletter / News 聚合

| 源 | URL | 用途 |
|---|---|---|
| The Changelog | https://changelog.com | OSS 周报+播客 |
| Console.dev | https://console.dev | 开发者工具周报 |
| TLDR Tech | https://tldr.tech | 全行业每日摘要 |
| Bytes.dev | https://bytes.dev | JS 周报 |
| JavaScript Weekly | https://javascriptweekly.com | JS 周报 |
| Python Weekly | https://www.pythonweekly.com | Python 周报 |
| Rust Weekly (This Week in Rust) | https://this-week-in-rust.org | Rust 周报 |
| Go Weekly | https://golangweekly.com | Go 周报 |
| Ruby Weekly | https://rubyweekly.com | Ruby 周报 |
| Postgres Weekly | https://postgresweekly.com | Postgres 周报 |
| KubeWeekly | https://www.cncf.io/kubeweekly | K8s 周报 |
| Awesome Newsletters | https://github.com/zudochkin/awesome-newsletters | 周报的元清单 |

---

## 11. AI / ML 专门源

### 11.1 模型 / 数据 / Benchmark

| 源 | URL | 用途 | API |
|---|---|---|---|
| Hugging Face Hub | https://huggingface.co | 模型/数据集/Spaces 中心 | ✅ |
| Kaggle | https://www.kaggle.com | 数据集 + 比赛 | ✅ |
| Replicate | https://replicate.com | 模型 hosting + 调用 | ✅ |
| Modal | https://modal.com | ML 部署平台 | ✅ |
| LMArena | https://lmarena.ai | LLM 对战排行榜 | ❌ |
| Open LLM Leaderboard | https://huggingface.co/spaces/open-llm-leaderboard/open_llm_leaderboard | 开源 LLM 排行 | ❌ |
| AlpacaEval | https://tatsu-lab.github.io/alpaca_eval | LLM eval 排行 | ❌ |
| Chatbot Arena | https://chat.lmsys.org | LMSYS 对战 | ❌ |
| HumanEval / MBPP / SWE-bench | (各 GitHub) | 代码 LLM 标准 benchmark | ✅ |

### 11.2 数据集

| 源 | URL |
|---|---|
| Awesome Public Datasets | https://github.com/awesomedata/awesome-public-datasets |
| data.gov | https://data.gov |
| World Bank Open Data | https://data.worldbank.org |
| UN Data | http://data.un.org |
| Google Dataset Search | https://datasetsearch.research.google.com |

---

## 12. 安全

| 源 | URL | 用途 | API |
|---|---|---|---|
| CVE Mitre | https://cve.mitre.org | CVE 主索引 | ✅ |
| NVD | https://nvd.nist.gov | NIST 漏洞数据库 | ✅ |
| GitHub Advisory Database | https://github.com/advisories | GitHub 安全警告 | ✅ |
| Snyk Vulnerability DB | https://security.snyk.io | 包级漏洞 | ✅ |
| Exploit-DB | https://www.exploit-db.com | 公开 exploit | ❌ |
| OWASP | https://owasp.org | Web 安全最佳实践 | ❌ |
| HackerOne reports | https://hackerone.com/hacktivity | 公开 bug bounty 报告 | ⚠️ |
| Bugcrowd | https://www.bugcrowd.com | 同上 | ⚠️ |
| Krebs on Security | https://krebsonsecurity.com | 安全新闻 | ❌ |
| Schneier on Security | https://www.schneier.com | Bruce Schneier 博客 | ❌ |
| Risky Business | https://risky.biz | 安全行业播客 | ❌ |

---

## 13. 数据库 / 性能

| 源 | URL | 用途 |
|---|---|---|
| DB-Engines.com | https://db-engines.com | 数据库流行度排行 + 对比 |
| Use the Index, Luke! | https://use-the-index-luke.com | SQL 索引最佳实践 |
| PlanetScale Blog | https://planetscale.com/blog | 数据库工程 |
| Postgres Weekly | https://postgresweekly.com | Postgres 周报 |
| Aphyr / Jepsen | https://jepsen.io | 分布式系统正确性测试 |
| The Morning Paper | https://blog.acolyer.org | 论文每日解读(已停更但 archive 极有价值) |

---

## 14. 视频 / 会议

### 14.1 会议(几乎都在 YouTube 有官方频道)

| 会议 | URL | 用途 |
|---|---|---|
| NeurIPS | https://nips.cc | ML 顶会 |
| ICML | https://icml.cc | ML 顶会 |
| ICLR | https://iclr.cc | ML 顶会 |
| ACL | https://www.aclweb.org | NLP 顶会 |
| USENIX | https://www.usenix.org | 安全/系统 |
| SOSP / OSDI | https://www.sigops.org | 操作系统顶会 |
| POPL / PLDI | https://popl24.sigplan.org | 编程语言 |
| KubeCon | https://www.cncf.io/kubecon-cloudnativecon-events | K8s 大会 |
| AWS re:Invent | https://reinvent.awsevents.com | AWS 年会 |
| Google I/O | https://io.google | Google 年会 |
| Apple WWDC | https://developer.apple.com/wwdc | Apple 年会 |
| Microsoft Build | https://build.microsoft.com | 微软年会 |
| React Conf / JSConf | (各自) | JS 会议 |
| PyCon | https://us.pycon.org | Python 年会 |
| Strange Loop | https://thestrangeloop.com | 编程语言+系统跨界 |
| QCon | https://qconferences.com | 工程实践 |
| GOTO Conference | https://gotopia.tech | 工程实践 |

### 14.2 视频平台

| 源 | URL | 用途 |
|---|---|---|
| YouTube | https://www.youtube.com | 几乎所有会议 talk + 教程 |
| Bilibili(中文) | https://www.bilibili.com | 中文技术视频 |
| Frontend Masters | https://frontendmasters.com | 付费前端课程 |
| Pluralsight | https://www.pluralsight.com | 付费技术课程 |
| Egghead.io | https://egghead.io | 付费短视频教程 |
| LinuxFoundation Training | https://training.linuxfoundation.org | Linux/CNCF 课程 |

---

## 15. 书 / 课程 / 训练

### 15.1 书

| 源 | URL | 用途 |
|---|---|---|
| O'Reilly Online Learning | https://www.oreilly.com/online-learning | 付费技术书+课程 |
| Manning | https://www.manning.com | 付费技术书 |
| Pragmatic Bookshelf | https://pragprog.com | 付费技术书 |
| Free Programming Books | https://github.com/EbookFoundation/free-programming-books | 免费书清单 |

### 15.2 训练 / 算法

| 源 | URL | 用途 |
|---|---|---|
| LeetCode | https://leetcode.com | 算法刷题 |
| HackerRank | https://www.hackerrank.com | 算法刷题 |
| CodeWars | https://www.codewars.com | 算法 kata |
| Exercism | https://exercism.org | 多语言练习 + 导师 |
| Advent of Code | https://adventofcode.com | 年度圣诞节挑战 |
| Project Euler | https://projecteuler.net | 数学+编程题 |

---

## 16. Curated Lists / Meta-resources

| 源 | URL | 用途 |
|---|---|---|
| sindresorhus/awesome | https://github.com/sindresorhus/awesome | **awesome 列表的元清单** |
| public-apis | https://github.com/public-apis/public-apis | 公共 API 清单 |
| system-design-primer | https://github.com/donnemartin/system-design-primer | 系统设计入门 |
| you-dont-know-js | https://github.com/getify/You-Dont-Know-JS | JS 深度学习 |
| 30-seconds-of-code | https://www.30secondsofcode.org | 短代码片段 |
| free-programming-books | https://github.com/EbookFoundation/free-programming-books | 免费书 |
| awesome-mcp-servers | https://github.com/punkpeye/awesome-mcp-servers | MCP server 清单 |
| awesome-llm | https://github.com/Hannibal046/Awesome-LLM | LLM 清单 |
| awesome-machine-learning | https://github.com/josephmisiti/awesome-machine-learning | ML 清单 |

---

## 17. 招聘 / 行业动态(间接信号)

| 源 | URL | 用途 |
|---|---|---|
| HN "Who is hiring?" 月度帖 | https://news.ycombinator.com/submitted?id=whoishiring | 看哪些公司在招聘 = 哪些技术栈在用 |
| Levels.fyi | https://www.levels.fyi | 薪资数据 |
| LinkedIn 工程招聘 | https://www.linkedin.com | 工程方向信号 |

---

## 自查问题(筛选时可参照)

筛的时候问自己:

1. **过去一周我会查这个源吗?** 不查 → 直接 ❌
2. **它有干净 API 吗?** 没 API 但内容贵重 → 标记 "需要爬虫/Tavily 兜底"
3. **它和我的 4 个核心场景**(找 repo / 论文 / 市场 / API doc)对得上吗? 对不上但你看重 → 加新场景
4. **它的内容会"过时"吗?** 教程/年报这种过时快;论文/issue 不过时
5. **覆盖到我做比赛和项目最常碰的技术栈了吗?**(列一下你常用的语言/框架,看每一个有没有覆盖)
6. **这件事是不是别人已经替我做了 AI 处理?**(对比 §0)→ 如果有现成的 AI-augmented 二级源,优先白嫖,不要自己跑 LLM

---

## 待你反馈的下一步

筛选完之后,告诉我你打勾保留的源,我帮你按以下维度二次分析:
- 哪几个**必须有原生集成**(API 直连)
- 哪几个**应该靠 Tavily/Exa 兜底**(无 API)
- 哪几个**真的需要专门设计 fit ranking**(因为通用 ranking 不够好)
- 整体能拼成几条"垂直 pipeline"
