# n8n Automation Workflows

A collection of self-hosted n8n automation workflows for daily digests, news, weather, and meeting notes. All workflows use Claude AI for summarization and analysis.

## Projects

| Folder | Description | Trigger | Status |
|--------|-------------|---------|--------|
| [gh-digest](./gh-digest/) | Daily GitHub activity digest across all repos | Schedule 8:00 AM | ✅ Live |
| [ai-news-digest](./ai-news-digest/) | AI/tech news digest from 9 RSS sources with styled HTML email | Schedule 7:30 AM, 12:30 PM + Webhook | ✅ Live |
| [my-meeting-notes](./my-meeting-notes/) | POST transcript via webhook, get structured notes + email | Webhook | ✅ Live |
| [ai-weather-forecast](./ai-weather-forecast/) | 7-day weather forecast for configurable cities with visual charts | Schedule 7:40 AM + Webhook | 🆕 New |

## Architecture

All workflows follow the same pattern:

```
Trigger → Fetch Data → Build Prompt → Claude API → Format Output → Save File + Email
```

- **AI Engine:** Anthropic Claude (claude-haiku-4-5-20251001) via direct HTTP Request
- **Email:** Styled HTML templates (table-based, email-client safe) via SMTP
- **Storage:** Markdown files saved to `/home/node/output/`
- **APIs:** Open-Meteo (weather), GitHub API (activity), RSS feeds (news)

## Tech Stack

- **n8n** - Self-hosted workflow automation (Docker)
- **Anthropic Claude** - AI summarization and analysis (claude-haiku-4-5-20251001)
- **GitHub API** - Repository activity, commits, PRs, issues (gh-digest)
- **RSS/Atom** - 9 feeds including Hacker News, Ars Technica, TechCrunch, Anthropic, OpenAI (ai-news-digest)
- **Open-Meteo** - 7-day weather forecast API, free, no key required (ai-weather-forecast)
- **SMTP** - Styled HTML email delivery
- **Webhooks** - On-demand triggers for testing and external integrations

## Setup

### Requirements

- Self-hosted n8n instance (Docker)
- Anthropic API key
- SMTP credentials for email delivery
- GitHub PAT (for gh-digest only)

### Import a Workflow

1. Open n8n
2. Go to **Workflows → Import from file**
3. Select the `workflow.json` from the project folder
4. Configure credentials as described in the project README

### Output Directories

Ensure these directories exist on the n8n host:

```bash
mkdir -p /home/node/output/ai-news
mkdir -p /home/node/output/weather
mkdir -p /home/node/output/meetings
mkdir -p /home/node/output/digests
```

## Project Structure

```
n8n-project/
├── gh-digest/              # GitHub activity digest
│   ├── README.md
│   └── workflow.json
├── ai-news-digest/         # AI/tech news from RSS feeds
│   ├── README.md
│   └── workflow.json
├── my-meeting-notes/       # Meeting transcript summarizer
│   ├── README.md
│   └── workflow.json
├── ai-weather-forecast/    # Weather forecast with charts
│   ├── README.md
│   └── workflow.json
├── LICENSE                 # GPL-3.0
└── README.md
```

## License

[MIT](./LICENSE)
