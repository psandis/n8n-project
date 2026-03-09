# GitHub Activity Digest Workflow

An n8n workflow that automatically generates a daily markdown digest of all GitHub activity across every repository under the `psandis` account.

## What it does

1. **Triggers daily at 8:00 AM** (Europe/Madrid timezone)
2. **Fetches all repositories** under the `psandis` GitHub account
3. **Collects activity from the last 24 hours** for each repo: commits, pull requests, and issues
4. **Sends the data to Anthropic Claude** which summarizes it into a clean, structured markdown digest organized by repository and activity type
5. **Saves the digest** as a markdown file to disk

## Prerequisites

- **n8n** instance (self-hosted or cloud)
- **GitHub Personal Access Token (PAT)** with `repo` scope (or `public_repo` if you only have public repos)
- **Anthropic API key** for Claude summarization

## How to import and configure

### 1. Import the workflow

- Open your n8n instance
- Go to **Workflows** and click **Add Workflow** (or use the menu)
- Click the **...** menu in the top-right, select **Import from File**
- Select `workflow.json` from this directory

### 2. Configure GitHub credentials

- Go to **Settings > Credentials** in n8n
- Create a new **Header Auth** credential:
  - **Name**: `GitHub PAT Header Auth`
  - **Header Name**: `Authorization`
  - **Header Value**: `Bearer ghp_YOUR_GITHUB_PAT_HERE`
- Open the workflow and update all HTTP Request nodes to use this credential (they will show a warning icon until linked)

### 3. Configure Anthropic credentials

- In **Settings > Credentials**, create a new **Anthropic API** credential:
  - **Name**: `Anthropic API`
  - **API Key**: your Anthropic API key
- Link it to the "Claude Summarize Digest" node

### 4. Activate the workflow

- Toggle the workflow to **Active**
- It will run automatically every day at 8:00 AM Madrid time

## Where digests are saved

Digest files are written to:

```
/home/node/.n8n/digests/digest-YYYY-MM-DD.md
```

Make sure the `/home/node/.n8n/digests/` directory exists on your n8n server. You can create it by running:

```bash
mkdir -p /home/node/.n8n/digests
```

If running n8n in Docker, this path is inside the container. Map a volume if you want to access digests from the host:

```bash
docker run -v /path/on/host/digests:/home/node/.n8n/digests ...
```

## Workflow nodes overview

| Node | Type | Purpose |
|------|------|---------|
| Daily 8AM Trigger | Schedule Trigger | Fires every day at 08:00 Europe/Madrid |
| List All Repos | HTTP Request | GET all repos for `psandis` |
| Fetch Commits Per Repo | HTTP Request | GET commits from last 24h per repo |
| Fetch PRs Per Repo | HTTP Request | GET pull requests updated in last 24h per repo |
| Fetch Issues Per Repo | HTTP Request | GET issues updated in last 24h per repo |
| Merge Activity Data | Merge | Combines the three activity streams |
| Aggregate Activity | Code | Structures all data into a single payload |
| Claude Summarize Digest | Anthropic Chat Model | Generates the markdown digest |
| Convert to Binary | Convert to File | Converts text output to binary for file writing |
| Save Digest File | Write Binary File | Writes the `.md` file to disk |
