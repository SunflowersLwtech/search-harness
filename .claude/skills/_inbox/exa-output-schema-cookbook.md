---
status: draft / unfiltered
source: §1 #20 Exa
complexity: outputSchema 是 Exa 杀手锏,但模板每次重写
---

## 为什么需要

`outputSchema` JSON 是结构化提取的关键。常见模板(extract company/people/product/comparison)模型每次重发明。

## 封装什么

```bash
exa-extract.sh "query" <template:companies|people|products|pricing|libraries> [--num N]
```

内部:
1. 5 个固定 outputSchema 模板(extract list of {name, description, url})
2. POST `api.exa.ai/search` body `{query, type:"auto", outputSchema:<template>, contents:{highlights:true}}`
3. 输出 `output.content` + grounding citations

## 模板示例(libraries)

```json
{
  "type":"object",
  "required":["libraries"],
  "properties":{
    "libraries":{"type":"array","items":{
      "type":"object","required":["name","strength"],
      "properties":{"name":{"type":"string"},"strength":{"type":"string"}}
    }}
  }
}
```

## 实测验证

`runs/exa/2026-05-01T103612Z_outputSchema-rust-async.json` Tokio/async-std/smol/glommio 4 条 + 8 字段 grounding citations,$0.007 / call。
