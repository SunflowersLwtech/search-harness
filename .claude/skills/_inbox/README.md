# `_inbox/` — 待筛选 skills 暂存

## 用途

每个 `.md` 是一个**复杂 source 的待选 skill 草稿**:粗记录 "为什么需要 + 封装什么 + 输入/输出形态",**等以后筛选 / 扩写 / drop**。

不是生产级 SKILL.md(没有完整 imperative / cookbook / 错误处理)——只是**占位 + 元信息**,供后续决策。

## 当前 8 候选(对应 [`domain-knowledge.md` §5.1 #23 F](../../../domain-knowledge.md))

| 文件 | 复杂在哪 | 简单 source 不写 skill 的对照 |
|---|---|---|
| `bluesky-auth-search.md` | 3-step auth(createSession → JWT → searchPosts) | HN Algolia / arXiv 1-line curl |
| `twitterapi-default-pack.md` | 默认 query 套餐 + 1/5s sleep + 字段提取 | npm search 内联 downloads |
| `pm-info-fusion.md` | PyPI/Maven/Go 三源融合(native + deps.dev + libraries.io) | crates.io 单源 |
| `sourcegraph-stream-parse.md` | SSE 解析(`event: matches` 不是 JSON) | OpenAlex 单 JSON response |
| `reddit-oauth-tier.md` | OAuth client_credentials → bearer → oauth.reddit.com(用量大时) | anon Reddit 100/10min 已够 |
| `openreview-two-stage.md` | `notes/search` → `notes?forum=<id>` 二段抓取 | Stack Exchange 单 query |
| `osv-deep-references.md` | vulns + 跟每条 `references[].url` 元数据 | CVE/NVD 单 cveId 一次返完 |
| `exa-output-schema-cookbook.md` | 常用 outputSchema 模板复用 | npm search 不需要模板 |

## 决策门槛(用以后再筛)

每个候选要回答:
1. 实际用到几次?(< 3 次/月 → drop)
2. 模型能不能 raw curl 自己搞?(能 → drop,加进 sources.md inline 注解即可)
3. 写一份 50-100 行 bash 比让模型每次重做省多少 token?(< 30% → drop)
4. 跟 `case_study/scripts/test-*.sh` 现有模板能否合并?(能 → 不开新文件)

通过 4 关再扩写为生产级 SKILL.md,提升到 `.claude/skills/<name>/SKILL.md`。

## 不在这里的

- **简单 source**(GitHub Search / arXiv / OpenAlex / npm 等 14 个):1-line curl + .env key,**不写 skill**
- **vendor 已发 OpenAPI 的**:走 `npx openapi-to-skills <yaml-url>` 自动生成,**不手写**
- **已有官方 SKILL.md 的**(Firecrawl):直接装,**不重复发明**
