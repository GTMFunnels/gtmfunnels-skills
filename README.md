# GTMFunnels Skills

Pre-built skills for integrating GTMFunnels with AI agent platforms.

## Available Skills

### `/claude` — Claude Code Skill
A Claude Code custom skill that gives Claude access to the GTMFunnels API. Claude can list funnels, generate landing pages, manage leads, query knowledge bases, and configure chat widgets — all through natural conversation.

**Setup:** See [claude/README.md](claude/README.md)

### `/openclaw` — OpenClaw Skill
An OpenClaw agent skill that connects OpenClaw agents to GTMFunnels. Enables OpenClaw agents to manage funnels, leads, and chatbots as part of their tool repertoire.

**Setup:** See [openclaw/README.md](openclaw/README.md)

## API Reference

Both skills use the GTMFunnels Developer API. Full documentation at [developer.gtmfunnels.ai](https://developer.gtmfunnels.ai).

**Base URL:** `https://api.gtmfunnels.ai/v1` (production) | `https://dev-api.gtmfunnels.ai/v1` (development)

**Authentication:** `X-API-KEY` header with a key from My Account > API Keys.
