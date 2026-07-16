# File Manager

Browse, edit, and upload files in your agent's persistent workspace.

## Overview

The File Manager gives you direct access to your agent's filesystem:

- `/home/agent/work` — your main working directory (repos, project files)
- `/home/agent` — agent home directory (configuration, memory, skills)

## Capabilities

| Action | Description |
|--------|-------------|
| Browse | Navigate the directory tree |
| View | Read file contents with syntax highlighting, or as a rendered preview (images, PDF, SVG, Markdown, HTML) |
| Fullscreen | Open a rendered preview at full size in a dialog via the **Fullscreen** button |
| Edit | Modify files directly in the browser |
| Upload | Add files from your local machine to the workspace |
| Download | Save workspace files to your local machine |

## Editor Features

The built-in editor supports:

- Syntax highlighting for common languages (Python, TypeScript, Go, YAML, JSON, Markdown, etc.)
- Line numbers
- Direct in-browser editing with save

When viewing a file that renders (image, PDF, SVG, Markdown, or HTML), a **Fullscreen** button opens the rendered preview at full size in a dialog. Leaving fullscreen or switching to another file returns to the side panel.

## Limits

| Limit | Value |
|-------|-------|
| Max viewable file size | 50 MB |
| Max upload file size | 50 MB |
| Max import bundle size | 5 GB |
| Supported formats | All text files; binary files are downloadable but not viewable |

## Key Notes

- Each row has an action menu (the hover **⋯** button or right-click) with context-appropriate actions — for a file: Edit, Download, Rename, Delete; for a folder: New file, New folder, Upload here, Rename, Delete
- Files persist across sessions, restarts, and hibernation
- Files modified in chat sessions appear immediately in the File Manager (and vice versa)
- The workspace is shared across all sessions for an agent
- Uploaded files are extracted to the workspace root (`/home/agent/work`), replacing any existing top-level entry with the same name — regardless of which directory you're currently browsing
- Files attached to a chat are staged into the session's workspace and passed to the agent as a path it can read. The Bob agent accepts chat file attachments this way
