# CleoLM for Claude

Timestamped, citable transcripts of YouTube videos and podcasts for your agent.
Search 158 continuously synced channels plus anything you ingest; every answer
links to the exact second it was said. Your model does the reasoning; CleoLM
supplies the source.

## Install

**Claude Code**

```
/plugin marketplace add ibenna/cleolm-agent-kit
/plugin install cleolm@cleolm
```

Then run `/mcp` and sign in to CleoLM.

**Claude.ai / Desktop / Cowork:** Settings → Connectors → Add custom connector →
`https://cleolm.com/mcp`.

## What's inside

- **MCP connector** (`.mcp.json`) → `https://cleolm.com/mcp` (OAuth, or an API key from
  [cleolm.com/developers](https://cleolm.com/developers))
- **Skills**
  - `media-research`: what experts have said about a topic, cited to timestamps
  - `episode-brief`: ingest a video or episode and produce a timestamped brief
  - `channel-monitor`: what's new from a channel or show

## Pricing

Agent access is included with CleoLM Pro ($25/mo, 1,000 credits) and Max ($99/mo,
5,000 credits). Search is 1 credit; reading passages is free. Prepaid credit packs,
never auto-recharge. Details: [cleolm.com/agents](https://cleolm.com/agents).

Support: support@cleolm.com
