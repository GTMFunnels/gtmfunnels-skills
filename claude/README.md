# GTMFunnels Skill for Claude Code

Give Claude Code access to the GTMFunnels API — manage funnels, leads, chatbots, and knowledge bases through natural conversation.

## Setup

1. Get your API key from [GTMFunnels Dashboard](https://app.gtmfunnels.ai) > My Account > API Keys
2. Copy the `.claude/` folder into your project root
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
"Score the leads on my latest funnel"
"Upload this PDF to my knowledge base"
"What's my API usage this month?"
```
