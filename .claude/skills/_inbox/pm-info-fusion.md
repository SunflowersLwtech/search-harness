---
status: draft / unfiltered
source: §1 #11 libraries.io 兜底 PyPI/Maven/Go(三家无 native ranking)
complexity: 3 源融合(native + deps.dev + libraries.io)
---

## 为什么需要

PyPI/Maven/Go native API 都坏(downloads=-1 / Solr 无 license / pkg.go.dev 全 HTML)。要拿一个包的完整信息要查 3 个端点,模型每次重做。

## 封装什么

```bash
pm-info.sh <pm:python|maven|go> <name> [version]
```

内部:
1. native:`pypi.org/pypi/<pkg>/json`(metadata) / `search.maven.org/solrsearch` / 跳过 Go
2. deps.dev:`api.deps.dev/v3/systems/<sys>/packages/<name>/versions/<ver>`(license + advisory)
3. libraries.io:`libraries.io/api/search?platforms=<sys>&q=<name>`(SourceRank ranking)
4. merge 成统一 JSON,advisoryKeys 直接给 OSV ID
