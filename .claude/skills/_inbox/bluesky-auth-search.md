---
status: draft / unfiltered
source: §1 #18 Bluesky
complexity: 3-step auth + JWT 管理
---

## 为什么需要

`searchPosts` anon 403(2024 后政策)。每次让 host 自己重做 auth flow 很笨。

## 封装什么

```bash
bluesky-search.sh "query" [--limit N]
```

内部:
1. `POST bsky.social/xrpc/com.atproto.server.createSession` `{$BLUESKY_HANDLE, $BLUESKY_APP_PASSWORD}` → 拿 `accessJwt`(2h)
2. `GET .../app.bsky.feed.searchPosts?q=...&limit=...` + `Authorization: Bearer $accessJwt`
3. 提取 `posts[].record.text / .author.handle / .likeCount / .repostCount / .replyCount / .indexedAt`
4. accessJwt 缓存到 `/tmp/bluesky-jwt.<handle>`,2h 内复用

## stdout 形态

```json
{"posts":[{"text":"...","author":"...","likes":5,"reposts":0,"replies":2,"indexedAt":"..."},...]}
```

## 不写的话

模型每次 1️⃣ POST createSession 2️⃣ jq 抽 accessJwt 3️⃣ 拼 GET searchPosts 4️⃣ 抽字段 — **4 步 50+ token 来回**。
