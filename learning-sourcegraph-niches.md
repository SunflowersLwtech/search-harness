# Sourcegraph 免费匿名层的两条独家原语 — 学习笔记

> **写于** 2026-04-26 · 实测 + 综合多轮分析
>
> **学习目标**:
> 1. 搞清楚 SG 在免费匿名层"独家"的两条能力具体是什么
> 2. 知道**什么时候用、什么时候别用、踩什么坑**
> 3. 能上手跑 demo,验证手感
>
> **前置阅读**(本仓内):
> - [`adapter-vs-skill.md`](./adapter-vs-skill.md) §2 — 23 个 adapter 的 scope
> - [`domain-knowledge.md`](./domain-knowledge.md) §5.1 #14 — GitHub 检索能力实测的建设性结论
> - [`architecture.md`](./architecture.md) §3.9 — adapter vs skill 决策树

---

## TL;DR(回头先看这一段)

Sourcegraph 公网免费匿名层(无 auth、无登录)只有 **2 条原语**真正值得为它写 skill:

1. **`type:symbol`** — 跨所有 OSS 找一个名字的**定义**,带 `kind` / `containerName` / `file:line` 结构化字段
2. **`repo:has.file(path:X content:Y) select:repo`** — 找所有"有路径=X 且文件内容含 Y"的 **repo**(不是 file)

其他能力都不值得依赖(structural search 在免费层 timeout 残废、stars 过滤墙、关键词 ranking 烂)。

**经济价值**:这两条都把"agent 拿到结果后还要 LLM 二次判断"这一轮省掉了——具体省 token 数量见 §6。

**端点**:`https://sourcegraph.com/.api/search/stream`(SSE),不要用 GraphQL(anon 硬封顶 30)。

---

## 1. 背景:为什么这两条特别?

任何代码搜索引擎本质做两件事之一:

| 类型 | 索引内容 | 返回什么 |
|---|---|---|
| **Content match**(GitHub Code Search、grep.app、searchcode) | 文件全文倒排 | "包含字符串 X 的文件" |
| **Symbol-aware**(SG `type:symbol`) | 通过 SCIP indexer 解析后的 AST 节点 | "X 这个符号被定义/引用的位置,带 kind 元数据" |
| **Repo-topology**(SG `repo:has.*`) | repo 元数据 + 文件路径×内容 | "满足复合 file/content 条件的 **repo 列表**" |

GitHub Code Search 始终是 Content match 一类。Sourcegraph 是少数公开提供 Symbol-aware 和 Repo-topology 的服务,而且免费匿名能用。

**这两类做的是 Content match 做不到的"结构化"事**——所以才独家。

---

## 2. 原语 1:`type:symbol` 跨仓符号定义

### 2.1 它返回什么

实测一例(查 `Tokenizer` 在 Rust 里的定义):

```bash
curl -sN "https://sourcegraph.com/.api/search/stream?q=$(printf %s 'type:symbol Tokenizer lang:rust' | jq -sRr @uri)"
```

事件流里关键的 `event: matches` 包含的字段(每个匹配):

```json
{
  "type": "symbol",
  "repository": "github.com/awslabs/duvet",
  "path": "tokenizer.rs",
  "branches": ["main"],
  "symbols": [
    {
      "name": "Tokenizer",
      "kind": "STRUCT",
      "containerName": "duvet::markdown::tokenizer",
      "url": "/github.com/awslabs/duvet/-/blob/...#L38:13-38:22",
      "line": 38
    }
  ]
}
```

**关键信号**:
- `kind`:`STRUCT` / `METHOD` / `FUNCTION` / `INTERFACE` / `CLASS` / `TYPE` / `FIELD` / `CONSTANT` 等
- `containerName`:这个符号在哪个 module / class / namespace 里
- `line` + `url`:精确到行号的跳转链

agent 拿到这些**不需要再 LLM 判断"这是不是定义"**——已经标注好了。

### 2.2 GitHub 同题给什么(对比)

```bash
gh search code "Tokenizer" --language Rust
```

返回:

