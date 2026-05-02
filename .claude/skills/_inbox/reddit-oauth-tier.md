---
status: draft / unfiltered
source: §1 #5 Reddit
complexity: anon 100/10min 限制 → 高用量切 OAuth client_credentials
---

## 为什么需要

anon Reddit 跑 case_study 类大批量会撞 100/10min 限。OAuth tier 60-100 req/min。flow:client_id+secret → token endpoint → bearer → oauth.reddit.com。

## 封装什么

```bash
reddit-search.sh <subreddit> "query" [--mode oauth|anon]
```

内部:
1. anon 默认:GET `reddit.com/r/<sub>/search.json?q=&t=year`(UA 必填)
2. `--mode oauth`:POST `www.reddit.com/api/v1/access_token` 拿 token → GET `oauth.reddit.com/r/<sub>/search.json`
3. token 缓存 1h

## 何时用 oauth

`case_study/_brief/rollup.md` 类大批量 fan-out。日常单次查 anon 即可。
