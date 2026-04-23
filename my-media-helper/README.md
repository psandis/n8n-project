# my-media-helper

LinkedIn post drafting pipeline. POST a topic + story via webhook, get 3 draft versions scored and emailed.

## How to trigger

```bash
curl -X POST https://<your-n8n-host>/webhook/my-media-helper \
  -H "Content-Type: application/json" \
  -d '{
    "topic": "what the post is about",
    "story": "2-3 sentences of real experience - what happened, what you tried, what you found",
    "audience": "tech professionals",
    "email": "<recipient-email>"
  }'
```

- `topic`: the subject of the post
- `story`: real experience behind it - the more specific the better
- `audience`: who you are writing for (default: tech professionals)
- `email`: where to send the result (default: petri.sandholm@gmail.com)

## Pipeline

```
POST /webhook/my-media-helper
         |
         v
  Load History
  Reads all .md files from published/ to learn voice and style
         |
         v
  Story Extractor (Claude)
  Extracts: hook, conflict, shift, lesson, proof, closing question
         |
         v
  Draft 1 (Claude) - SLA framework
  Story, Lesson, Application structure
         |
         v
  Draft 2 (Claude) - MRS framework
  Mistake, Realization, Shift structure
         |
         v
  Draft 3 (Claude) - Contrarian framework
  Challenges a common assumption
         |
         v
  Critic (Claude)
  Scores all 3 drafts against 9-point checklist, picks the best
         |
         v
  Save Draft (.md file) + Send Email (3 versions + scores)
```

## What each step does

### Load History
Reads every `.md` file from `/home/node/output/my-media-helper/published/` and feeds them to all agents as voice examples. This is how the pipeline learns to write like you.

### Story Extractor
Takes your raw topic + story and extracts structured building blocks:
- `hook`: opening line under 10 words
- `conflict`: the specific challenge with real details
- `shift`: the turning point or realization
- `lesson`: the single key insight
- `proof`: a specific detail, number, or outcome
- `question`: closing question for comments

These elements are reused across all 3 drafts.

### Draft 1: SLA (Story, Lesson, Application)
Structure: open with hook, tell story with specific details, state the lesson, show how to apply it, end with question.

### Draft 2: MRS (Mistake, Realization, Shift)
Structure: bold admission of a mistake or wrong assumption, describe the mistake, the moment of realization, what changed and why it matters, end with question.

### Draft 3: Contrarian
Structure: challenge conventional thinking, what most people believe, your experience that disproves it, the real insight, provocative closing question.

### Critic
Scores all 3 drafts against a concrete 9-point checklist:
1. Hook under 10 words
2. First 140 characters hooks before "see more"
3. Total length 1300-1600 characters
4. Paragraphs 1-3 lines max
5. No corporate buzzwords (leverage, utilize, delve, synergy)
6. Personal story or specific proof present
7. Ends with a question
8. No external links
9. Reads like a human, not AI

Outputs a score out of 10 per draft, improvement notes, and picks the best.

## Email output

- LinkedIn blue header with topic and date
- Recommended version highlighted in green
- All 3 drafts with score and character count
- Critic evaluation with full checklist breakdown at the bottom

## History management

- Published posts live in `/home/node/output/my-media-helper/published/` on the server
- After you post on LinkedIn, copy the draft file from `drafts/` to `published/` manually
- Seed with existing LinkedIn posts as `.md` files - the more the better
- Only approved/posted content goes into history

## Server directories

Create before first run:
```bash
docker exec aibots-n8n mkdir -p /home/node/output/my-media-helper/published
docker exec aibots-n8n mkdir -p /home/node/output/my-media-helper/drafts
```

Copy existing LinkedIn posts into published:
```bash
# On your Mac, copy to server /tmp first
scp post-*.md user@your-server:/tmp/

# On the server, copy into the container
for f in /tmp/post-*.md; do docker cp "$f" aibots-n8n:/home/node/output/my-media-helper/published/; done
```

## Credentials (set in n8n UI)

| Node | Credential |
|---|---|
| Story Extractor Claude | Header Auth: x-api-key = Anthropic key |
| Draft 1 Claude | Header Auth: x-api-key = Anthropic key |
| Draft 2 Claude | Header Auth: x-api-key = Anthropic key |
| Draft 3 Claude | Header Auth: x-api-key = Anthropic key |
| Critic Claude | Header Auth: x-api-key = Anthropic key |
| Send Email | SMTP |

## Requirements

- `NODE_FUNCTION_ALLOW_BUILTIN=fs,path` in docker-compose environment for the n8n service (needed for Load History to read files)

## Architecture

- Model: `claude-haiku-4-5-20251001` (all 5 Claude calls)
- History: read at runtime from published folder, no database
- Draft files saved as: `draft-YYYY-MM-DDTHH-MM-SS.md`
- Estimated cost per run: ~$0.003 (6x Haiku calls, 1-2k tokens each)
