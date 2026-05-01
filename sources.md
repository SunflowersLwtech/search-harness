# Sources

## 1. Phase 1 ★ adapters(20)

| # | 来源 | 端点 | Auth | 独家信号 |
|---|---|---|---|---|
| 1 | **GitHub** | `https://api.github.com/search/{repositories,code,issues}` + `gh` CLI | `Authorization: token $GITHUB_TOKEN` | `stars` / `forks` / `license` / `topic`;Code Search;Discussions / Gists 走 `api.github.com/graphql` |
| 2 | **GitLab**(多实例) | `https://{gitlab.com,gitlab.gnome.org,gitlab.freedesktop.org}/api/v4/...` | `PRIVATE-TOKEN: $GITLAB_TOKEN` | `stars` / `license` / `pushed_at`;`?license=true` 需 N+1 detail |
| 3 | **Stack Exchange**(多站) | `https://api.stackexchange.com/2.3/search/advanced?site={stackoverflow,dba,serverfault,codereview,ai}` | anon | `vote` / `is_accepted` / `score` |
| 4 | **HN Algolia** | `https://hn.algolia.com/api/v1/search?query=&numericFilters=created_at_i>...,points>...` | anon | `created_at_i>` × `points>` 时间窗 + 票数过滤 |
| 5 | **Reddit** | `https://www.reddit.com/r/{sub}/search.json?q=&t=year&restrict_sr=1` | anon(UA 必填),OAuth 升 60-100 req/min | `subreddit` / `upvote_ratio` / `score` |
| 6 | **arXiv** | `http://export.arxiv.org/api/query?search_query=&start=&max_results=` | anon | `abstract` / `category` / 时间窗;Atom XML;多词须 `%22quote%22` |
| 7 | **OpenAlex** | `https://api.openalex.org/works?search=&filter=...` | anon,UA 加 `mailto:` 进 polite pool | `fwci` + `citation_normalized_percentile.is_in_top_1_percent` + `concepts` + `counts_by_year` + `is_retracted` + `referenced_works` |
| 8 | **OpenReview** | `https://api2.openreview.net/notes/search` → `notes?forum=<id>` | anon | 公开评审 / decision / 反驳;二段抓取 |
| 9 | **npm** | `https://registry.npmjs.org/-/v1/search?text=` | anon | `downloads.weekly` + `dependents` 内联 |
| 10 | **crates.io** | `https://crates.io/api/v1/crates?q=&sort=downloads` | anon(UA 必填) | `recent_downloads`(90d 滚动)+ `sort=downloads` |
| 11 | **libraries.io** | `https://libraries.io/api/{platform}/{name}` + `/api/search?platforms=&q=&api_key=...` | detail anon / `/search` 需 key | `SourceRank` + `dependent_repos_count` |
| 12 | **Hugging Face Hub** | `https://huggingface.co/api/{models,datasets,spaces}?search=` + `/api/daily_papers` | anon | `downloads` + `likes` + `pipeline_tag` |
| 13 | **CVE / NVD** | `https://services.nvd.nist.gov/rest/json/cves/2.0?cveId=&keywordSearch=&pubStartDate=` | anon(key 推荐) | CVSS + CWE + CPE 版本树 + `references[]` |
| 14 | **twitterapi.io** | `https://api.twitterapi.io/twitter/{tweet/advanced_search,user/info,user/last_tweets,tweet/replies}` | `X-API-Key: $TWITTERAPI_IO_API_KEY` | `viewCount` + `isBlueVerified` + `likeCount` + `retweetCount` + `pinnedTweetIds[0]` + `replies[]` |
| 15 | **OSV.dev** | `POST https://api.osv.dev/v1/query` body `{"package":{"name","ecosystem":"PyPI\|npm\|Go\|Maven\|crates.io\|RubyGems\|Hex\|NuGet"},"version"}` + `/v1/querybatch` + `GET /v1/vulns/{id}` | anon | package+version → vulns 反向映射;CVSS 3.1 + `affected[]` 版本树 |
| 16 | **deps.dev**(Google OSI) | `GET https://api.deps.dev/v3/systems/{npm,pypi,maven,go,cargo,nuget,rubygems}/packages/<name>/versions/<ver>` | anon | 7 ecosystem 一个 schema;`licenses`(SPDX)+ `advisoryKeys`(直链 OSV)+ `links.SOURCE_REPO` + `slsaProvenances` + `attestations` |
| 17 | **Lobste.rs** | `https://lobste.rs/{hottest,newest}.json` + `/t/<tag>.json` | anon(UA 推荐) | tag + `score` + `comment_count` + `submitter_user` |
| 18 | **Bluesky**(AT-Protocol) | 1️⃣ `POST https://bsky.social/xrpc/com.atproto.server.createSession` `{identifier,password}` 拿 `accessJwt`<br>2️⃣ `GET /xrpc/app.bsky.feed.{searchPosts,getAuthorFeed,getPostThread}` + `app.bsky.actor.{searchActors,getProfile}` | `Authorization: Bearer $accessJwt`(从 `BLUESKY_HANDLE` + `BLUESKY_APP_PASSWORD` 换) | `likeCount` + `repostCount` + `replyCount` + `did`;authed searchPosts 无 rate limit |
| 19 | **Sourcegraph stream** | `GET https://sourcegraph.com/.api/search/stream?q=context:global+...` | anon(`Accept: text/event-stream`) | 跨 2M+ OSS repo trigram + facet(`repo:has.topic` / `lang:` / `type:symbol`)+ commit SHA + lineNumber |
| 20 | **Exa** | `POST https://api.exa.ai/search` body `{query, type:"auto\|fast\|instant\|deep-lite\|deep\|deep-reasoning", num_results, contents:{highlights\|text\|summary}, outputSchema?, maxAgeHours?, includeDomains?, excludeDomains?}` + `POST /contents` for known URLs | `x-api-key: $EXA_API_KEY` | neural 语义搜索(非词法);**`outputSchema` 结构化 JSON + 字段级 grounding citations**;`type:deep` multi-hop(WebWalker 81%);`maxAgeHours` 强制 livecrawl freshness;Code/People/Company vertical |

