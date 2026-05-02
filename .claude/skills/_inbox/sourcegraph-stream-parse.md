---
status: draft / unfiltered
source: §1 #19 Sourcegraph stream
complexity: SSE event-stream 解析(event: matches 不是 JSON,需要按行 split)
---

## 为什么需要

Sourcegraph stream API 返回 SSE,模型每次要解析 `event: matches\ndata: [...]` 形态很丑。

## 封装什么

```bash
sg-stream.sh "context:global+content:..." [--limit N]
```

内部:
1. `curl -H 'Accept: text/event-stream' .../search/stream?q=...`
2. awk/jq 抽 `event: matches` 块的 data array
3. flatten 成 JSON array:`[{repo, repoStars, commit, path, lineNumber, line}]`

## stdout 形态

```json
[{"repo":"github.com/...","repoStars":3617,"commit":"...","path":"...","lineNumber":24,"line":"..."}, ...]
```