- `crates/foo/src/lib.rs` 第 12 行 — `use crate::tokenizer::Tokenizer;`(import 行)
- `crates/foo/src/main.rs` 第 87 行 — `let t = Tokenizer::new(...);`(调用方)
- `crates/foo/Cargo.toml` 第 5 行 — `tokenizer = "0.3"`(依赖声明)
- `crates/foo/tests/tok.rs` 第 23 行 — `// Tokenizer should handle UTF-8`(注释)
- `crates/foo/src/tokenizer.rs` 第 38 行 — `pub struct Tokenizer { ... }`(**真正的定义**)

agent 拿到这堆,要么二次 LLM 判断"这条是定义吗"(每条 ~200 token,5 条 1000 token,加上模型推理),要么瞎挑——实战通常瞎挑挑错。

### 2.3 5 个真实 coding-agent 场景

| 场景 | query 模板 | agent 拿到能干嘛 |
|---|---|---|
| **API archaeology**——"我看到 stack trace 里 `Tokenizer::new`,这是哪个库?" | `type:symbol Tokenizer kind:struct lang:rust` | 一次列出所有 candidate 库;`containerName` 区分 `tantivy::tokenizer::Tokenizer` vs `huggingface::tokenizers::Tokenizer` |
| **Idiom 调研**——"我要写自己的 BloomFilter,看 5 个真实开源实现作参考" | `type:symbol BloomFilter kind:struct lang:rust` | 各 crate 实现一次性铺出来,直接拉源码对比 |
| **命名冲突 triage**——"两个 crate 都 export `Result`,出 trait conflict" | `type:symbol type Result kind:type lang:rust` | 看每个 crate 的 Result 各是什么 alias |
| **迁移辅助**——"我从 serde_json 迁到 simd-json,前者的 `Value::as_str()` 在后者怎么实现的?" | `type:symbol fn as_str lang:rust repo:^github\.com/simd-lite/.*$` | 直接跳到对应方法定义,看签名差异 |
| **学习"X 框架的标准用法"** | `type:symbol trait Handler lang:rust repo:^github\.com/tokio-rs/axum$` | 跨多个版本 / feature flag 的实现一次性可见 |

### 2.4 Footgun(写 skill 时必须告诉 agent)

| 坑 | 现象 | 正确写法 |
|---|---|---|
| 加 `fn` 前缀 | `type:symbol fn parse_yaml` 返回 0 | 直接 `type:symbol parse_yaml`,kind 通过 `kind:function` 表达 |
| `kind:` 拼写 | 大小写敏感,`kind:Struct` 可能不命中 | 用全大写 `kind:STRUCT` 或全小写 `kind:struct`(实测两者都成,但保持一致) |
| 多语言一次查 | `type:symbol parse_yaml` 不带 `lang:` 时混杂 Ruby/Python/JS,signal 稀释 | 加 `lang:<X>` 收窄 |
| SCIP 覆盖偏科 | Java/TS/Python/Rust/Go/C++ 较成熟,Elixir/OCaml/Zig 稀疏 | 写 skill 时**显式说明覆盖语言**,小语种 fallback GH grep |

---

## 3. 原语 2:`repo:has.file(path:X content:Y) select:repo` 反向依赖审计

### 3.1 它返回什么

查"所有 Cargo.toml 里依赖 tantivy 的 repo":

```bash
curl -sN "https://sourcegraph.com/.api/search/stream?q=$(printf %s 'repo:has.file(path:Cargo\.toml content:tantivy) select:repo' | jq -sRr @uri)"
```

返回 35 个 repo 名单(实测),每条只有 repo URL + description + stars(如果 SG 有缓存)。**不是文件命中,是 repo 命中**——`select:repo` 是关键。

### 3.2 GitHub 同题做不了的原因

| 工具 | 能做啥 | 缺啥 |
|---|---|---|
| `gh search code "tantivy" --filename Cargo.toml` | 单条件搜内容 + filename 过滤 | 返回 **file 命中**——同 repo 几十个文件占满 top-N;agent 要自己 dedupe |
| `gh search code` 多条件 | 不支持"路径=A **且** 内容=B"复合 | path 和 content 是同一个搜索字段,无法精确组合 |
| GitHub Dependents tab | repo X 被谁依赖 | UI-only **没 API**;只覆盖在 GitHub 上发布过 release 的包;精度差 |
| **libraries.io / deps.dev** | "X 依赖 Y" 但只看包管理器声明字段 | 不看文件里具体 content / 锁定版本 / feature flag;**只能查 manifest 类文件** |

