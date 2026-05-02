---
status: draft / unfiltered
source: §1 #8 OpenReview
complexity: 二段抓取(notes/search → notes?forum=<id> 拿 review thread)
---

## 为什么需要

`notes/search` 只给论文 metadata + forum id;真正的 review/decision/反驳在 `notes?forum=<id>` 第二个 endpoint。模型每次记不住二段。

## 封装什么

```bash
or-thread.sh "title keywords" [--decision-only]
```

内部:
1. GET `api2.openreview.net/notes/search?term=<query>` → 拿 `notes[0].forum`(top hit)
2. GET `api2.openreview.net/notes?forum=<id>` → 整 thread
3. 抽 `decision / replies[].content / replies[].signatures`(reviewer ID)

## stdout 形态

```json
{"paper":"...","decision":"Accept","reviews":[{"reviewer":"...","content":"...","rating":7},...]}
```
