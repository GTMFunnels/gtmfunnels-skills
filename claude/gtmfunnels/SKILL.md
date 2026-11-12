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

Edit the HTML of a funnel's pages. The live page changes only on publish, and earlier versions stay in the history. Scopes: `pages:read`, `pages:write` (drafts, adding a step), `pages:publish` (publish and restore). The list, get and draft page routes take `?version=N` to work on the newest unpublished version (page publish does not: publish the version).

| Method | Path | Description |
|--------|------|-------------|
| GET | `/v1/funnels/{funnelCode}/pages` | List the editable pages in funnel order: `{items: [{pageId, name, kind, purposeType, order, url, revision, draft}]}`. `kind` is `page` or `email`; an email item also has `subject` (emails are listed only for funnels built through the API) |
| GET | `/v1/funnels/{funnelCode}/pages/{pageId}` | One page with its live `html`, `revision` and `context` (`brandCode`, `hasFormInsertionPoint`, `serverSideForm`) |
| GET | `/v1/funnels/{funnelCode}/pages/{pageId}/draft` | The open draft, or 404 |
| PUT | `/v1/funnels/{funnelCode}/pages/{pageId}/draft` | Save a draft. Body: `{html, baseRevision, subject?}`; `subject` is for an email only (one line, 1-200 characters, no `{{`; left out keeps the current one; otherwise 422 on `subject`). Returns `{revision, previewUrl, warnings, updatedAt, html, baseRevision}` |
| DELETE | `/v1/funnels/{funnelCode}/pages/{pageId}/draft` | Discard the draft (204) |
| POST | `/v1/funnels/{funnelCode}/pages/{pageId}/publish` | Make the draft live. Optional `Idempotency-Key` header |
| GET | `/v1/funnels/{funnelCode}/pages/{pageId}/history` | Earlier live versions, newest first: `{items: [{revision, source, publishedBy, publishedAt, replacedAt}]}` |
| POST | `/v1/funnels/{funnelCode}/pages/{pageId}/restore` | Put an earlier version back live. Body: `{revision}` |
| POST | `/v1/funnels/{funnelCode}/pages` | Add a step. Body: `{purposeType: "ChatPage"|"ThankYouPage", name?, position?}`. On a live funnel it goes into an unpublished version |
| POST | `/v1/funnels/{funnelCode}/publish` | Publish a funnel version. Body: `{version}`. Returns `{status, liveUrl, warnings}` |
| POST | `/v1/funnels/{funnelCode}/regenerate` | Rebuild as a new unpublished version (202). Poll `GET /v1/funnels/generate/{run_id}` for `version` |

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

**Talking to the user:** call a blueprint by its name only (for example "AI Chat Landing Page"), in lists, questions, confirmations and summaries. Never show a blueprint id; ids are only for requests.

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

### Adding a step, versions and publishing a version

Do these only when the user asks. `POST /v1/funnels/{code}/pages` with `{purposeType: "ChatPage" | "ThankYouPage", name?, position?}` adds a step and returns `{pageId, name, purposeType, version, pending}`. A funnel version has at most one page of each type, and a thank-you page can't be first. A thank-you page goes with a lead form. After a form submit visitors go to the next step in order, so put the form slot on its page first: left out, `position` places a thank-you page straight after the page with the form (else after the first step). After adding one, set the form's `redirectType: "step"` and `redirectPageId` to its `pageId` (`PUT .../form`; read `warnings` in the response). On a live funnel the new step goes into an **unpublished version** (`pending: true`): edit its pages by adding `?version=N` to the page routes (list, get and draft: `GET .../pages?version=N`, `GET`/`PUT .../pages/{pageId}/draft?version=N`; this also works for pages the unpublished version shares with the live one; publishing one such page is refused with `409 in_unpublished_version`: publish the version). Preview it, and make it live only when the user asks with `POST /v1/funnels/{code}/publish` and `{"version": N}`, which returns `{status, liveUrl, warnings}`. `409 newer_version_live` or `version_already_live` means nothing to do: tell the user. Publish nothing on your own.

