# GTMFunnels Skill for Claude Code

Give Claude Code access to the GTMFunnels API: manage funnels, leads, chatbots and knowledge bases, and improve or fix your funnel pages, through natural conversation.

## Setup

1. Get your API key from [GTMFunnels Dashboard](https://app.gtmfunnels.ai) > My Account > API Keys. To edit pages, give it the Pages scopes: Pages (Read), Pages (Draft) and, to publish, Pages (Publish).
2. Copy the skill folder into your project (or into `~/.claude/skills/` to use it in every project):

```bash
mkdir -p .claude/skills
cp -r gtmfunnels/ .claude/skills/gtmfunnels/
```

3. Set the environment variable:

```bash
export GTMFUNNELS_API_KEY="gtm_live_your_key_here"
```

4. Optionally set the base URL (defaults to production):

```bash
export GTMFUNNELS_API_URL="https://dev-api.gtmfunnels.ai"  # for dev
```

## What Claude Can Do

- **Funnels** — List, generate, publish, unpublish, update, delete funnels
- **Pages** — Pull a page's HTML, edit it, save a draft with a preview link, publish it, and restore an earlier version
- **Leads** — List leads, create leads, export as CSV, score leads with AI
- **Chat Settings** — Get/update chat widget configuration, get embed code
- **Knowledge Base** — List documents, upload new content, query with RAG
- **Workflows** — List executions, trigger workflow runs, check status
- **Blueprints** — List available funnel templates
- **Analytics** — View funnel metrics, account usage

## Example Prompts

```
"List all my active funnels"
"Generate a lead magnet funnel for my SaaS product at https://myapp.com"
"Show me the leads for funnel ABC123"
"Make the headline on the ABC123 landing page more specific and send me a preview"
"Undo the last change to the ABC123 landing page"
"Score the leads on my latest funnel"
"Upload this PDF to my knowledge base"
"What's my API usage this month?"
```
