# File Manager

Browse, edit, and upload files in your agent's persistent workspace.

## Overview

The File Manager gives you direct access to your agent's filesystem:

- `/home/agent` — the root of the tree (configuration, memory, skills)
- `/home/agent/work` — your main working directory (repos, project files), a folder inside it

Two names are reserved by the platform and won't appear: `.triggers` and `.initialized`. Per-import
staging folders (`.import-staging-…`) are hidden too. Ordinary repo noise like `.git` and
`node_modules` is browsable as usual.

## Capabilities

| Action | Description |
|--------|-------------|
| Browse | Navigate the directory tree |
| View | Read file contents with syntax highlighting, or as a rendered preview (images, PDF, SVG, Markdown, HTML) |
| Fullscreen | Open a rendered preview at full size in a dialog via the **Open fullscreen** button |
| Edit | Modify files directly in the browser |
| Upload | Add files from your local machine to the workspace |
| Download | Save workspace files to your local machine |
| Move | Drag a file or folder onto a folder in the tree to move it there |

## Editor Features

The built-in editor supports:

- Syntax highlighting when viewing a file (Python, TypeScript, Go, YAML, JSON, Bash, SQL, etc.); the
  editor itself highlights JavaScript/TypeScript, JSON, Markdown and YAML
- Line numbers
- Direct in-browser editing with save

SVG, Markdown and HTML files open rendered; **Raw** flips to the source and **Render** flips back.

When viewing a file that renders (image, PDF, SVG, Markdown, or HTML), an **Open fullscreen** button opens the rendered preview at full size in a dialog — it's offered only while the rendered view is showing. Press **Escape** or the close button to leave it; switching to another file also returns you to the side panel.

## Limits

| Limit | Value |
|-------|-------|
| Max previewable file size | 50 MB |
| Max download / transfer size | 50 MB |
| Max upload file size | 50 MB |
| Max import bundle size | 5 GiB (your operator can lower this — and should, if your agent's disk is smaller) |
| Supported formats | Text files with syntax highlighting; images, PDF, SVG, Markdown and HTML render as previews; other binary files open as a hex dump of the first kilobyte |

## Key Notes

- Each row has an action menu (the hover **⋯** button or right-click) with context-appropriate actions — for a file: **Edit**, **Download**, **Rename**, **Delete**; for a folder: **New file…**, **New folder…**, **Upload files here…**, **Rename**, **Delete**
- **Move by drag and drop** — drag a file or folder onto a destination folder in the tree to move it there (a folder can't be moved into itself or its own subtree)
- **Uploading files** lands them in the folder you upload *into* — use a folder's **Upload files here…** action, or drag them onto it. If a file of that name already exists, you're asked before it's overwritten.
- **Uploading a whole folder** is a bulk import instead: each top-level folder you bring replaces any existing top-level entry of the same name under `/home/agent/work`, and other top-level entries are left alone. Nothing is filtered on the way up — the folder goes as it sits, `node_modules` and `.venv` included — so prune it first, or use `dam import`, which skips `node_modules`, `.venv`, `__pycache__` and `.DS_Store` for you. Either way `.git` is kept.
- Files modified in chat sessions appear immediately in the File Manager (and vice versa)
- The workspace is shared across all sessions for an agent
- Files you attach to a chat message are saved under your home directory, grouped by session, and handed to the agent as a path it can read — they don't land in your working directory

!!! note "What survives hibernation"
    Everything under `/home/agent` — your work, `$HOME`, installed language packages — comes back
    after a sleep, **except `~/.cache`**, which is cleared on every wake. Software installed
    *outside* your home directory (system packages, `/usr/local`) does not survive either. Bake
    those into a [custom image](../guides/custom-image.md) instead.
