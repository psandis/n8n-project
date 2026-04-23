# my-media-helper

5-agent LinkedIn post pipeline. POST a topic via webhook, get a polished draft by email.

## How it works

```
POST /webhook/my-media-helper { "topic": "..." }
         |
         v
  Load History (reads published/ folder)
         |
         v
   Draft Agent --> Critic Agent --> Finetune Agent --> Prepare Agent --> Final Check Agent
         |
         v
  Email (final post + critic feedback) + saved draft file
```

Each agent is a Code node that builds a prompt + an HTTP Request to the Anthropic API. No n8n AI Agent nodes.

## Agents

| Agent | Role |
|---|---|
| Draft | Writes the initial post matching voice from history |
| Critic | Gives numbered critique (max 8 points): voice match, authenticity, AI phrases, structure |
| Finetune | Rewrites the draft addressing the critic's points |
| Prepare | Formats for LinkedIn: paragraph spacing, hashtags only if author uses them |
| Final Check | Outputs APPROVED or NEEDS REVISION + corrected final post |

## History management

- Published posts live in `/home/node/output/my-media-helper/published/` on the server
- All agents read from `published/` to learn voice and style
- Drafts are saved to `/home/node/output/my-media-helper/drafts/`
- After you actually post on LinkedIn, move the draft file to `published/` manually
- Only approved/posted content goes into history — bad drafts never corrupt the voice model

## Email output

- Status badge (APPROVED / NEEDS REVISION)
- Final post, ready to copy
- Critic feedback visible for context

## Setup

### Trigger

Webhook — POST to:
```
https://your-n8n-host/webhook/my-media-helper
```

Body:
```json
{ "topic": "What to write about" }
```

### Server directories

Create on the server before first run:
```
mkdir -p /home/node/output/my-media-helper/published
mkdir -p /home/node/output/my-media-helper/drafts
```

Seed published posts by copying your existing LinkedIn posts as `.md` files into `published/`.

### Credentials (set in n8n UI)

| Node | Credential |
|---|---|
| Draft Claude, Critic Claude, Finetune Claude, Prepare Claude, Final Check Claude | Header Auth — `x-api-key: <Anthropic key>` |
| Send Email | SMTP |

## Architecture

- Model: `claude-haiku-4-5-20251001` (all 5 agents)
- History: read at runtime from published folder — no database
- File output: `draft-YYYY-MM-DDTHH-MM-SS.md`
- Estimated cost per run: ~$0.002 (5x Haiku calls, ~1-2k tokens each)
