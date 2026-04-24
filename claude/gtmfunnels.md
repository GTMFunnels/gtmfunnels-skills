---
name: gtmfunnels
description: Manage GTMFunnels funnels, leads, chatbots, knowledge base, and workflows via the Developer API.
---

# GTMFunnels API Skill

You have access to the GTMFunnels Developer API. Use it to manage marketing funnels, leads, AI chatbots, knowledge bases, and workflows.

## Configuration

- **Base URL:** Use env var `GTMFUNNELS_API_URL` or default to `https://api.gtmfunnels.ai`
- **Auth:** Include header `X-API-KEY: <GTMFUNNELS_API_KEY>` on every request
- **Rate limit:** 1,000 requests/hour per key

## Available Endpoints

### Funnels

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/funnels?skip=0&limit=20` | List funnels |
| POST | `/v1/funnels/generate` | Generate funnel from blueprint. Body: `{blueprintId, companyUrl, goal?, funnelName?, returnUrl?}`. Set `returnUrl: true` to wait for completion and get the published URL. |
| GET | `/v1/funnels/{funnelCode}` | Get funnel details |
| PUT | `/v1/funnels/{funnelCode}` | Update funnel. Body: `{name?, status?, externalCampaignId?, notes?}` |
| DELETE | `/v1/funnels/{funnelCode}` | Delete funnel |
| POST | `/v1/funnels/{funnelCode}/publish` | Publish funnel |
| POST | `/v1/funnels/{funnelCode}/unpublish` | Unpublish funnel |

### Chat Settings

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/funnels/{funnelCode}/chat-settings` | Get chat widget config |
| PUT | `/v1/funnels/{funnelCode}/chat-settings` | Update chat settings. Body: `{enabled?, colors?, title?, subtitle?, logoUrl?, enableLeadForm?}` |
| GET | `/v1/funnels/{funnelCode}/analytics` | Get funnel analytics (views, leads, conversions) |
| GET | `/v1/funnels/{funnelCode}/embed-code` | Get chat widget embed snippet |

### Leads

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/funnels/{funnelCode}/leads?skip=0&limit=20` | List leads for a funnel |
| POST | `/v1/leads` | Create lead. Body: `{funnelCode, email, name?, phone?, metadata?}` |
| DELETE | `/v1/leads?email=x` | Delete leads by email |
| DELETE | `/v1/funnels/{funnelCode}/leads` | Delete all leads for a funnel |
| GET | `/v1/funnels/{funnelCode}/leads/export` | Export leads as CSV |
| POST | `/v1/funnels/{funnelCode}/leads/score` | AI-score leads. Body: `{limit?}` (default 20) |

### Blueprints

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/blueprints` | List available funnel blueprints |
| GET | `/v1/blueprints/{blueprintId}` | Get blueprint details |

### Knowledge Base

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/knowledge-base` | List KB documents |
| POST | `/v1/knowledge-base/query` | RAG query. Body: `{query, limit?}` |
| POST | `/v1/knowledge-base/upload` | Upload document. Multipart form: `file` field |

### Workflows

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/workflows/executions?funnelCode=X` | List workflow executions |
| GET | `/v1/workflows/executions/{executionId}` | Get execution details |
| GET | `/v1/workflows/executions/{executionId}/status` | Get execution status |
| POST | `/v1/workflows/execute` | Trigger workflow. Body: `{funnelCode, triggerData?}` |

### Account

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/account/usage` | Get token usage, funnel count, lead count |
| POST | `/v1/api-keys` | Create API key. Body: `{name, scopes[]}` |
| GET | `/v1/api-keys` | List API keys |
| DELETE | `/v1/api-keys/{keyId}` | Revoke API key |
| GET | `/v1/api-keys/{keyId}/usage` | Get usage for specific key |

## How to Make Requests

Use the Bash tool with `curl`:

```bash
curl -s -H "X-API-KEY: $GTMFUNNELS_API_KEY" \
  "${GTMFUNNELS_API_URL:-https://api.gtmfunnels.ai}/v1/funnels" | python3 -m json.tool
```

For POST requests:

```bash
curl -s -X POST \
  -H "X-API-KEY: $GTMFUNNELS_API_KEY" \
  -H "Content-Type: application/json" \
  -d '{"blueprintId": 1, "companyUrl": "https://example.com", "returnUrl": true}' \
  "${GTMFUNNELS_API_URL:-https://api.gtmfunnels.ai}/v1/funnels/generate"
```

## Common Workflows

**Generate and publish a funnel:**
1. `GET /v1/blueprints` — pick a blueprint
2. `POST /v1/funnels/generate` with `returnUrl: true` — generates and returns URL
3. Funnel is auto-published when `returnUrl: true`

**Export leads:**
1. `GET /v1/funnels` — find the funnel code
2. `GET /v1/funnels/{funnelCode}/leads/export` — download CSV

**Query knowledge base:**
1. `POST /v1/knowledge-base/query` with `{query: "your question"}`
2. Returns matched documents with relevance scores
