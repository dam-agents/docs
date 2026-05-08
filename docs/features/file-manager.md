# File Manager

Browse, edit, and upload files in your agent's persistent workspace.

## Overview

The File Manager gives you direct access to your agent's filesystem:

- `/workspace` — your main working directory (repos, project files)
- `/home/agent` — agent home directory (configuration, memory, skills)

## Capabilities

| Action | Description |
|--------|-------------|
| Browse | Navigate the directory tree |
| View | Read file contents with syntax highlighting |
| Edit | Modify files directly in the browser |
| Upload | Add files from your local machine to the workspace |
| Download | Save workspace files to your local machine |

## Editor Features

The built-in editor supports:

- Syntax highlighting for common languages (Python, TypeScript, Go, YAML, JSON, Markdown, etc.)
- Line numbers
- Direct in-browser editing with save

## Limits

| Limit | Value |
|-------|-------|
| Max viewable file size | 1 MB |
| Max upload size | 50 MB |
| Supported formats | All text files; binary files are downloadable but not viewable |

## Key Notes

- Files persist across sessions, restarts, and hibernation
- Files modified in chat sessions appear immediately in the File Manager (and vice versa)
- The workspace is shared across all sessions for an agent
- Uploaded files land in the current directory you're browsing
