# 待筛选 skills — 仅官方 / 社区已存在且验证过的

> 本文档**只列外部已存在 + 公开维护**的 skill 资源。**不放自写内容**。
> 用作未来选装时的索引。每条带 source link + 最近一次实测/验证状态。

## 已实测可用

| Skill | 来源 | 实测状态 | 装法 |
|---|---|---|---|
| **Context7 CLI**(`ctx7`) | [upstash/context7](https://github.com/upstash/context7) | ✓ 已装(`/opt/homebrew/bin/ctx7` v0.4.0)| `npm install -g ctx7 && npx ctx7 setup` |
| **Playwright CLI** | Microsoft 官方 npm `playwright` | ✓ 已装(`/opt/anaconda3/bin/playwright` v1.58.0)| `npm install -g playwright && npx playwright install chromium` |
| **yt-dlp** | [yt-dlp/yt-dlp](https://github.com/yt-dlp/yt-dlp) | ✓ 已装(`/opt/homebrew/bin/yt-dlp` 2026.3.17)| `brew install yt-dlp` |
| **DeepWiki MCP** | Cognition / Devin | ✓ 已装(`https://mcp.deepwiki.com/mcp` HTTP)| `claude mcp add --transport http deepwiki https://mcp.deepwiki.com/mcp` |

## 已扫官方 SKILL.md(未装,等用例触发)

| Skill | 官方 SKILL.md URL | 用例 |
|---|---|---|
| **Firecrawl agent skill** | [firecrawl.dev/agent-onboarding/SKILL.md](https://www.firecrawl.dev/agent-onboarding/SKILL.md) | SPA / login wall / JS-heavy 站抓取 |
| **callstackincubator/github**(gh CLI workflow)| [agent-skills/skills/github/SKILL.md](https://github.com/callstackincubator/agent-skills/tree/main/skills/github) | GitHub PR / squash / rebase / stacked-PR(write workflow,**与 search-agent read-only 用例错位**)|
| **huggingface/skills**(13 个)| [huggingface/skills](https://github.com/huggingface/skills) — `hf-cli` / `huggingface-datasets` / `huggingface-papers` / `huggingface-paper-publisher` / `huggingface-llm-trainer` / `huggingface-vision-trainer` / `huggingface-gradio` / `huggingface-best` / `huggingface-community-evals` / `huggingface-local-models` / `huggingface-tool-builder` / `huggingface-trackio` / `transformers-js` | HF Hub 管理(download/upload/auth/jobs/buckets,**与 search-agent 检索用例错位**)|

## OpenAPI → SKILL.md 自动生成工具

| 工具 | 来源 | 实测状态 |
|---|---|---|
| **openapi-to-skills** | [neutree-ai/openapi-to-skills](https://github.com/neutree-ai/openapi-to-skills) | ✓ 实测 v0.3.0(petstore → 19 ops + 171 文件 / OpenAI Stainless YAML → 241 ops + 1369 文件) |
| **Speakeasy Agent Skills** | [speakeasy.com/blog/release-agent-skills](https://www.speakeasy.com/blog/release-agent-skills) | 未试(SaaS) |
| **vblagoje/openapi-llm** | [vblagoje/openapi-llm](https://github.com/vblagoje/openapi-llm) | 未试 |

## Vendor canonical OpenAPI 入口(用前先 fetch)

| Vendor | Canonical URL |
|---|---|
| **OpenAI** | `https://app.stainless.com/api/spec/documented/openai/openapi.documented.yml`(2.6MB / 241 ops) |
| **Stripe** | `https://github.com/stripe/openapi` |
| **Twilio** | `https://github.com/twilio/twilio-oai` |
| **GitHub** | `https://github.com/github/rest-api-description` |
| **DigitalOcean** | `https://github.com/digitalocean/openapi` |
| **Slack** | `https://github.com/slackapi/slack-api-specs` |
| **Google Maps** | `https://github.com/googlemaps/openapi-specification`(Google 唯一自家发的)|
| **Google 其他 299 个**(社区 mirror) | `https://github.com/APIs-guru/openapi-directory/tree/main/APIs/googleapis.com/<service>/v1/openapi.yaml` |
| **Anthropic** | ✗ 不公开发 |
| **Notion** | ✗ 不公开发 |

## 集市 / index 入口(查新候选用)

- [VoltAgent/awesome-agent-skills](https://github.com/VoltAgent/awesome-agent-skills)(1000+ skills)
- [sickn33/antigravity-awesome-skills](https://github.com/sickn33/antigravity-awesome-skills)(1400+ skills)
- [anthropics/skills](https://github.com/anthropics/skills)(17 通用,无 source-specific)
- [officialskills.sh](https://officialskills.sh/)
- [APIs-guru/openapi-directory](https://github.com/APIs-guru/openapi-directory)(2000+ vendor OpenAPI mirror)

## 不放在这里的

- 任何手写 stub / 自创 SKILL.md(本文档纪律)
- 已 drop 的(grok-cli wrapper / Sentry MCP / GitHub MCP / Playwright MCP / Sourcegraph MCP — 见 `domain-knowledge.md` §5.1 #23 A)
