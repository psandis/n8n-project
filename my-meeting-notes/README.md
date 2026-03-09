# My Meeting Notes - n8n Workflow

POST a meeting transcript via webhook — get structured notes with summary, decisions, action items, and discussion points. Saved as markdown and emailed to you.

## Workflow Overview

**Trigger:** Webhook (POST)
**Endpoint:** `/webhook/meeting-notes`
**Timezone:** Europe/Madrid
**Output:** Markdown file + styled HTML email

### Nodes

1. **Receive Transcript** - Webhook (POST `/webhook/meeting-notes`)
2. **Parse & Build Prompt** - Code node that detects format, strips timestamps from subtitle formats, and builds the prompt
3. **Claude Summarize** - HTTP Request to Anthropic API (claude-haiku-4-5-20251001)
4. **Prepare Output** - Code node converting markdown to email-safe HTML with styled template
5. **Save Meeting Notes** - Writes markdown to `/home/node/output/meetings/notes/`
6. **Email Notes** - Sends styled meeting notes to your email

### Supported Formats

| Format | Extension | Source |
|--------|-----------|--------|
| Plain text | `.txt` | Manual, Otter.ai export |
| Markdown | `.md` | Manual |
| WebVTT | `.vtt` | Zoom, browser recordings |
| SubRip | `.srt` | Google Meet, media players |
| YouTube | `.sbv` | YouTube Studio |

Timestamps and sequence numbers are automatically stripped from subtitle formats.

### Output Structure

- **Summary** - 3-5 sentence executive summary
- **Key Decisions** - Formal decisions made during the meeting
- **Action Items** - Checklist with owners and deadlines
- **Discussion Points** - Organized by topic
- **Open Questions** - Unresolved items needing follow-up
- **Attendees** - Extracted from transcript if identifiable

## Setup

### Import

Import `workflow.json` into n8n via **Workflows → Import from file**.

### Credentials

**Claude Summarize** — Header Auth credential:
- **Header Name:** `x-api-key`
- **Header Value:** Your Anthropic API key

**Email Notes** — SMTP credential:
- Configure with your email provider settings

Also update the `fromEmail` and `sendTo` fields in the Email Notes node.

### Directories

Ensure the output directory exists on the n8n host:

```bash
mkdir -p /home/node/output/meetings/notes
```

## Usage

POST a JSON payload to the webhook:

```bash
curl -X POST http://your-n8n-host:5678/webhook/meeting-notes \
  -H "Content-Type: application/json" \
  -d '{"transcript": "your transcript text here...", "title": "Meeting Name"}'
```

Or pipe a file:

```bash
curl -X POST http://your-n8n-host:5678/webhook/meeting-notes \
  -H "Content-Type: application/json" \
  -d "{\"transcript\": \"$(cat transcript.txt | sed 's/\"/\\\"/g' | tr '\n' ' ')\", \"title\": \"My Meeting\"}"
```

## Cost

~$0.005 per transcript with claude-haiku-4-5-20251001.
