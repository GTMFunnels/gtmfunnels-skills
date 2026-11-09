---
name: gtmfunnels
description: Manage GTMFunnels funnels, leads, chatbots, knowledge base and workflows through the Developer API, and edit a funnel's landing pages (pull the HTML, draft, preview, publish, restore). Use when the user mentions GTMFunnels, their funnels or leads, or wants to improve or fix a funnel page.
---

# GTMFunnels API Skill

You have access to the GTMFunnels Developer API. Use it to manage marketing funnels, leads, AI chatbots, knowledge bases and workflows, and to edit a funnel's pages.

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

### Pages

Edit the HTML of a funnel's pages. The live page changes only on publish, and earlier versions stay in the history. Scopes: `pages:read`, `pages:write` (drafts), `pages:publish` (publish and restore).

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/funnels/{funnelCode}/pages` | List the editable pages in funnel order: `{items: [{pageId, name, purposeType, order, url, revision, draft}]}` |
| GET | `/v1/funnels/{funnelCode}/pages/{pageId}` | One page with its live `html`, `revision` and `context` (`brandCode`, `hasFormInsertionPoint`, `serverSideForm`) |
| GET | `/v1/funnels/{funnelCode}/pages/{pageId}/draft` | The open draft, or 404 |
| PUT | `/v1/funnels/{funnelCode}/pages/{pageId}/draft` | Save a draft. Body: `{html, baseRevision}`. Returns `{revision, previewUrl, warnings, updatedAt, html, baseRevision}` |
| DELETE | `/v1/funnels/{funnelCode}/pages/{pageId}/draft` | Discard the draft (204) |
| POST | `/v1/funnels/{funnelCode}/pages/{pageId}/publish` | Make the draft live. Optional `Idempotency-Key` header |
| GET | `/v1/funnels/{funnelCode}/pages/{pageId}/history` | Earlier live versions, newest first: `{items: [{revision, source, publishedBy, publishedAt, replacedAt}]}` |
| POST | `/v1/funnels/{funnelCode}/pages/{pageId}/restore` | Put an earlier version back live. Body: `{revision}` |

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

## Editing a Page

Work on the HTML as a local file, not inside a JSON string.

1. **Pull.** List the pages, then save the one to edit. This writes its HTML to `page.html` and its revision to `page.rev`:

```bash
API="${GTMFUNNELS_API_URL:-https://api.gtmfunnels.ai}"
curl -s -H "X-API-KEY: $GTMFUNNELS_API_KEY" "$API/v1/funnels/ABC123/pages/PAGE_ID" > page.json
python3 -c 'import json; d=json.load(open("page.json")); open("page.html","w").write(d["html"]); open("page.rev","w").write(d["revision"])'
```

2. **Edit** `page.html` with the Edit tool.

3. **Draft.** Send it with the revision you pulled (set `API` again if this is a new shell):

```bash
python3 -c 'import json,sys; print(json.dumps({"html": open("page.html").read(), "baseRevision": sys.argv[1]}))' "$(cat page.rev)" > draft.json
curl -s -X PUT -H "X-API-KEY: $GTMFUNNELS_API_KEY" -H "Content-Type: application/json" \
  --data-binary @draft.json "$API/v1/funnels/ABC123/pages/PAGE_ID/draft"
```

4. **Preview.** Give the user the `previewUrl` from the response and pass on any `warnings`. The link works for 7 days, until the draft is replaced, published or discarded. A lead submitted from a preview is a real lead and runs the funnel's emails and webhooks, so suggest a test address.

5. **Publish** only when the user says so. It changes the live page that visitors see:

```bash
curl -s -X POST -H "X-API-KEY: $GTMFUNNELS_API_KEY" "$API/v1/funnels/ABC123/pages/PAGE_ID/publish"
```

To undo a publish, call `history` and `restore` the previous `revision`.

### What a draft must not do

A draft that breaks a rule is refused with `422 validation_failed`. Every problem is listed in `details` as `{loc, msg, line}`. Fix them all and send the draft again.

- **No new code.** Don't add anything the live page doesn't already have: `<script>` (JSON-LD is allowed), `<iframe>`, `<object>`, `<embed>`, `on*` attributes such as `onclick`, `srcdoc`, `<base>`, `<meta http-equiv="refresh">`, `<link>`, `<noscript>`, or `javascript:`/`data:text/html` URLs. This also covers visible text: a new heading such as "JavaScript: the basics" is refused. Removing, keeping or moving existing code is fine. Make design changes with HTML and CSS (a `<style>` block or `style` attributes).
- **Keep the lead form.** Don't remove `<div id="form-section">` or the form block.
- **No new placeholders.** Don't add `{{name}}` text that the live page doesn't have. Write the final copy.
- **Images:** use `{{asset:<id>}}` with a numeric id from `GET /v1/assets`, or an asset's `url`. Don't invent image URLs.
- **Size:** the page must stay under 2 MB, and be a complete HTML page.

`409 conflict` means the page changed after you pulled it, or the funnel was regenerated. Pull it again, redo the edit on the new HTML and draft again with the new `revision`.

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