### 3.3 8 个真实 coding-agent 场景

| 场景 | query | 价值 |
|---|---|---|
| **CVE 影响审计**——"openssl-sys 0.9.x 有 CVE,Rust 生态谁中招?" | `repo:has.file(path:Cargo\.lock content:openssl-sys.0.9) select:repo` | 直接给受影响 repo;libraries.io 只看声明依赖,看不到锁定版本 |
| **库采用度调研**——"我开发的 crate 真实用户有哪些?" | `repo:has.file(path:Cargo\.toml content:my-crate) select:repo` | 比 crates.io 的 dependents tab 更广(含 fork、私有投影到 GH 的) |
| **Pattern mining**——"用 Next.js 14 server actions 的真实配置长什么样?" | `repo:has.file(path:next\.config\.js content:experimental.serverActions) select:repo` | 看真实生产项目怎么开 experimental 特性 |
| **Migration 评估**——"我要重命名公共 API,谁会爆?" | `repo:has.file(path:.+ content:my_old_function_name) select:repo` | 给 downstream 影响清单 |
| **CI/CD 模式调研**——"用 Cloudflare Workers 的 GH Actions 怎么写?" | `repo:has.file(path:\.github/workflows/.+\.ya?ml content:wrangler.deploy) select:repo` | path + content 复合,GH 做不到 |
| **Dotfiles archaeology**——"用 NixOS + Home Manager 配 nvim 的人怎么组织?" | `repo:has.file(path:flake\.nix content:home-manager) select:repo` | 找特定栈的配置模板 |
| **License 审计**——"哪些 repo 同时有 MIT 和 Apache 双授权?" | `repo:has.file(path:LICENSE-MIT) repo:has.file(path:LICENSE-APACHE) select:repo` | 多个 has.file 子句叠加 |
| **Security templating**——"业界 .env.example 怎么写 OPENAI_API_KEY?" | `repo:has.file(path:\.env\.example content:OPENAI_API_KEY) select:repo` | 调研标准模板 |

### 3.4 Footgun

| 坑 | 现象 | 处理 |
|---|---|---|
| **shard timeout** | 大查询(如 `path:.+ content:<common keyword>`)20s+ 后只返回部分仓 | **必须解析响应里的 `progress.skipped[reason=shard-timeout]`**,告诉 host"这次没查全";否则 agent 把"漏的"当"没有的" |
| `repo:has.meta(stars >= N)` 不可用 | 免费匿名层返回 0 | 只能拿到 repo 列表后用 GitHub API 拉 stars;skill 文档要明说 |
| 路径正则要转义 | `path:Cargo.toml` 把 `.` 当通配,误命中 `Cargolytoml` | 用 `path:Cargo\.toml` |
| content 大小写敏感 | `content:tantivy` vs `content:Tantivy` 实测命中量不同 | 加 `case:no` 或用 `(?i)` 正则 |
| 复合子句 ≥3 个会显著变慢 | `repo:has.file(...)` × 4 可能 30s+ 或 timeout | 拆成两次,本地求交 |

---

## 4. 端点 / 协议(用之前先搞清)

### 4.1 必须用 stream API,不要用 GraphQL

| 维度 | Stream API | GraphQL `search` |
|---|---|---|
| URL | `https://sourcegraph.com/.api/search/stream` | `https://sourcegraph.com/.api/graphql` |
| 协议 | SSE(`event: matches\ndata: {...}\n\n`) | JSON-RPC over HTTPS |
| 匿名 result cap | **`count:all` 真有效**(实测 1.5M matches/19.8k repos) | **硬封顶 30**,无 cursor 可分页 |
| 包含 progress 元数据 | ✅(`event: progress` 里有 `skipped[]`) | ❌ |
| 解析复杂度 | 中(SSE,要按 event type 分发) | 低(标准 JSON) |

**结论**:agent harness 的 SG adapter 必须基于 stream API,不要被 GraphQL 简单性诱惑。

### 4.2 Stream 事件类型

```
event: matches      ← 实际命中,你要的
data: {"type":"symbol","repository":"...","symbols":[...]}

event: progress     ← 关键!里面有 skipped[]
data: {"matchCount":1234,"skipped":[{"reason":"shard-timeout","title":"...big repo missing..."}]}

event: filters      ← 提供给 UI 的 facet,可忽略
data: [...]

event: done         ← 流结束
data: {}
```

