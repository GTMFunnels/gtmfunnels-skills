---
name: gtmfunnels
description: Manage GTMFunnels marketing funnels, leads, AI chatbots, knowledge base and workflows via the Developer API, and edit a funnel's pages (draft, preview, publish, restore).
metadata.openclaw: {"emoji":"🚀","requires":{"env":["GTMFUNNELS_API_KEY"]}}
---

# GTMFunnels API

You can manage GTMFunnels marketing funnels using the Developer API. Use the `exec` tool to make API calls with `curl`.

## Authentication

Every request must include the `X-API-KEY` header:

```bash
curl -s -H "X-API-KEY: $GTMFUNNELS_API_KEY" "${GTMFUNNELS_API_URL:-https://api.gtmfunnels.ai}/v1/funnels"
```

## Endpoints

### Funnels

**List funnels:**
```bash
curl -s -H "X-API-KEY: $GTMFUNNELS_API_KEY" "${GTMFUNNELS_API_URL:-https://api.gtmfunnels.ai}/v1/funnels?skip=0&limit=20"
```

**Generate a funnel from a blueprint:**
```bash
curl -s -X POST -H "X-API-KEY: $GTMFUNNELS_API_KEY" -H "Content-Type: application/json" \
  -d '{"blueprintId": 1, "companyUrl": "https://example.com", "goal": "Generate leads", "returnUrl": true}' \
  "${GTMFUNNELS_API_URL:-https://api.gtmfunnels.ai}/v1/funnels/generate"
```
Set `returnUrl: true` to wait for generation to complete and get the published URL back.

**Get funnel details:**
```bash
curl -s -H "X-API-KEY: $GTMFUNNELS_API_KEY" "${GTMFUNNELS_API_URL:-https://api.gtmfunnels.ai}/v1/funnels/{funnelCode}"
```

**Update funnel:** `PUT /v1/funnels/{funnelCode}` — Body: `{name?, status?, externalCampaignId?, notes?}`

**Delete funnel:** `DELETE /v1/funnels/{funnelCode}`

**Publish/Unpublish:** `POST /v1/funnels/{funnelCode}/publish` | `POST /v1/funnels/{funnelCode}/unpublish`

### Pages

Scopes: `pages:read`, `pages:write`, `pages:publish`. The live page changes only on publish.

- `GET /v1/funnels/{funnelCode}/pages`: the editable pages, with `pageId` and `revision`
- `GET /v1/funnels/{funnelCode}/pages/{pageId}`: one page with its live `html` and `revision`
- `PUT /v1/funnels/{funnelCode}/pages/{pageId}/draft`: body `{html, baseRevision}`; returns a `previewUrl` (valid 7 days) and `warnings`
- `GET` / `DELETE /v1/funnels/{funnelCode}/pages/{pageId}/draft`: read or discard the draft
- `POST /v1/funnels/{funnelCode}/pages/{pageId}/publish`: make the draft live. Only when the user asks.
- `GET /v1/funnels/{funnelCode}/pages/{pageId}/history`, `POST .../restore` with `{revision}`: go back to an earlier version

A draft is refused (`422`, with a `line` for each problem) if it adds code the live page doesn't have (`<script>` except JSON-LD, `<iframe>`, `on*` attributes, `<link>`, `javascript:` URLs and similar), removes the lead form (`<div id="form-section">`), adds new `{{name}}` placeholders, or uses an asset id that isn't a number from `GET /v1/assets`. `409` means the page changed: pull it again and redo the edit. Leads submitted from a preview are real.

### Leads

**List leads:**
```bash
curl -s -H "X-API-KEY: $GTMFUNNELS_API_KEY" "${GTMFUNNELS_API_URL:-https://api.gtmfunnels.ai}/v1/funnels/{funnelCode}/leads?skip=0&limit=20"
```

**Create lead:** `POST /v1/leads` — Body: `{funnelCode, email, name?, phone?, metadata?}`

**Export as CSV:** `GET /v1/funnels/{funnelCode}/leads/export`

**Score leads with AI:** `POST /v1/funnels/{funnelCode}/leads/score` — Body: `{limit?: 20}`

**Delete leads by email:** `DELETE /v1/leads?email=user@example.com`

**Delete all leads for funnel:** `DELETE /v1/funnels/{funnelCode}/leads`

### Chat Settings

**Get chat settings:** `GET /v1/funnels/{funnelCode}/chat-settings`

**Update chat settings:** `PUT /v1/funnels/{funnelCode}/chat-settings` — Body: `{enabled?, colors?, title?, subtitle?, logoUrl?, enableLeadForm?}`

**Get embed code:** `GET /v1/funnels/{funnelCode}/embed-code`

**Get analytics:** `GET /v1/funnels/{funnelCode}/analytics`

### Blueprints

**List blueprints:** `GET /v1/blueprints` — Returns available funnel templates (squeeze, leadmagnet, webinar, quiz, sales, etc.)

**Get blueprint:** `GET /v1/blueprints/{blueprintId}`

### Knowledge Base

**List documents:** `GET /v1/knowledge-base`

**Query with RAG:**
```bash
curl -s -X POST -H "X-API-KEY: $GTMFUNNELS_API_KEY" -H "Content-Type: application/json" \
  -d '{"query": "What is our pricing?", "limit": 5}' \
  "${GTMFUNNELS_API_URL:-https://api.gtmfunnels.ai}/v1/knowledge-base/query"
```

**Upload document:** `POST /v1/knowledge-base/upload` — Multipart form with `file` field

### Workflows

**List executions:** `GET /v1/workflows/executions?funnelCode={funnelCode}`

**Trigger workflow:** `POST /v1/workflows/execute` — Body: `{funnelCode, triggerData?}`

**Get execution status:** `GET /v1/workflows/executions/{executionId}/status`

### Account

**Get usage:** `GET /v1/account/usage` — Returns token usage, funnel count, lead count

## Common Workflows

1. **Generate a funnel:** List blueprints → pick one → call generate with `returnUrl: true` → get the published URL
2. **Export leads:** List funnels → find the funnel code → call export endpoint → returns CSV
3. **Score leads:** Call score endpoint → returns AI-scored leads ranked hot/warm/cold
4. **Query knowledge:** Post a natural language query → get matched documents with relevance scores
