# Google Workspace

The Google Workspace agent automates Gmail, Calendar, Drive, and Sheets tasks using your Google account.

## When to Use Google Workspace

- Triaging and summarizing email
- Managing calendar events and scheduling
- Uploading and organizing Drive files
- Reading and writing spreadsheet data
- Automating repetitive Google Workspace workflows

## Required Connections

| Connection | Required? | Purpose |
|------------|-----------|---------|
| Google OAuth | **Yes** | Access to Gmail, Calendar, Drive, and Sheets APIs |
| Anthropic API key | **Yes** | Powers the AI agent |

## Available Skills

The Google Workspace agent comes pre-installed with:

| Skill | Purpose |
|-------|---------|
| `gmail-triage` | Scan and categorize inbox messages, identify action items |
| `calendar-agenda` | Summarize upcoming events, find free time, manage meetings |
| `drive-upload` | Upload files to Google Drive, organize folders |
| `drive-manage` | Search, move, and organize Drive files |
| `sheets-data` | Read, write, and analyze spreadsheet data |

## Example Prompts

```
Triage my inbox for the past week. Flag anything that needs a response.
```

```
What meetings do I have tomorrow? Summarize the agenda for each.
```

```
Upload the Q4 report from /workspace/reports/ to my "Reports" Drive folder.
```