`POST /v1/funnels/{code}/regenerate` (202) rebuilds a funnel as a new **unpublished version**; the live pages stay as they are. `409 regenerate_running` means one is already running. Poll `GET /v1/funnels/generate/{run_id}` until done; it returns the `version`. Preview it, and publish that version only when asked.

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

   Rules (a break is `422 validation_failed`, with `details[].loc` naming the field, e.g. `["body","fields",2,"options"]`): a field `email` of type `email` that is active and `required: true` is always needed; `type` is one of `text`, `email`, `tel`, `textarea`, `select`, `multiselect`, `radio`, `checkbox`, `date`, `number`, `url`; `select`, `multiselect`, `radio` and `checkbox` need `options` (1-50, each up to 100 characters, no commas, no duplicates) and other types take none; a checkbox can't be `required` (use a one-option `radio` for a required consent); a new `name` is snake_case (`^[a-z][a-z0-9_]{0,39}$`), unique, not `utm_params` and not ending `__cc`; at most 30 fields; label up to 200, placeholder up to 150, title and button text up to 100, disclaimer up to 1000 characters; no `<` or `>` in any of that text (plain text only); colours are `#rrggbb`; `redirectType: "custom"` needs an `https://` `redirectUrl`. A field you leave out is switched off if it is one of the default form's fields (`name`, `first_name`, `last_name`, `email`, `mobile`) and removed otherwise. The response's `republish` is `done`, `not_needed` or `failed`; on `failed` the form is saved but published pages may show the old form until republished, so tell the user. Needs `funnels:read` and `funnels:write`.
4. **Style the card, not the form.** On the page, wrap `<div data-block="lead_form"></div>` in your own element (`.signup-card` with `var(--t-surface)`, padding, a heading). The form's own look comes from its `colors` and `styles`.

## Funnel or Template?

Decide first, before building anything.

- **Build a funnel** (the default) when the user names a specific offer, resource, date or webinar, or asks for a page or funnel to use now. Write real content. See "Building a Funnel".
- **Design a template** when the user asks for a template, style, theme, design or layout to reuse or keep in the gallery, or to "save this design". Write `{{placeholders}}`. See "Designing a Template".
- **Ambiguous** (for example "a lead magnet with our brand colours"): ask one question: "Build a funnel now, or a reusable design for your gallery?"
- In the connector (Claude Desktop or claude.ai) only funnels can be built. Templates are made in Claude Code or with the dashboard's Save as template; say so.

## Building a Funnel

Three blueprints can be built this way: **AI Chat Landing Page**, **Lead Magnet** and **Webinar Registration**. List them with `GET /v1/blueprints?buildable=true` (each has `owner`, `gtmfunnels` or `organisation`, and `type`, its tier such as Free or Paid) and find the `id` by name; ids differ between environments, so never hard-code one. If two buildable blueprints share a name, ask the user which one they mean, describing them by tier, owner and description, never by id. When you tell the user, name the blueprint, never its id. Needs keys with `funnels:build`, `blueprints:read`, `pages:read`, `pages:write` (plus `funnels:read` and `funnels:write` for the form, `pages:publish` to publish, `knowledge:write` to upload sources).

1. **Fetch the rules, every build.** `GET /v1/page-contract` returns `{version, markdown}`. Read the markdown and follow it for this build; never rely on a copy from an earlier build.
2. **Read the plan.** `GET /v1/blueprints/{blueprintId}/build-plan` gives the page briefs, required components (for the AI Chat Landing Page, `chat_widget`), optional ones (`faq`), chat agent instructions and the `inputs` keys to collect. A `404` means that blueprint has no build plan.
3. **Ask the user** clarifying questions before writing anything: the company, offer, audience, tone, and which website URLs or files to draw on. For the AI Chat Landing Page also ask whether they want the FAQ. For a **Lead Magnet** ask for the resource's title and **link**, the audience and the offer (`resource_title`, `resource_link`, `audience`, `offer`). For a **Webinar Registration** ask for the title, date, time, time zone, presenters, agenda and, if they have it, the join link (`webinar_title`, `webinar_date`, `webinar_time`, `webinar_timezone`, `presenters`, `agenda`, `join_link`). Upload any files with `POST /v1/knowledge-base/upload`; the URLs it returns are the `sourceUrls`.
4. **Build.** Send an `Idempotency-Key` header:

```bash
# BLUEPRINT_ID: the id of the blueprint, found by name in GET /v1/blueprints?buildable=true
API="${GTMFUNNELS_API_URL:-https://api.gtmfunnels.ai}"
curl -s -X POST -H "X-API-KEY: $GTMFUNNELS_API_KEY" -H "Idempotency-Key: $(uuidgen)" -H "Content-Type: application/json" \
  -d "{\"blueprintId\": $BLUEPRINT_ID, \"name\": \"Acme Guide\", \"websiteUrls\": [\"https://acme.example\"], \"inputs\": {\"resource_title\": \"Retirement Guide\", \"resource_link\": \"https://acme.example/guide.pdf\"}}" "$API/v1/funnels/build"
```

   Body: `{blueprintId, name, goal, brandCode?, websiteUrls? (max 10), sourceUrls? (max 20), inputs?}`. Always send `goal`: what the user wants the funnel to do, in a sentence or two in their words (up to 2000 characters). It is shown as the funnel's prompt in the dashboard and is the goal if the funnel is regenerated. `inputs` holds the answers from step 3 as up to 20 string values (keys `^[a-z][a-z0-9_]{0,39}$`, values up to 2000 characters) and is saved with the funnel. It returns `201 {funnelCode, status: "draft", pages: [{pageId, name, purposeType}], contractVersion}`. `403` = the org is over its usage limit or the key's user isn't a member; `404` = unknown brand; `422` = the blueprint can't be built by Claude Code yet, or a bad body.
5. **Write the pages** (the Lead Magnet and Webinar Registration funnels have a landing or registration page, a thank-you page and an email) each to a local file, e.g. `page.html`, following the contract and the brief. On the AI Chat Landing Page the chat widget is exactly `<div data-block="chat_widget"></div>` (lower-case, double quotes, nothing inside, once); the server turns it into the widget. The Lead Magnet and Webinar Registration pages have no chat widget: their landing or registration page holds exactly one `<div data-block="lead_form"></div>` inside your own card (steps 7a and 7b). An optional FAQ uses `.faq-container` (or `#faqAccordion`) > `.faq-item` > `.faq-question` + `.faq-answer`. If the brief wants a lead form on the page (the Lead Magnet landing page and the Webinar Registration page do), place exactly `<div data-block="lead_form"></div>` where it goes, inside your own card. No scripts, forms or external stylesheets, except Google Fonts (`fonts.googleapis.com`, `fonts.gstatic.com`, https, written plainly). Colours and fonts go through the `:root` `--t-*` block defined as `var(--gtm-*, fallback)`.
6. **Draft.** `GET /v1/funnels/{code}/pages/{pageId}` for the current `revision`, then `PUT /v1/funnels/{code}/pages/{pageId}/draft` with `{html, baseRevision}` as in "Editing a Page". `422 validation_failed` lists problems in `details[]` (`msg`, `line`): fix every one and send the draft again. `409` means the page changed: read it again and retry.
7. **Preview.** Give the user the `previewUrl` from the draft response. The funnel's own link shows only published pages, so until the page is published it shows the empty placeholder: send the draft's preview link, not the funnel's. Iterate on their feedback with new drafts.
7a. **The email** (Lead Magnet and Webinar Registration). `GET .../pages` lists it with `kind: "email"`. Write one message for every lead, following section 9 of the page contract: table layout about 600px wide, inline styles, literal colours and fonts (no `var()`, no theme variables), no scripts, forms, iframes, `<link>` or `@import`, https or mailto links only, 600 bytes to 100 KB, and no `{{tags}}` (write "Hi there", not a first name). Save it with `PUT .../draft` including a `subject`. The draft's `previewUrl` opens the email itself (valid 7 days). Leads receive it after they sign up, sent to them from the funnel owner's name.
7b. **The resource link** (Lead Magnet). Put the customer's resource link on the thank-you page and in the email's button. Never invent a link or leave `href="#"`; if they have none, ask for it.
8. **Customise the form** if the page has one: read it, propose fields, button text and disclaimer to the user, and `PUT` them as in "Customise the form". This saves at once (it is not a draft); say so. It shows on the live site once the funnel is published.
9. **Publish only when the user asks.** A preview or draft is not a request to publish. Then `POST /v1/funnels/{code}/pages/{pageId}/publish`, and `POST /v1/funnels/{code}/publish` to make the funnel live. Publish the pages, the email and then the funnel. `POST /v1/funnels/{code}/publish` returns `409 pages_not_written` ("Write these pages before publishing: <names>.") while any page still holds the placeholder from the build: write it first. Never publish on your own. Before regenerating a funnel you built (`POST .../regenerate`), warn the user that it replaces the pages you wrote.

## Designing a Template

