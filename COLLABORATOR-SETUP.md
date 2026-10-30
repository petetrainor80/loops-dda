# loops-dda: collaborator setup

**For the person reading this:** drop this file into Claude Code (desktop app or terminal) and say:

> Set me up on this project using this guide.

Claude does the work. You'll only need to:
1. Have a GitHub account and send Pete your GitHub username, so he can invite you to the repo.
2. Click "Authorize" in a browser window once, when Claude asks you to sign in to GitHub.
3. Possibly type your Mac password if Claude needs to install a tool.

You do **not** need a Vercel account.

---

## Instructions for Claude

You're setting up a collaborator on an existing project. Work through the steps in order and run the commands yourself. Stop and ask the user only where a step says so. Keep explanations short and non-technical: the user wants this to be frictionless.

### Project facts

| | |
|---|---|
| GitHub repo | https://github.com/petetrainor80/loops-dda (public) |
| Owner | Pete Trainor (GitHub: `petetrainor80`) |
| Stack | Next.js 16 (App Router), TypeScript, Tailwind CSS, npm |
| Hosting | Vercel project `loops-dda`, already connected to the GitHub repo |
| Live URLs | https://ddaagents.app and https://loops-dda.vercel.app |
| Deploys | Every push to `main` deploys to production automatically. Every push to any other branch gets its own preview URL. |

**Important:** don't install or run the Vercel CLI, and don't run `vercel`, `vercel link` or `vercel deploy`. Deployment happens entirely through GitHub pushes. Running Vercel commands would create a separate, disconnected Vercel project under the user's own account.

### Step 1: Check tools

Check for `git`, `node` (must be 20.9 or newer), `npm` and `gh` (GitHub CLI):

```bash
git --version; node --version; npm --version; gh --version
```

Install whatever is missing or too old.
- **macOS:** use Homebrew (`brew install git node gh`). If Homebrew itself is missing, ask the user to run the official one-line installer from https://brew.sh in their own terminal, because it needs their password.
- **Windows:** use `winget install --id Git.Git`, `winget install OpenJS.NodeJS.LTS` and `winget install GitHub.cli`.

### Step 2: Sign in to GitHub

Check with `gh auth status`. If the user isn't logged in, `gh auth login` is interactive, so ask the user to run this themselves. In the Claude Code terminal they can prefix it with `!`.

```bash
gh auth login --hostname github.com --git-protocol https --web
```

Tell them it shows a one-time code and opens a browser: they paste the code and click Authorize. Afterwards, run `gh auth setup-git` so `git push` uses the same login.

Get their username with `gh api user --jq .login`.

### Step 3: Accept the repo invitation

List pending invitations:

```bash
gh api user/repository_invitations --jq '.[] | "\(.id) \(.repository.full_name)"'
```

If `petetrainor80/loops-dda` is listed, accept it with its id:

```bash
gh api -X PATCH user/repository_invitations/<id>
```

If it's not listed, check whether they already have push access:

```bash
gh api repos/petetrainor80/loops-dda --jq .permissions.push
```

If the result isn't `true`, stop. Tell the user to send Pete their GitHub username (from step 2) and ask him to add them under **GitHub → loops-dda → Settings → Collaborators**. Then wait and re-check.

### Step 4: Clone and install

Ask the user where they keep projects. If they have no preference, use `~/Projects`. Then:

```bash
gh repo clone petetrainor80/loops-dda
cd loops-dda
npm ci
```

Use `npm ci`, not `npm install`, so they get the exact versions in `package-lock.json`.

### Step 5: Run it locally

Start the dev server in the background with `npm run dev`, then confirm http://localhost:3000 loads. Show the user the page if you have a browser available.

### Step 6: Make a first test push

This proves the whole path works: their machine, then GitHub, then Vercel.

```bash
git switch -c setup/<their-github-username>
git commit --allow-empty -m "Test push from <name>"
git push -u origin HEAD
```

A branch push creates a **preview** deploy and leaves the live site alone. Wait about a minute, then check the deploy:

```bash
gh api repos/petetrainor80/loops-dda/commits/$(git rev-parse HEAD)/status --jq '.statuses[] | "\(.context): \(.state) \(.target_url)"'
```

If the result is empty or `pending`, wait a bit and try again. A `Vercel: success` line means it worked: open the link to show them the preview. Then clean up:

```bash
git switch main
git push origin --delete setup/<their-github-username>
git branch -D setup/<their-github-username>
```

### Step 7: Tell the user how day-to-day work goes

Summarise this for them in plain language, then follow it yourself in every future session on this repo.

- **Before starting any work:** `git switch main && git pull --rebase`. Two people push to the same repo, so always start from the latest code.
- **Small changes:** commit to `main`, run `git pull --rebase`, then `git push`. This goes live on ddaagents.app within a minute or two.
- **Bigger or risky changes:** work on a branch and push it to get a preview URL to share. When it's approved, open a pull request with `gh pr create --fill` and merge it with `gh pr merge --squash --delete-branch`.
- **Before pushing:** run `npm run build` locally. If it fails here, it will fail on Vercel too.
- **If a push is rejected** because the remote has new commits: run `git pull --rebase`, fix any conflicts with the user, then push again. Never use `--force` on `main`.
- **Secrets and environment variables:** never commit `.env` files (they're gitignored). If the app needs an API key in production, ask Pete to add it in the Vercel project settings.
- **Checking a deploy:** use the `gh api .../commits/<sha>/status` command from step 6. Look for the `Vercel` line.

### Done when

- [ ] `gh auth status` shows the user logged in
- [ ] The repo is cloned and `npm run dev` serves http://localhost:3000
- [ ] A test branch push reported `Vercel: success` with a preview URL
- [ ] The test branch is deleted and the user knows the daily routine