agent 必须收集所有 `matches` + 检查最后的 `progress.skipped`,才能给 host 报"是否查全"。

---

## 5. Skill 文件应该怎么写(参考模板)

```markdown
# When to use Sourcegraph (free anonymous tier only)

You **MUST** use the Sourcegraph stream API
(`https://sourcegraph.com/.api/search/stream`) — never the GraphQL endpoint
(it caps anonymous results at 30 with no usable cursor).

## Trigger 1 — Cross-repo SYMBOL DEFINITION lookup

If user asks "where is X defined" / "show me implementations of X" /
"find all definitions of <name>" — across multiple repos or all of OSS:

  q = `type:symbol <name> [kind:STRUCT|METHOD|FUNCTION|TYPE] [lang:<X>] [repo:^...$]`

DO NOT prefix with `fn`/`def`/`class` — that breaks the symbol parser.
Use `kind:` filters instead.

After parsing matches, present `name + kind + containerName + file:line + url`
to the user. Do NOT additionally guess what's a definition; SCIP already classified.

## Trigger 2 — Reverse dependency / topology query

If user asks "which repos depend on X" / "which repos use file pattern Y with
content Z" / "find all CI configs that ..." / "find all repos that contain
both file A and file B":

  q = `repo:has.file(path:<regex> content:<keyword>) [repo:has.file(...)]* select:repo`

Always pass `select:repo` to fold to repo-level results (otherwise you get
file-level which causes deduplication overhead).

## MUSTs and DO-NOTs

MUST:
- Always check `progress.skipped[]` in the final progress event.
  If reason is `shard-timeout`, warn the host: results may be incomplete.
- Always escape regex chars in `path:` (e.g., `Cargo\.toml` not `Cargo.toml`).
- Pass `case:no` if case-insensitive matching is desired.

DO NOT:
- Use `patterntype:structural` on the public anonymous tier — it silently
  shard-times-out at scale (~60s, 0 results returned).
- Use `repo:has.meta(stars >= N)` — it returns 0 on free anon (enterprise feature).
  For star-filtered repo discovery, fall back to the GitHub adapter.
- Use Sourcegraph for general code search ("find files containing X") —
  GitHub Code Search via PAT is faster (1-3s vs 7-19s) and more reliable.
- Promise non-GitHub platform coverage. Codeberg / SourceHut /
  GitLab self-hosted forges are not indexed.
```

约 50 行,包含全部触发 + 4 个硬约束 + 4 个反模式。Progressive 加载,只在相关 query 触发时进 host context。

---

## 6. 经济性论证(为什么 30-50 行 markdown 划算)

设一次"查 `parse_yaml` 在哪里定义的"任务在 agent 里的 token 成本(粗估,Claude Sonnet 4.6 单价):

### 不用 SG 的路径
1. agent 调 `gh search code "def parse_yaml" --language Python` → 返回 30 条 file 命中,大约 5K tokens(JSON)
2. agent 用 LLM 判断每条"是定义还是引用",30 条 × 200 tokens prompt = 6K tokens 输入
3. LLM 输出筛选结果 ~500 tokens
4. **总:~12K tokens + 2 轮 round-trip**

### 用 SG 的路径
1. agent 调 SG stream `type:symbol parse_yaml lang:python` → 返回 20 条已分类的 symbol,~2K tokens
2. agent 直接挑 `kind:METHOD` 的几条返回给 user,无需 LLM 二次判断
3. **总:~2K tokens + 1 轮 round-trip**

**差**:大约 10K token + 1 轮延迟 / 任务。

如果一周用到 10 次这类 query,就是 100K token / 周。skill 文档约 1.5K token(progressive 加载,只在 trigger 命中时进 context)。**净收益约 50-60x**。

反向依赖审计同理,而且因为 GH 没有等价能力(只能逐 file 自己 dedupe),省的 token 更多。

---

## 7. 何时**不**用 SG(也很重要)

| 任务 | 用啥 | 理由 |
|---|---|---|
| 通用 keyword 代码搜索("含某字符串的文件") | **GitHub Code Search + PAT** | 1-3s vs SG 7-19s,且 GH 有 stars/license/size 过滤 |
| Repo discovery 按"概念/意图"找 repo | **GitHub Search API + skill 教 query rewriting + HN/Reddit cross-ref** | SG 关键词 ranking 烂(实测漏 tantivy/meilisearch) |
| 找带 stars 阈值的 repo | **GitHub adapter** | SG 免费匿名 stars 过滤完全无效 |
| 重度 structural pattern(`Result<:[T], :[E]>` 这种) | **本地 ast-grep / semgrep** | SG 免费层 structural search shard timeout 不可用 |
| 深度 repo Q&A | **DeepWiki MCP** | SG 没这能力 |
| 非 GitHub 平台(Codeberg / SourceHut) | **直接调对应平台 API** | SG 不索引 |

---

## 8. 上手 demo(回头自己跑一遍验证手感)

### 8.1 Symbol search 实跑

```bash
# 跨仓找 BloomFilter 的 Rust 定义
Q='type:symbol BloomFilter kind:struct lang:rust'
curl -sN "https://sourcegraph.com/.api/search/stream?q=$(printf %s "$Q" | jq -sRr @uri)" \
  | grep -E '^(event|data):' | head -50