A template is a reusable style for the organization's gallery. Needs a secret key with `templates:write` and a paid plan to create a draft and to save. Routes (all under `/v1/templates`): `GET /templates?purpose=` lists active styles; `POST /templates/drafts {purpose, fromTemplate?}` returns `draftCode`, `requiredKeys`, `optionalKeys`; `GET /templates/drafts/{draftCode}` returns pages with `sampleUrl`/`outlineUrl` and `missingKeys`; `PUT /templates/drafts/{draftCode}/pages/{key} {templateHtml, schema, sampleContent}`; `POST /templates/drafts/{draftCode}/save {name}`; `DELETE /templates/drafts/{draftCode}` (204).

Purposes and keys: `lead_magnet` (lp, tp, email); `webinar_registration` (lp, tp, email; optional: the other webinar keys (join, completion, live-tp, live-email, reminder-email, evergreen-email)); `hosted_webinar` (join, completion; optional lp, live-tp, live-email, reminder-email); `evergreen_webinar` (join, completion; optional live-tp, evergreen-email). `lp` and `join` are landing pages, each with exactly one `<div data-block="lead_form"></div>` anchor; `tp` is a thank-you page; `live-tp` is the registration thank-you page and holds exactly one `<div data-block="webinar_join"></div>` (the countdown and Join Now button); email keys follow the email rules (section 9).

1. **Read the rules and the brand.** `GET /v1/page-contract` (section 10 is the template rules, section 9 the email rules), the brand (`GET /v1/brands`), and the purpose's keys from the draft you create.
2. **Ask clarifying questions** about the look, tone and structure before writing.
3. **Start a draft** with the purpose. To tweak a style saved through this API, start with `fromTemplate` (its slug); a style saved from a funnel in the dashboard returns `404`.
4. **Design one email first** and reuse its shell (header, colours, footer, layout) for every email key; only the content placeholders differ. A webinar email may use `{{webinar_personal_url}}`; no other `{{...}}` survives into an email.
5. **Push page by page.** Each page has `templateHtml` with `{{section_key}}` placeholders, a flat `schema` (3 to 150 keys, snake_case with a section prefix such as `hero_title`, each a one-line description of at most 300 characters with no `"`, backslash, line break or tab: the AI writes the funnel's copy from it) and `sampleContent` with the same keys, plain text with no markup, each value at most 5000 characters. Write each placeholder exactly `{{key}}`: no spaces inside the braces, no filters. Placeholders and schema keys match one to one; runtime merge tags are not placeholders. The page filled with its sample content must pass the contract. Content images are placeholders; the logo may be fixed. Image URLs (`src`, `srcset`, `poster`) are absolute `https://` URLs (for a library image, the asset's `url` from `GET /v1/assets`; `{{asset:...}}` tokens are refused in templates); links `https://`, `mailto:`, `tel:` or relative; never `data:` or `http:`. No `<noscript>`, `<template>`, `<style>`/`<title>` inside an inline `<svg>`, or `<` in a `<style>`. Template at most 1 MB. A `422` lists every problem in `details[]` (`msg`, `line`) and stores nothing: fix them all and push again.
6. **Give the user both preview links** for each page: `sampleUrl` (with sample content) and `outlineUrl` (the placeholders). They last while the draft exists (at most 7 days) and stop working after save or discard. Iterate on feedback by pushing the page again.
7. **Save only when the user approves**, with a name: `POST .../save`. A `409 template_incomplete` lists missing keys; push them and save again. The result is an active style (`slug`, `themeReady: true`) shown in the org's gallery for blueprints whose pages it covers.
8. **A saved template never changes.** A tweak starts a new draft with `fromTemplate`. Renaming and deleting are done in the dashboard. Discard an unwanted draft with `DELETE`.

## Common Workflows

**Generate and publish a funnel:**
1. `GET /v1/blueprints` — pick a blueprint (to build one yourself, use `GET /v1/blueprints?buildable=true` and see "Building a Funnel")
2. `POST /v1/funnels/generate` with `returnUrl: true` — generates and returns URL
3. Funnel is auto-published when `returnUrl: true`

**Export leads:**
1. `GET /v1/funnels` — find the funnel code
2. `GET /v1/funnels/{funnelCode}/leads/export` — download CSV

**Query knowledge base:**
1. `POST /v1/knowledge-base/query` with `{query: "your question"}`
2. Returns matched documents with relevance scores
