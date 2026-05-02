---
status: draft / unfiltered
source: §1 #15 OSV.dev
complexity: vulns 拿到后追每条 references[].url(advisory + commit + patch + writeup)
---

## 为什么需要

`POST /v1/query` 拿 vuln 列表后,每个 vuln 的 `references[]` 数组有 advisory blog / commit fix / writeup。模型每次手动展开。

## 封装什么

```bash
osv-deep.sh <pm:PyPI|npm|...> <name> [version]
```

内部:
1. POST `api.osv.dev/v1/query` body `{package:{name,ecosystem},version}`
2. 对每个 vuln 的 `references[]` 选 `type: ADVISORY` 优先 fetch markdown 摘要
3. 输出统一 schema:每个 vuln 含 `id / summary / cvss / refs[] / fix_commit_url(若有)`
