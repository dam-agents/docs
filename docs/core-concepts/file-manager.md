# File Manager

Browse, edit, and upload files in your sandbox's persistent workspace.

## Overview

The File Manager gives you direct access to your sandbox's filesystem:

- `/home/agent/work` — your main working directory (repos, project files)
- `/home/agent` — home directory (configuration, memory, skills)

Two names under the working directory are reserved by the platform and won't appear: `.triggers` and
`.initialized`. Ordinary repo noise like `.git` and `node_modules` is browsable as usual.

## Capabilities

| Action | Description |
|--------|-------------|
| Browse | Navigate the directory tree |
| View | Read file contents with syntax highlighting, or as a rendered preview (images, PDF, SVG, Markdown, HTML) |
| Fullscreen | Open a rendered preview at full size in a dialog via the **Fullscreen** button |
| Edit | Modify files directly in the browser |
| Upload | Add files from your local machine to the workspace |
| Download | Save workspace files to your local machine |
| Move | Drag a file or folder onto a folder in the tree to move it there |

## Editor Features

The built-in editor supports:

- Syntax highlighting for common languages (Python, TypeScript, Go, YAML, JSON, Bash, SQL, etc.)
- Line numbers
- Direct in-browser editing with save

When viewing a file that renders (image, PDF, SVG, Markdown, or HTML), a **Fullscreen** button opens the rendered preview at full size in a dialog. Press **Escape** or the close button to leave it; switching to another file also returns you to the side panel.

## Limits

| Limit | Value |
|-------|-------|
| Max previewable file size | 10 MB |
| Max download / transfer size | 50 MB |
| Max upload file size | 50 MB |
| Max import bundle size | 5 GB (your operator can lower this) |
| Supported formats | Text files with syntax highlighting; images, PDF, SVG, Markdown and HTML render as previews; other binary files open as a hex dump |

## Key Notes

- Each row has an action menu (the hover **⋯** button or right-click) with context-appropriate actions — for a file: Edit, Download, Rename, Delete; for a folder: New file, New folder, Upload here, Rename, Delete
- **Move by drag and drop** — drag a file or folder onto a destination folder in the tree to move it there (a folder can't be moved into itself or its own subtree)
- **Uploading files** lands them in the folder you upload *into* — use a folder's **Upload files here…** action, or drag them onto it. If a file of that name already exists, you're asked before it's overwritten.
- **Uploading a whole folder** (or using `dam import`) is a bulk import instead: each top-level folder you bring replaces any existing top-level entry of the same name under `/home/agent/work`, and other top-level entries are left alone. `node_modules`, `.venv`, `__pycache__` and `.DS_Store` are skipped automatically; `.git` is kept.
- Files modified in chat sessions appear immediately in the File Manager (and vice versa)
- The workspace is shared across all sessions for a sandbox
- Files you attach to a chat message are saved under your home directory, grouped by session, and handed to the agent as a path it can read — they don't land in your working directory

!!! note "What survives hibernation"
    Everything under `/home/agent` — your work, `$HOME`, installed language packages — comes back
    after a sleep, **except `~/.cache`**, which is cleared on every wake. Software installed
    *outside* your home directory (system packages, `/usr/local`) does not survive either. Bake
    those into a [custom image](../guides/custom-image.md) instead.
