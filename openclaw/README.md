# GTMFunnels Skill for OpenClaw

Give your OpenClaw agent the ability to manage GTMFunnels — generate funnels, manage leads, configure chatbots, and query knowledge bases.

## Setup

1. Get your API key from [GTMFunnels Dashboard](https://app.gtmfunnels.ai) > My Account > API Keys

2. Copy the `gtmfunnels/` skill folder to your OpenClaw workspace:

```bash
cp -r gtmfunnels/ ~/.openclaw/workspace/skills/gtmfunnels/
```

3. Set the API key in your OpenClaw config or environment:

```bash
export GTMFUNNELS_API_KEY="gtm_live_your_key_here"
```

4. Restart the gateway or run `/new` to load the skill:

```bash
openclaw gateway restart
```

5. Verify the skill is loaded:

```bash
openclaw skills list
```

## What Your Agent Can Do

- **Generate funnels** — "Create a lead magnet funnel for my SaaS product"
- **Manage leads** — "Export the leads from my webinar funnel as CSV"
- **Score leads** — "Score the leads on funnel ABC123"
- **Configure chatbots** — "Enable the chat widget on my landing page"
- **Query knowledge base** — "Search my KB for pricing information"
- **Run workflows** — "Trigger the follow-up workflow for funnel XYZ"

## Environment

| Variable | Default | Description |
|---|---|---|
| `GTMFUNNELS_API_KEY` | — | Required. Your GTMFunnels API key |
| `GTMFUNNELS_API_URL` | `https://api.gtmfunnels.ai` | API base URL |
