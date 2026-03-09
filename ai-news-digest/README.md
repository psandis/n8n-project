# AI News Digest - n8n Workflow

Daily AI news digest that collects top articles from 9 RSS feeds, summarizes them with Claude, saves a markdown report, and emails it to you.

## Workflow Overview

**Triggers:** Morning (7:30 AM), Midday (12:30 PM), and Webhook on-demand
**Timezone:** Europe/Madrid
**Output:** Markdown file + Gmail email

### Nodes

1. **Morning 7:30AM** - Schedule trigger (cron: `30 7 * * *`)
2. **Midday 12:30PM** - Schedule trigger (cron: `30 12 * * *`)
3. **Webhook On-Demand** - GET webhook at `/webhook/ai-news-digest`
4. **Fetch RSS Feeds** - Code node that fetches and parses 9 RSS feeds (max 10 articles per feed, last 24 hours)
5. **Build Prompt** - Code node that constructs the Claude API request
6. **Claude Summarize** - HTTP Request to Anthropic API (claude-3-5-haiku) to pick the top 10-15 stories organized by topic
7. **Prepare File** - Code node converting the markdown response to a binary file
8. **Save News Digest** - WriteBinaryFile node saving to `/home/node/output/ai-news/`
9. **Email Digest** - Sends the digest to your Gmail

### RSS Sources

| Source | Feed URL |
|--------|----------|
| Hacker News (AI) | `https://hnrss.org/newest?q=AI+OR+artificial+intelligence+OR+LLM+OR+GPT+OR+Claude&points=50` |
| Ars Technica | `https://feeds.arstechnica.com/arstechnica/technology-lab` |
| TechCrunch AI | `https://techcrunch.com/category/artificial-intelligence/feed/` |
| The Verge AI | `https://www.theverge.com/rss/ai-artificial-intelligence/index.xml` |
| Google AI Blog | `https://blog.google/technology/ai/rss/` |
| DeepMind Blog | `https://deepmind.google/blog/rss.xml` |
| Anthropic Blog | `https://www.anthropic.com/rss.xml` |
| OpenAI Blog | `https://openai.com/blog/rss.xml` |
| Ben's Bites | `https://bensbites.beehiiv.com/feed` |

## Setup

### Import

Import `workflow.json` into n8n (tested with n8n 2.7.5).

### Credentials

After importing, assign credentials to these nodes:

**Claude Summarize** — Header Auth credential:
- **Header Name:** `x-api-key`
- **Header Value:** Your Anthropic API key

**Email Digest** — SMTP credential:
- **Host:** `smtp.gmail.com`
- **Port:** `465` (SSL)
- **User:** Your Gmail address
- **Password:** Gmail App Password (Google Account → Security → App Passwords)

Also update the `fromEmail` and `sendTo` fields in the Email Digest node with your Gmail address.

### Output Directory

Ensure the output directory exists on the n8n host:

```bash
mkdir -p /home/node/output/ai-news
```

## Architecture Notes

- Max 10 articles per feed to keep input focused
- Claude picks the top 10-15 stories — quality over quantity
- Uses `n8n-nodes-base.httpRequest` for the Anthropic API call (not langchain nodes)
- Uses `n8n-nodes-base.code` for RSS fetching and data processing
- Each RSS feed fetch is wrapped in try/catch so one failing source does not break the workflow
- Articles are organized by topic (not by source) in the final digest
- Prepare File fans out to both Save File and Email in parallel
- Estimated cost: ~$0.01/day with claude-3-5-haiku
