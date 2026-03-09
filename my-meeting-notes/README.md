# My Meeting Notes - n8n Workflow

Drop a meeting transcript into a folder — get structured notes with summary, decisions, action items, and discussion points. Emailed to you automatically.

## Workflow Overview

**Trigger:** Watch folder (polls every minute) + Webhook on-demand
**Timezone:** Europe/Madrid
**Output:** Markdown file + styled HTML email

### Nodes

1. **Watch Inbox** - Local File Trigger watching `/home/node/output/meetings/inbox/`
2. **Webhook On-Demand** - GET webhook at `/webhook/ai-meeting-notes`
3. **Read Transcript** - Reads the dropped file as binary
4. **Parse & Build Prompt** - Code node that detects format (.txt, .md, .vtt, .srt, .sbv), strips timestamps from subtitle formats, and builds the Claude prompt
5. **Claude Summarize** - HTTP Request to Anthropic API (claude-haiku-4-5-20251001)
6. **Prepare Output** - Code node converting markdown to email-safe HTML with styled template
7. **Save Meeting Notes** - Writes markdown to `/home/node/output/meetings/notes/`
8. **Email Notes** - Sends styled meeting notes to your email

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

Import `workflow.json` into n8n.

### Credentials

**Claude Summarize** — Header Auth credential:
- **Header Name:** `x-api-key`
- **Header Value:** Your Anthropic API key

**Email Notes** — SMTP credential:
- Configure with your email provider settings

Also update the `fromEmail` and `sendTo` fields in the Email Notes node.

### Directories

Ensure the directories exist on the n8n host:

```bash
mkdir -p /home/node/output/meetings/inbox
mkdir -p /home/node/output/meetings/notes
```

## Usage

Drop a transcript file (`.txt`, `.md`, `.vtt`, `.srt`, `.sbv`) into `/home/node/output/meetings/inbox/`. The workflow picks it up within a minute, summarizes it, saves notes to `/home/node/output/meetings/notes/`, and emails you.

**Tip:** Name the file descriptively (e.g., `2026-01-15-product-roadmap.txt`) — the filename becomes the meeting title.

## Cost

~$0.005 per transcript with claude-haiku-4-5-20251001.
