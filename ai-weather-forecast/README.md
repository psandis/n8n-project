# AI Weather Forecast - n8n Workflow

Daily weather forecast that fetches 7-day forecasts from Open-Meteo for configurable cities, summarizes them with Claude, and delivers a styled HTML email with temperature bar charts, precipitation graphs, and detailed data tables.

## Workflow Overview

**Trigger:** Morning (7:40 AM) and Webhook on-demand
**Timezone:** Europe/Madrid
**Output:** Markdown file + HTML email

### Nodes

1. **Morning 7:40AM** - Schedule trigger (cron: `40 7 * * *`)
2. **Webhook On-Demand** - GET webhook at `/webhook/ai-weather-forecast`
3. **Fetch Weather** - Code node that calls Open-Meteo API for each configured city (free, no API key)
4. **Build Prompt** - Code node that constructs the Claude API request with weather data
5. **Claude Summarize** - HTTP Request to Anthropic API (claude-haiku-4-5-20251001) for natural language analysis
6. **Prepare Output** - Code node that builds HTML email with charts/tables from raw data + Claude's text
7. **Save Weather Forecast** - WriteBinaryFile node saving to `/home/node/output/weather/`
8. **Email Forecast** - Sends the styled HTML forecast email

### Default Cities

| City | Coordinates | Timezone |
|------|-------------|----------|
| Malaga | 36.72, -4.42 | Europe/Madrid |
| Madrid | 40.42, -3.70 | Europe/Madrid |
| Faro (Algarve) | 37.02, -7.93 | Europe/Lisbon |
| Nice | 43.71, 7.27 | Europe/Paris |

Additional cities (London, Paris, Stockholm, Helsinki, New York, Barcelona, Lisbon, Rome) are available as comments in the Fetch Weather node.

### Email Features

- Claude's natural language overview, daily highlights, and travel tips
- Reading guide with UV index, wind speed, and weather icon legend
- Per-city temperature range bar charts (color-coded blue to red)
- Per-city precipitation probability bars
- Per-city detailed 7-day forecast tables (max/min C/F, feels-like, rain%, precipitation, wind km/h, UV with label)
- Dark theme, table-based layout (email-client safe)

## Setup

### Import

Import `workflow.json` into n8n.

### Credentials

After importing, assign credentials to these nodes:

**Claude Summarize** - Header Auth credential:
- **Header Name:** `x-api-key`
- **Header Value:** Your Anthropic API key

**Email Forecast** - SMTP credential:
- Configure your SMTP provider
- Update the `fromEmail` and `sendTo` fields in the Email Forecast node

### Output Directory

Ensure the output directory exists on the n8n host:

```bash
mkdir -p /home/node/output/weather
```

### Configuration

Edit the cities array at the top of the **Fetch Weather** Code node to add, remove, or change cities. Find coordinates and IANA timezones at [open-meteo.com](https://open-meteo.com/).

## Architecture Notes

- Uses Open-Meteo API (free, no key required) for weather data
- Uses `n8n-nodes-base.httpRequest` for the Anthropic API call (not langchain nodes)
- Claude provides natural language analysis; charts and tables are built from raw data for accuracy
- Global temperature range is calculated across all cities for consistent bar chart scaling
- Each city API call is wrapped in try/catch so one failing city does not break the workflow
- Prepare Output fans out to both Save File and Email in parallel
- Estimated cost: ~$0.005/day with claude-haiku-4-5-20251001
