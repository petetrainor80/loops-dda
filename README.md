# DDA Agents (loops-dda)

The site for the IFS Loops DDA project, live at **[ddaagents.app](https://ddaagents.app)**.

| | |
|---|---|
| Live site | https://ddaagents.app (also https://loops-dda.vercel.app) |
| Hosting | Vercel project `loops-dda`, which deploys automatically from this repo |
| Stack | Next.js 16 (App Router), TypeScript, Tailwind CSS |
| Owner | Pete Trainor ([@petetrainor80](https://github.com/petetrainor80)) |

## How deploys work

There's no manual deploy step. Pushing to GitHub is the deploy.

- **Push to `main`:** the change goes live on ddaagents.app in a minute or two.
- **Push any other branch:** Vercel builds a private preview URL for that branch, and the live site is untouched. Use this to share work in progress.
- **Check a deploy:** look at the tick or cross next to the commit on GitHub, or run:
  ```bash
  gh api repos/petetrainor80/loops-dda/commits/$(git rev-parse HEAD)/status --jq '.statuses[] | "\(.context): \(.state) \(.target_url)"'
  ```

Don't run `vercel deploy` or `vercel link` from a collaborator's machine. It creates a separate, disconnected Vercel project.

## Working on it

```bash
git switch main && git pull --rebase   # always start from the latest code
npm ci                                  # install exact dependency versions
npm run dev                             # http://localhost:3000
```

Before pushing:

```bash
npm run build                           # if it fails here, it fails on Vercel
git pull --rebase && git push
```

Two people push to this repo, so:
- pull before you start and again before you push;
- use a branch and pull request for bigger or risky changes (`gh pr create --fill`);
- never force-push `main`.

## Project layout

```
src/app/          pages and layouts (App Router): page.tsx is the home page
src/app/globals.css   global styles and Tailwind
public/           static files served from the site root
vercel.json       pins the Vercel framework preset to Next.js
.devcontainer/    Codespaces setup: Node 24, GitHub CLI, Claude Code
```

## New collaborators

See [COLLABORATOR-SETUP.md](COLLABORATOR-SETUP.md). It's written so you can hand it to Claude Code and have it do the whole setup for you. You'll first need an invite to this repo from Pete.

No local installs or admin rights needed: either use [Claude Code on the web](https://claude.ai/code), or open a GitHub Codespace (**Code → Codespaces → Create codespace on main**). The Codespace comes with Node, the GitHub CLI and Claude Code pre-installed.

## Secrets

Never commit `.env` files (they're gitignored). Production environment variables live in the Vercel project settings. Ask Pete to add any that are needed.