twitterapi.io free-tier QPS = 1 req / 5s,串行 + sleep 5s。

---

## 2. Co-install(全局 CLI 优先 / MCP 仅在无 CLI 替代时)

| 工具 | 维护方 | 形态 | 用途 | 安装 |
|---|---|---|---|---|
| **Context7 CLI**(`ctx7`) | Upstash | CLI | 主流库 docs grounding;`ctx7 library <name> <q>` + `ctx7 docs <libId> <q>` | `npm install -g ctx7 && npx ctx7 setup` |
| **Playwright CLI** | Microsoft | CLI | browser automation / JS-rendered scrape / login walls | `npm install -g playwright && npx playwright install chromium` |
| **DeepWiki MCP** | Cognition / Devin | MCP | 50k+ OSS repo AI wiki + Q&A | `claude mcp add --transport http deepwiki https://mcp.deepwiki.com/mcp` |
| **yt-dlp** | yt-dlp 团队 | CLI | `yt-dlp "ytsearch5:..."`(搜)+ `--dump-single-json <url>`(metadata)+ `--write-auto-sub --sub-langs en --sub-format vtt`(transcript);1000+ 视频站 | `brew install yt-dlp` |

---

## 3. 官方开发者文档

| 输入形态 | 工具 | 端点 / 命令 | Auth |
|---|---|---|---|
| 主流库 docs grounding | Context7 CLI(§2) | `ctx7 docs <libId> <q>` | OAuth |
| 任意静态 URL → markdown(RFC / mkdocs / sphinx / 静态 HTML) | **Jina Reader** | `curl https://r.jina.ai/<url>` | 无 key |
| SPA / login wall / JS-heavy 站(Mintlify / GitBook 类) | **Firecrawl** | `POST https://api.firecrawl.dev/v1/scrape` body `{url, formats:["markdown"]}` | `Authorization: Bearer $FIRECRAWL_API_KEY` |
| vendor 发 OpenAPI YAML/JSON | **openapi-to-skills** | `npx openapi-to-skills <yaml-url> -o .claude/skills/` | 无(本地) |

---

## 4. Agent 自带 web search(last-resort fallback)

§1 #20 Exa 失败时,host LLM 自带 `web_search` 工具承接(Claude / Gemini / GPT 都自带)。**不在我们调用清单**——search-agent 用尽时的 graceful degradation。
