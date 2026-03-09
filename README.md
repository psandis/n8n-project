# n8n Projects

A collection of self-hosted n8n automation workflows.

## Projects

| Folder | Description | Status |
|--------|-------------|--------|
| [gh-digest](./gh-digest/) | Daily GitHub activity digest across repos | 🚧 In progress |
| [my-meeting-notes](./my-meeting-notes/) | Paste transcript → Claude summarizes → saved to file/Notion | 🚧 In progress |
| [linkedin-feed](./linkedin-feed/) | LinkedIn content automation | 🚧 In progress |

## Setup

Each project folder contains:
- `README.md` — description, nodes used, setup instructions
- `workflow.json` — importable n8n workflow

### Import a Workflow

1. Open n8n
2. Go to **Workflows → Import from file**
3. Select the `workflow.json` from the project folder
4. Configure credentials as described in the project README
