# Artifacts

An **artifact** is a finished output your agent publishes to keep — a report, a dashboard, a
generated page, a data file. Unlike a file in the sandbox workspace, an artifact **outlives the
sandbox that made it**: delete the sandbox and the artifact stays in your library.

Artifacts are yours, not the sandbox's. The library is owner-scoped — you see your own.

## What an artifact can be

| Kind | Renders as |
|------|-----------|
| `html` | A live page |
| `jsx` | A live React component |
| `markdown` | Formatted prose |
| `code` | Syntax-highlighted source |
| `text` | Plain text |
| `binary` | Anything else — downloadable (images, PDFs, archives, datasets) |

The kind is detected from the file name and content when the agent doesn't set it. HTML and JSX
artifacts render and run, which is what makes a dashboard or an interactive report work — those
aren't separate kinds, just things you author as HTML, JSX, or markdown.

!!! note "Self-contained only"
    A rendered artifact is a single file. Anything that exists only inside the sandbox — a companion
    file, an installed package, a running server — won't exist for a viewer. Resources must be
    inlined or referenced by absolute public URL.

    A JSX artifact must export its React component as the **default export**, and it pulls React and
    charting libraries from public CDNs in the viewer's browser — so it needs the viewer to be
    online.

Artifacts over **10 MB** don't preview or render; they're offered as a download instead. Of the
binary kinds, only images preview inline.

## Publishing one

Ask the agent for the output you want and to publish it. It has built-in tools for the whole
lifecycle — creating, updating, organizing into folders, and setting sharing — so nothing needs
configuring first. Text content up to **2 MB** is sent inline; anything larger or binary is uploaded
directly, so big files never pass through the conversation.

When an agent publishes something, it links it in chat — opening that link previews the artifact
beside the conversation, so you can read a report or click through a dashboard without leaving the
session.

You can also upload a file yourself with **Upload artifact**.

Everything collects in the **Artifacts** destination, searchable by title and file name. Each
sandbox also shows the artifacts it produced.

An agent can only publish into its own owner's library, and attribution can't be spoofed.

## Versions

Updating an artifact's content publishes a **new version** rather than overwriting it. The artifact
keeps its identity: **the share link stays the same**, and viewers can switch between versions.

Editing only metadata — title, file name, folder — changes it in place without a new version.

## Folders

Group artifacts into folders to keep a library browsable. Folders are a **single flat level** — no
nesting. Deleting a folder does **not** delete the artifacts inside it; they become ungrouped.

## Sharing

Artifacts start **private**. Use **Sharing…** on an artifact to publish it; the link is served from a
separate share domain, so a shared artifact can never reach your DAM session.

!!! warning "The link is the access control"
    The share URL contains an unguessable random token, and **that token is the only thing
    protecting it**. Anyone who has the link can view the artifact — no sign-in, no allowlist, and
    **no password option**. Treat a public link as public: guarding it is up to you.

    Sending it to an external service publishes it. It may be cached or indexed even if you later
    make the artifact private.

Turning visibility back to private disables the link immediately.

Folders can be shared too — a folder page lists the artifacts inside it **that are themselves
shared**, so check what's public before sharing the folder.

Each artifact records **who published it** (which sandbox, or you if you uploaded it) and a **view
count** of public renders. The count is a bare number — no viewer identity or timestamps — and
in-app previews don't add to it.

!!! note "Shared HTML and JSX is live code"
    Rendered artifacts execute as authored — the platform doesn't sanitize HTML or JSX. That's what
    makes interactive dashboards work, and it's why you should only share content you trust.

## Expiry deletes the artifact

An expiry is **retention, not link lifetime**. This is the one thing worth reading twice:

!!! danger "An expired artifact is permanently deleted"
    Expiry applies to the artifact itself, **regardless of visibility** — a *private* artifact with
    an expiry is deleted too, and making a public artifact private again does **not** cancel its
    expiry. It is not just the share link switching off.

    Once the expiry passes, the link reports itself as expired and you can still renew it. After a
    **7-day grace period** the content is permanently deleted, every version with it.

The Sharing dialog offers **Never expires**, 1 hour, 1 day, 7 days, or 30 days. Leave it at *never*
to keep the artifact indefinitely; set one when you want the content gone on a schedule.

Deleting an artifact is also permanent — all versions go, and the share link dies with them.

## Related

Experiments publish their dashboards here — see [Experiments](experiments.md#dashboards).
