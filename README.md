# Proyekto

A project board that pairs a Kanban workflow with a Gantt-style timeline. The app uses React, TypeScript, a Cloudflare Worker, and D1. When Google sign-in is not configured, it opens in a clearly labeled local demo mode.

## AI agent skill

Use the portable [Proyekto Kanban skill](SKILL.md) to guide Codex, Claude Code, and other AI agents when they read or update a project's `KANBAN.md`. It covers the managed markers, task fields, resources and roles, safe editing, and GitHub sync behavior.

The source repository is private. To share the skill publicly without exposing the application source, publish `SKILL.md` in a separate public repository or a public GitHub Gist. Making this repository public would expose all files in it.

## Current deployment

The Worker is live at [proyekto-kanban.ronartos.workers.dev](https://proyekto-kanban.ronartos.workers.dev). Google sign-in, project access approval, D1 task storage, GitHub synchronization, and private R2 task attachments are configured.

## Run locally

```sh
npm install
npm run dev
```

If you only need to preview the browser UI without starting the local Worker runtime, use `npm run dev:ui`.

Create a production bundle with `npm run build`. Deploying requires a Cloudflare account and Wrangler authentication.

## Demo workspace

The first milestone includes two sample projects. Open the Proyekto logo menu to switch between isolated task lists. Add a task with the plus button in To Do, drag cards between status columns, or use the Move to control for keyboard access. Click a card to edit details, dates, progress, and assignee; saving adds a dated activity entry. Timeline bars follow each task's start date, due date, and status color.

Demo changes are saved in this browser's local storage, scoped by project. Configured accounts instead load their project tasks from D1.

## Cloudflare setup

The project uses the Cloudflare Vite plugin and Worker static assets. Before deploying:

1. Create a D1 database with `npx wrangler d1 create proyekto-db`.
2. Replace the all-zero `database_id` in `wrangler.jsonc` with the ID Wrangler returns.
3. Apply the schema locally with `npx wrangler d1 migrations apply proyekto-db --local`, then remotely with `npx wrangler d1 migrations apply proyekto-db --remote`. The migrations create the initial app schema and add GitHub sync state.
4. Enable R2 in the Cloudflare dashboard, then create the private bucket with `npx wrangler r2 bucket create proyekto-attachments`. The app uses this bucket for private task attachments; it does not expose a public bucket URL.
5. Create a Google OAuth web client. Set its authorized JavaScript origin to the app's domain. Set `GOOGLE_CLIENT_ID` as a Worker variable (and in `.dev.vars` for local auth).
6. Set Worker secrets `SESSION_SECRET` (a random value of at least 32 characters) and `BOOTSTRAP_ADMIN_EMAILS` (comma-separated verified Google account emails). The bootstrap account becomes the initial platform admin.
7. Create a GitHub App for Proyekto with **Contents: read and write** permission. Set its setup callback URL to `https://<your-worker-domain>/api/github/callback`, its webhook URL to `https://<your-worker-domain>/api/github/webhook`, a webhook secret, and subscribe to **Push** events. Install it only on repositories that need project sync. Configure `GITHUB_APP_ID`, `GITHUB_APP_SLUG`, and the private key as Worker settings (private key and webhook secret as secrets).
8. For local auth and GitHub sync, copy `.dev.vars.example` to `.dev.vars` and replace the placeholders. Do not commit `.dev.vars`.

The first platform admin can create projects, approve or decline project access requests, and assign editor or viewer roles. Project admins can review requests for their own project. Google sign-in establishes identity; D1 membership checks authorize each project request.

## GitHub task publishing

Set a project's private repository in `owner/repository` form when creating it. A project admin can connect the GitHub App from **GitHub sync**. For a new empty project with a compatible managed section already in `KANBAN.md`, choose **Import tasks** first. After that, task creates and edits publish the task snapshot to the marked section; local pushes to the configured branch import task edits into D1. Text outside the managed section is preserved. See [the KANBAN.md format](docs/KANBAN_FORMAT.md).

If GitHub changes since the last sync, the app flags a conflict instead of overwriting the newer file. A failed publish leaves D1 task changes saved and can be retried with **Sync now**. Removing tasks from the managed section is flagged for review rather than deleting D1 data. GitHub Contents API SHA checks also prevent overwriting a file changed during a publish.

R2 attachment usage is governed by Cloudflare's current account limits and storage pricing. Project admins can edit project details; changing the GitHub repository, branch, or KANBAN path clears the old connection so it can be safely reconnected.