```

预期看到 `event: matches` + `data: {"type":"symbol","symbols":[...]}` 多条,以及最后 `event: progress` 包含 matchCount。

### 8.2 反向依赖审计实跑

```bash
# 找所有 Cargo.toml 里依赖 tantivy 的 repo
Q='repo:has.file(path:Cargo\.toml content:tantivy) select:repo'
curl -sN "https://sourcegraph.com/.api/search/stream?q=$(printf %s "$Q" | jq -sRr @uri)" \
  | grep -E '^data:' | head -30
```

预期返回 ~30+ repo 列表。注意检查最后的 `progress` 事件里有没有 `skipped`——大概率有,会包含若干被 shard timeout 的大库。

### 8.3 复合反向依赖(进阶)

```bash
# 找所有有 LICENSE-MIT 又有 LICENSE-APACHE 的 repo(双授权)
Q='repo:has.file(path:LICENSE-MIT) repo:has.file(path:LICENSE-APACHE) select:repo'
curl -sN "https://sourcegraph.com/.api/search/stream?q=$(printf %s "$Q" | jq -sRr @uri)" \
  | grep -E '^data:' | wc -l
```

预期一个数字——双授权在生态里其实很常见,看看到底多少。

### 8.4 footgun 验证

跑这一条**应该返回 0**(`fn` 前缀坏掉):

```bash
Q='type:symbol fn parse_yaml lang:python'
curl -sN "https://sourcegraph.com/.api/search/stream?q=$(printf %s "$Q" | jq -sRr @uri)" \
  | grep matchCount
```

vs 这一条(去掉 `fn` 前缀)应该返回多条:

```bash
Q='type:symbol parse_yaml lang:python'
curl -sN "https://sourcegraph.com/.api/search/stream?q=$(printf %s "$Q" | jq -sRr @uri)" \
  | grep matchCount
```

亲手验证 footgun 是真实的——印象会比看文档深得多。

---

## 9. 延伸学习(如果想再深一层)

| 话题 | 资源 |
|---|---|
| SCIP 协议本身(SG 的符号搜索之所以能工作的协议) | https://github.com/sourcegraph/scip |
| 各语言的 SCIP indexer | scip-java / scip-typescript / scip-python / scip-clang(Apache-2.0,见 [domain-knowledge.md §5.1 #14]) |
| zoekt(SG 用的 trigram 搜索引擎) | https://github.com/sourcegraph/zoekt |
| Sourcegraph 查询语法完整文档 | https://docs.sourcegraph.com/code_search/reference/queries |
| `repo:has.*` 全集 | https://docs.sourcegraph.com/code_search/reference/queries#repo-has-predicate |
| Stream API 协议 | https://docs.sourcegraph.com/api/stream_api |

---

## 10. 一句话回顾

> **SG 免费匿名层,只为这两条原语而留:`type:symbol`(替你做了符号 vs 引用的判断)和 `repo:has.file(path:X content:Y) select:repo`(独家的 repo 拓扑过滤)。其余 95% 的代码搜索需求走 GitHub。这两条用 stream API、解 `progress.skipped`、躲 4 个 footgun,就完整覆盖。**
