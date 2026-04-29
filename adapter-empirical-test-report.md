# Adapter 实测报告 — 23 个核心(★) 是否真解决问题?

> **目的**:对 [`adapter-vs-skill.md` §2](./adapter-vs-skill.md) 的 23 个 Tier-1 必做 adapter 做实测,逐项验证它们是否真能为 **long-tail 研究任务**或 **instant 任务**提供 web_search 替代不了的 ranking 信号 / 结构化字段。
>
> **方法**:6 个并行 subagent,按"Repo / Q&A / 学术 / 主流包管理 / 小众包管理 / 特殊"6 类目分发,每个 adapter 跑 1 个 instant + 1 个 long-tail query,curl 真接 API,jq 解响应,与 web_search 能力对比给出判决。
>
> **判决档位**:`both`(instant + long-tail 双强)/ `instant-strong` / `long-tail-strong` / `drop`,附置信度 H/M/L。
>
> **生成时间**:2026-04-29
>
> **关联文档**:[`adapter-vs-skill.md` §2](./adapter-vs-skill.md) | [`domain-knowledge.md` §5.1 #14](./domain-knowledge.md) | [`architecture.md` §11](./architecture.md)

---

## 0. 摘要(TL;DR)

| 类别 | 总数 | 真过 | 待降级 | 必须 drop |
|---|---|---|---|---|
| Repo | 2 | 2 | 0 | 0 |
| Q&A | 3 | 3 | 0 | 0 |
| 学术 | 4 | 3 | 0 | **1**(Papers with Code 已死) |
| 主流包管理 | 5 | 2 | 3(架构改造) | 0 |
| 小众包管理 | 5 | 3 | 1(CRAN 重) | 1(Conda 搜索) |
| 特殊 | 4 | 3 | 1(ProductHunt 缓后) | 0 |
| **合计** | **23** | **16** | **5** | **2(强降)** |

**核心修正**:
1. **Papers with Code 站已 302 全部重定向到 huggingface.co/papers** — adapter 直接 drop,Phase 1 从 23 降到 **22**
2. **GitHub PAT 从"推荐"升为"实质必需"** — 匿名 code search 401(此前已记录,本次再确认)
3. **PyPI / Maven / pkg.go.dev 三家都没有原生搜索或下载量字段** — 长尾排序必须经 libraries.io 或 GitHub 兜底
4. **api.deps.dev (Google Open Source Insights) 浮现为隐藏主角** — Go/Maven/PyPI 的 license + version 元数据真正可靠的统一来源,值得后续考虑作为 cross-cut helper
5. **libraries.io 比预期还活着**(2026-04 数据新鲜) — 但 `/search` 需 key,package-detail 端点匿名可用,这个**两层 auth 现实**要写进 adapter 设计

---

## 1. 完整判决矩阵

| # | 类目 | Adapter | 判决 | Long-tail | Instant | Conf | 工程注解 |
|---|---|---|---|---|---|---|---|
| 1 | Repo | **GitHub** | both | ✓ | ✓ | H | PAT 实质必需(code search 401);描述关键词 miss 不可救于 adapter 层 |
| 2 | Repo | **GitLab** | both | ✓ | ✓ | H | 多实例必备(.com/.gnome.org/.freedesktop.org);N+1 license 调用 |
| 3 | Q&A | **Stack Exchange** | both | ✓ | ✓ | H | 一个 adapter 覆盖 SO/DBA/SF/CodeReview/AI.SE 5 站;`is_accepted` + 答案级 `score` 不可外抽 |
| 4 | Q&A | **HN Algolia** | long-tail-strong | ✓ | ~ | H | 唯一支持 `created_at_i>` + `points>` filter 的 HN 搜索 |
| 5 | Q&A | **Reddit** | long-tail-strong | ✓ | - | M | 匿名(UA 必填)目前可用,2023 后趋紧;OAuth 升级路径开放 |
| 6 | 学术 | **arXiv** | both | ✓ | ✓ | H | Atom XML;多词短语必须 `%22quote%22` |
| 7 | 学术 | **Semantic Scholar** | both | ✓ | ✓ H 信号 / M-H 工程 | `influentialCitationCount` 验证为独立信号(非 citationCount 线性);匿名烈性限流,**S2 key 升为强推** |
| 8 | 学术 | ~~**Papers with Code**~~ | **DROP** | - | - | H | **域名 302 → HF Papers,API 全死**。从 §2 删除,Phase 1 23→22 |
| 9 | 学术 | **OpenReview** | long-tail-strong | ✓ | ~ | H | 评审/决议/反驳结构化齐全;需 `notes?forum=<id>` 二段抓取 |
| 10 | 包管理 | **npm** | both | ✓ | ✓ | H | search 端点内联返回 downloads,无需二次调用;最干净的 adapter |
| 11 | 包管理 | **PyPI** | instant-strong | ✗→兜底 | ✓ | H | **JSON API 无 downloads,XMLRPC search 已废**;长尾经 libraries.io / pypistats |
| 12 | 包管理 | **crates.io** | both | ✓ | ✓ | H | `sort=downloads` + `recent_downloads` 原生工作;UA 必填 |
| 13 | 包管理 | **pkg.go.dev** → deps.dev | instant-via-deps.dev | 弱 | ✓ | H | **pkg.go.dev/api/v1/* 全返 HTML(404)** — 真路径是 `api.deps.dev`;Go 无原生 popularity 信号 |
| 14 | 包管理 | **Maven Central** | instant-strong | ✗→兜底 | ✓ | H | Solr 无 downloads / 无 license;pair deps.dev 拿 license,长尾经 libraries.io |
| 15 | 包管理 | **RubyGems** | both | ✓ | ✓ | H | `downloads` + `reverse_dependencies`;search 排序需客户端 |
| 16 | 包管理 | **NuGet** | both | ✓ | ✓ | H | `totalDownloads` 内联 + `verified` flag;搜索原生按下载排序 |
| 17 | 包管理 | **Hex.pm** | both | ✓ | ✓ | H | **唯一 4 段下载粒度**(all/recent/week/day);`sort=downloads` 原生 |
| 18 | 包管理 | **CRAN** (crandb + cranlogs) | both-via-pair | ✓ | ✓ | M | crandb 零下载量;需 cranlogs.r-pkg.org 配对;crandb 无原生搜索 |
| 19 | 包管理 | **Conda / Anaconda** | instant-only;搜索 drop | ✗ | ~ | M | 搜索充斥用户镜像 spam channels;`conda-forge/<pkg>` 直查可用,`/search` 不可用于发现 |
| 20 | 跨包管理器 | **libraries.io** | both(降级 instant) | ✓ key / ✗ anon | ✓ | M | package-detail 匿名可用;`/search` 需 30s 注册 key;**单 host bus-factor 风险** |
| 21 | AI/ML | **Hugging Face Hub** | both | ✓ | ✓ | H | `downloads` + `likes` + `pipeline_tag` 三连;教科书式案例 |
| 22 | 安全 | **CVE / NVD** | both | ✓ | ✓ | H | CVSS / CWE / CPE 版本树是 canonical 唯一源;upstream 慢但稳定 |
| 23 | 市场 | **ProductHunt** | (理论)instant-strong | ~ | ~ | L | **OAuth 强制 + GraphQL**,2-4× 实现成本;无 token 连 schema 都无法 introspect — 推荐 Phase 1 缓后或保留 skill-only |

**判决分布**:
- `both` (10): GitHub, GitLab, SE, arXiv, S2, npm, crates.io, RubyGems, NuGet, Hex, HF, NVD = **12 个**
- `long-tail-strong` (3): HN Algolia, Reddit, OpenReview = **3 个**
- `instant-strong` (3): PyPI, Maven, pkg.go.dev/deps.dev = **3 个**
- `instant-only / 搜索 drop` (1): Conda = **1 个**
- `condition: needs-key for full power` (1): libraries.io = **1 个**
- `defer / OAuth heavy` (1): ProductHunt = **1 个**
- `DROP outright` (1): **Papers with Code = 1 个**

---

## 2. 对 long-tail 任务"事实意义"分级

> **Long-tail = 研究式深查**(rank top-N by signal,跨时间/源汇总,等)
>
> 标准:adapter 必须返回 web_search 不可外抽的 **排序信号**(stars / votes / citations / downloads / influence)+ 结构化筛选字段(time / venue / tag / category)。

### A 档:杠杆最强(Wave A 必上)

| Adapter | 关键独家长尾能力 |
|---|---|
| **GitHub** | `sort=stars` × `language` × `archived` 联合过滤 — 通用 OSS 发现唯一规模化路径 |
| **Semantic Scholar** | `influentialCitationCount`(实测 ratio 4.5%~13.1% 有意义分布) + reference 图遍历 |
| **OpenReview** | 公开评审 / decision / metareview / 反驳的结构化 JSON,**互联网上唯一来源** |
| **HN Algolia** | `created_at_i>` × `points>` 时间窗 + 票数双过滤 — HN 自带搜索都用它 |

### B 档:垂直化必须(Wave B 上)

| Adapter | 长尾能力 |
|---|---|
| **GitLab** | 多实例发现独占项目(GNOME / freedesktop / Inkscape / Mesa) |
| **Stack Exchange** | 一个 API 5 站,`tag` × `score` × time-window |
| **Reddit** | subreddit-restrict + `t=year` + `upvote_ratio`(社区 sentiment) |
| **arXiv** | `cat:cs.LG` × `submittedDate` 时间排序 + 完整 abstract |
| **npm** | search 内联 downloads + `dependents`(weekly/monthly) |
| **crates.io** | `sort=downloads` × categories + `recent_downloads`(90 天滚动) |
| **NuGet** | `totalDownloads` × `verified` flag,自动加权排序 |
| **Hex.pm** | 4 段下载粒度(all/recent/week/day),细到天 |
| **Hugging Face Hub** | `pipeline_tag`(canonical task taxonomy)× `downloads` × `likes` |
| **CVE/NVD** | CVSS × CWE × `pubStartDate` 时间窗 — 漏洞研究 canonical |
| **libraries.io** | SourceRank + `dependent_repos_count` 跨 30+ 包管理器统一 |

### C 档:长尾弱、要兜底

| Adapter | 弱点 | 兜底 |
|---|---|---|
| **PyPI** | 无下载量 + 无搜索 API | 长尾完全经 libraries.io;`pypistats.org` 拿单包下载量 |
| **Maven Central** | Solr 无下载量,`sort=downloads` 静默忽略 | libraries.io 兜底 popularity;deps.dev 兜底 license |
| **pkg.go.dev** | 无 API,deps.dev 无 popularity 信号 | GitHub stars(经 `repo` 字段)兜底 popularity |
| **CRAN** | crandb 无下载,需 cranlogs 配对;无搜索 | 实现 = `cranlogs/top` + `crandb/<pkg>` 双调用合成 |
| **RubyGems** | search 不按 downloads 排序 | 客户端排序(取整页后 jq sort) |

### D 档:long-tail 不适用

| Adapter | 原因 |
|---|---|
| **Conda /search** | 充斥 user-mirror spam,**信号不可信** — 长尾必须 hardcode `conda-forge/bioconda/pytorch` 频道白名单或绕开 |
| **ProductHunt** | OAuth 障碍 + GraphQL 复杂度让"长尾市场扫描"代价远高于 `site:producthunt.com` skill |
| ~~**Papers with Code**~~ | **死** |

---

## 3. 对 instant 任务"事实意义"分级

> **Instant = 快速 factoid 抓取**(specific repo's stars / package's latest version + license / paper's citation count / CVE detail)
>
> 标准:adapter 必须返回 web_search 不可可靠抽取的 **结构化字段**(version / license / pushed_at / CVSS / 等)。

### A 档:基本盘,所有 ⭐ 适用

几乎所有 22 个(去掉 PwC)在 instant 任务上都达标。**这是 adapter 与 wrapper 的真正分界**:web_search 给的是 HTML/snippet,adapter 给的是 schema 化字段。

**最强 instant**:
- **NVD** — `CVE-2021-44228` 一行 query 出 CVSS 10.0 + CWE 4 个 + 103 references + CPE 版本树 + 时间线
- **HF Hub** — 一行 query 出 model 的 downloads / likes / pipeline_tag / lastModified
- **GitHub / GitLab** — `repos/owner/name` 一行出 stars+forks+pushed_at+SPDX license+topics
- **所有 registry**(npm/PyPI/crates/RubyGems/NuGet/Hex/Maven)— version + license + maintainer 是它们的核心契约

### 边缘 instant(因 auth / 架构变重)

| Adapter | 注意 |
|---|---|
| **CRAN** | 元数据 instant 强(30+ DESCRIPTION 字段),但下载量需第二个 host |
| **libraries.io** | 单包查询匿名可用(意外发现);跨包搜索需 key |
| **ProductHunt** | OAuth 才能 introspect,instant 也被 auth 挡 |

### Drop

- ~~**Papers with Code**~~

---

## 4. 跨切发现(超越单个 adapter)

### 4.1 `api.deps.dev`(Google OSI)是隐藏主角

实测中浮现:
- pkg.go.dev 的 `/api/v1/*` 全返 HTML — Go 元数据真路径是 deps.dev
- Maven Central Solr 无 license — license 在 deps.dev
- 同一 host 也支持 PyPI / crates / npm / NuGet / RubyGems 的 license + advisory keys

**建议**:作为 Phase 1 后期/Phase 2 的 cross-cut helper(类似 libraries.io 的角色),不是单独 adapter,而是若干 adapter 的内部 fallback。**优先级 P2**,不影响 22 adapter scope。

### 4.2 "无搜索 API" 三家(PyPI / Maven / CRAN / pkg.go.dev)的统一兜底

它们的长尾能力都缺失。**libraries.io 在这里就是 Phase 1 必备**——不是兜底,是结构化必需(尤其 PyPI / Maven)。这强化了 #20 的进 Phase 1 决策。

### 4.3 多实例 / 多站点 adapter 的隐藏成本

- **Stack Exchange**:1 个 adapter 处理 SO/DBA/SF/CodeReview/AI.SE,`site=` 参数切换 — **正向规模效应**
- **GitLab**:1 个 adapter 必须处理 .com/.gnome.org/.freedesktop.org 三实例,license 字段还需 `?license=true` + N+1 detail 调用 — **负向规模成本**

设计时要把这两类区分清楚。

### 4.4 Auth-tier 实测画像(影响"零配置"叙事)

| Tier | 成员 |
|---|---|
| **匿名完全可用** | arXiv, HN Algolia, OpenReview, npm, crates.io(UA), Maven Central, RubyGems, NuGet, Hex.pm, CRAN crandb, Anaconda(单包), HF Hub(只读), NVD |
| **匿名跑得动但限流烈,key 强推** | Semantic Scholar(秒级触发 429), GitHub(60/hr → PAT 5000/hr;code search 401), Reddit(UA 必填,趋紧) |
| **混合 auth**(部分端点 anon,核心需 key) | libraries.io(detail anon, search 需 key) |
| **OAuth-only** | ProductHunt |
| **Dead** | ~~Papers with Code~~ |

→ **零配置叙事仍然成立**(13/22 完全匿名),**但 GitHub PAT + S2 key 应文档为"30 秒强烈推荐",不是"可选"**。

### 4.5 Drop / 降级总账

| Adapter | 当前状态 → 建议状态 |
|---|---|
| **Papers with Code** | ★ 必做 → **从 §2 表格删除**(域名死) |
| **Conda /search 端点** | 完整 adapter → **adapter 保留,但搜索接口不暴露**;只允许 `conda-forge/<pkg>` 直查 |
| **PyPI 长尾** | adapter 内部实现 → **路由到 libraries.io 或 pypistats**(skill 层面) |
| **Maven 长尾** | adapter 内部实现 → **路由到 libraries.io**(skill 层面) |
| **pkg.go.dev** | 名字保留 → **真名 `go-deps-dev`**(adapter 实现指向 api.deps.dev) |
| **ProductHunt** | Wave C → **建议缓到 Phase 1.5 或保留 skill-only**(OAuth 不符合零配置精神) |

---

## 5. 修订后的 Phase 1 scope

**确认 22 个核心 adapter**(从 23 减去 PwC):

```
Repo:           GitHub, GitLab(多实例)
Q&A:            Stack Exchange(多站), HN Algolia, Reddit
学术:           arXiv, Semantic Scholar, OpenReview               [PwC 已 drop]
包管理(10):     npm, PyPI(无搜索), crates.io, go-deps-dev,
                Maven Central(无 popularity), RubyGems, NuGet,
                Hex.pm, CRAN(双服务), Conda(只单包查)
跨包管理器:     libraries.io(混合 auth)
AI/ML:          Hugging Face Hub
安全:           CVE / NVD
市场:           ProductHunt(建议缓后)
```

**修订实施次序**:

```
Wave A (零摩擦高 ROI,实测全部干净):
   GitHub(+ PAT 文档), arXiv, Stack Exchange, HN Algolia, npm, crates.io, Hex.pm, NuGet, NVD, Hugging Face Hub
   → 10 个,全 zero-key 或 30s key,数据清,工程量低

Wave B (要工程考量但仍核心):
   PyPI(指清"无搜索 API,经 libraries.io"), Reddit(规划 OAuth 兜底), 
   Semantic Scholar(强推 key), OpenReview(2-stage fetch), libraries.io(双 auth 模式),
   GitLab(多实例)
   → 6 个

Wave C (重 / 降级 / 缓):
   RubyGems, Maven Central(deps.dev 配对), go-deps-dev,
   CRAN(crandb+cranlogs 双服务), Conda(单包查模式)
   → 5 个

Phase 1.5 / 缓:
   ProductHunt
```

---

## 6. 对 architecture / adapter-vs-skill 文档的待落变更

> 不在本报告执行,列在这里供下次更新批量处理。

| 文档 | 变更 |
|---|---|
| `adapter-vs-skill.md` §2 表格 | 删除第 8 行 Papers with Code;Phase 1 总数 23 → 22;给第 7 行 S2 加注"key 强推";给第 13 行 pkg.go.dev 改名为 `go-deps-dev` 备注;给第 19 行 Conda 加"仅单包查"备注 |
| `architecture.md` §3.2 / §7 | GitHub PAT 措辞从"30 秒推荐升级"改为"实质必需(code search 匿名 401)" — 此变更在 [domain-knowledge.md §5.1 #14] 已待落,本报告再确认 |
| `architecture.md` §4 Tier 1 行 | "23 个核心 adapter" → "22 个核心 adapter" |
| `domain-knowledge.md` §5.1 #11 | 加一条"PwC 已死,2026-04-29 验证 302 → HF Papers" |
| `domain-knowledge.md` §1.7 / §5 | 加 `api.deps.dev`(Google OSI)条目作为 Go/Maven/PyPI cross-cut 元数据兜底 |
| `domain-knowledge.md` 新条目 | "**deps.dev 浮现**为统一 license + version metadata 后端 — 22 adapter 中 4 个 PM 都受益" |

---

## 7. 元发现(给未来的自己)

1. **半年前文档里的"必做"今天可能死掉** — Papers with Code 是教科书案例。**adapter scope 文档至少季度复测**。
2. **"有 API 文档不等于有 API"** — pkg.go.dev `/api/v1/*` 网页上明文列着,实测全返 HTML。**永远 curl 真接,别只读 docs**。
3. **"零下载量"是隐藏的 long-tail kill switch** — PyPI / Maven / CRAN / Conda 都跌进这个坑。**没有原生 popularity 信号 = 长尾必须兜底**,不是可选。
4. **多实例 adapter ≠ 多站点 adapter** — SE 是正向规模(一份代码 5 站),GitLab 是负向规模(多实例 + N+1 调用)。设计前要分清。
5. **"匿名可用"分两档** — 完全可用(13 个) vs 跑得动但要立刻配 key(GitHub / S2 / Reddit)。零配置叙事要实事求是,**不能装作 GitHub PAT 是可选**。
6. **deps.dev 这种"白嫖大厂的 metadata 后端"是 §3 元规则的实证**——下次设计前先问"Google / Microsoft / NVD 是不是已经替我做好了"。

---

## 附:测试命令样例(可复跑)

```bash
# 1. GitHub 描述关键词 miss
xh GET https://api.github.com/search/repositories \
  q=='real-time analytics database language:rust' sort=stars order=desc | jq '.items[].full_name'
# meilisearch / clickhouse 不在 — 同 §5.1 #14

# 2. PwC 已死
xh -h GET https://paperswithcode.com/api/v1/papers/  # 302 → huggingface.co/papers/trending

# 3. S2 影响力信号验证
xh GET 'https://api.semanticscholar.org/graph/v1/paper/search' \
  query=='attention is all you need' fields==citationCount,influentialCitationCount limit==1 | jq

# 4. PyPI 无下载量
xh GET https://pypi.org/pypi/requests/json | jq '.info.downloads'   # {-1, -1, -1}

# 5. pkg.go.dev API 真假
xh -h GET 'https://pkg.go.dev/api/v1/package/github.com/gin-gonic/gin' | grep -i content-type
# text/html — 假;真路径在 api.deps.dev

# 6. libraries.io 包详情匿名可用
xh GET https://libraries.io/api/PyPI/requests | jq '.rank, .dependent_repos_count'

# 7. NVD canonical 结构化
xh GET 'https://services.nvd.nist.gov/rest/json/cves/2.0?cveId=CVE-2021-44228' | jq '.vulnerabilities[0].cve.metrics.cvssMetricV31[0].cvssData.baseScore'
# 10.0
```
