---
status: draft / unfiltered
source: §1 #14 twitterapi.io
complexity: 默认 query 套餐 + free-tier 1/5s sleep + result 提取 + schema 修正(data 是 object 不是 array)
---

## 为什么需要

每次让模型记住 `min_faves:10 -is:retweet lang:en` + sleep 5s + `data.tweets[]` 而非 `data[]` — 浪费 token。

## 封装什么

```bash
tw-search.sh "query keywords" [--mode Top|Latest] [--no-defaults]
```

内部:
1. 默认拼接 `min_faves:10 -is:retweet lang:en`(`--no-defaults` 关闭)
2. URL-encode + GET `/twitter/tweet/advanced_search?query=...&queryType=Top`
3. 内置 sleep 5s 保护(检测连续调用)
4. 提取 `viewCount / isBlueVerified / likeCount / retweetCount / pinnedTweetIds[0] / replies[]`(选填)

## stdout 形态

同 Bluesky,统一字段子集。
