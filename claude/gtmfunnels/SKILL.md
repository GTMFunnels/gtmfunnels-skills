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

### Form

The funnel's lead-capture form: fields, button text, disclaimer, colours, look and redirect. A change is live at once. Scopes: `funnels:read`, `funnels:write`.

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/funnels/{funnelCode}/form` | The form: `{title, submitButtonText, disclaimer, fields[], colors, styles, redirectType, redirectUrl}`; 404 if the funnel has none yet |
| PUT | `/v1/funnels/{funnelCode}/form` | Change it; only the fields sent change, `fields` replaces the whole list. Live immediately. See "Customise the form" |

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
- **Keep the lead form.** Don't remove `<div id="form-section">`, `<div data-block="lead_form"></div>` or the form block. Never write your own `<form>`: change the fields with the form API ("Customise the form").
- **No new placeholders.** Don't add `{{name}}` text that the live page doesn't have. Write the final copy.
- **Images:** use `{{asset:<id>}}` with a numeric id from `GET /v1/assets`, or an asset's `url`. Don't invent image URLs.
- **Size:** the page must stay under 2 MB, and be a complete HTML page.

`409 conflict` means the page changed after you pulled it, or the funnel was regenerated. Pull it again, redo the edit on the new HTML and draft again with the new `revision`.

### Customise the form

The form's fields and look are settings, not page HTML, so changing them needs no draft. When the user wants different questions, button text or form colours:

1. **Read it.** `GET /v1/funnels/{code}/form` returns `{title, submitButtonText, disclaimer, fields: [{name, label, type, placeholder, required, active, options}], colors: {primary, secondary}, styles: {corners, inputBorder, inputFill, buttonStyle, buttonSize, buttonWidth}, redirectType, redirectUrl}`. `404` = the funnel has no form yet (a PUT creates it).
2. **Propose** the fields to the user from the offer: what to ask, which are required, the button text and disclaimer. Wait for their yes.
3. **Save** with `PUT /v1/funnels/{code}/form`, sending only what changes. `fields` is the whole ordered list and replaces the current one; keep each existing field's `name` when you resend it. Tell the user the change is **live immediately** on every page of the funnel, with no draft or publish step.

```bash
curl -s -X PUT -H "X-API-KEY: $GTMFUNNELS_API_KEY" -H "Content-Type: application/json" \
  -d '{"submitButtonText": "Get my quote", "fields": [
        {"name": "name", "label": "Name", "type": "text", "required": true},
        {"name": "email", "label": "Work email", "type": "email", "required": true},
        {"name": "team_size", "label": "Team size", "type": "select", "options": ["1-10", "11-50", "51+"]}]}' \
  "$API/v1/funnels/ABC123/form"
```

   Rules (a break is `422 validation_failed`, with `details[].loc` naming the field, e.g. `["body","fields",2,"options"]`): an active field `email` of type `email` is always required; `type` is one of `text`, `email`, `tel`, `textarea`, `select`, `multiselect`, `radio`, `checkbox`, `date`, `number`, `url`; `select`, `multiselect`, `radio` and `checkbox` need `options` (1-50, each up to 100 characters) and other types must not have them; a new `name` is snake_case (`^[a-z][a-z0-9_]{0,39}$`) and unique; at most 30 fields; label up to 100, placeholder up to 150, title and button text up to 100, disclaimer up to 1000 characters; no `<` or `>` in any of that text (plain text only); colours are `#rrggbb`; `redirectType: "custom"` needs an `https://` `redirectUrl`. Built-in fields you leave out (such as `name` or `mobile`) are switched off, not deleted. Needs `funnels:read` and `funnels:write`.
4. **Style the card, not the form.** On the page, wrap `<div data-block="lead_form"></div>` in your own element (`.signup-card` with `var(--t-surface)`, padding, a heading). The form's own look comes from its `colors` and `styles`.

## Building a Funnel

Only the AI Chat Landing Page blueprint (id 2) can be built this way. Needs keys with `funnels:build`, `blueprints:read`, `pages:read`, `pages:write` (plus `funnels:read` and `funnels:write` for the form, `pages:publish` to publish, `knowledge:write` to upload sources).

1. **Fetch the rules, every build.** `GET /v1/page-contract` returns `{version, markdown}`. Read the markdown and follow it for this build; never rely on a copy from an earlier build.
2. **Read the plan.** `GET /v1/blueprints/2/build-plan` gives the page briefs, required components (`chat_widget`), optional ones (`faq`), chat agent instructions and inputs. A `404` means that blueprint has no build plan.
3. **Ask the user** clarifying questions before writing anything: the company, offer, audience, tone, whether they want the FAQ, and which website URLs or files to draw on. Upload any files with `POST /v1/knowledge-base/upload`; the URLs it returns are the `sourceUrls`.
4. **Build.** Send an `Idempotency-Key` header:

```bash
API="${GTMFUNNELS_API_URL:-https://api.gtmfunnels.ai}"
curl -s -X POST -H "X-API-KEY: $GTMFUNNELS_API_KEY" -H "Idempotency-Key: $(uuidgen)" -H "Content-Type: application/json" \
  -d '{"blueprintId": 2, "name": "Acme Assistant", "websiteUrls": ["https://acme.example"]}' "$API/v1/funnels/build"
```

   Body: `{blueprintId, name, brandCode?, websiteUrls? (max 10), sourceUrls? (max 20)}`. It returns `201 {funnelCode, status: "draft", pages: [{pageId, name, purposeType}], contractVersion}`. `403` = the org is over its usage limit or the key's user isn't a member; `404` = unknown brand; `422` = the blueprint can't be built by Claude Code yet, or a bad body.
5. **Write the page** to a local `page.html` following the contract and the brief. The chat widget is exactly `<div data-block="chat_widget"></div>` (lower-case, double quotes, nothing inside, once); the server turns it into the widget. An optional FAQ uses `.faq-container` (or `#faqAccordion`) > `.faq-item` > `.faq-question` + `.faq-answer`. If the brief wants a lead form on the page, place exactly `<div data-block="lead_form"></div>` where it goes, inside your own card. No scripts, forms or external stylesheets, except Google Fonts (`fonts.googleapis.com`, `fonts.gstatic.com`, https, written plainly). Colours and fonts go through the `:root` `--t-*` block defined as `var(--gtm-*, fallback)`.
6. **Draft.** `GET /v1/funnels/{code}/pages/{pageId}` for the current `revision`, then `PUT /v1/funnels/{code}/pages/{pageId}/draft` with `{html, baseRevision}` as in "Editing a Page". `422 validation_failed` lists problems in `details[]` (`msg`, `line`): fix every one and send the draft again. `409` means the page changed: read it again and retry.
7. **Preview.** Give the user the `previewUrl`. Iterate on their feedback with new drafts.
8. **Customise the form** if the page has one: read it, propose fields, button text and disclaimer to the user, and `PUT` them as in "Customise the form". This saves at once (it is not a draft); say so. It shows on the live site once the funnel is published.
9. **Publish only when the user asks.** Then `POST /v1/funnels/{code}/pages/{pageId}/publish`, and `POST /v1/funnels/{code}/publish` to make the funnel live. Never publish on your own.

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
