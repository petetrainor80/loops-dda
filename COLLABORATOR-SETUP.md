# loops-dda: collaborator setup

**For the person reading this:** you don't need to install anything or have admin rights on your computer. Everything runs in your web browser. Before you start:

1. Send Pete your GitHub username and accept his invite to the repo. It arrives by email, or appears at https://github.com/petetrainor80/loops-dda/invitations. If you don't have a GitHub account, create one free at github.com first.
2. Pick one of the two routes below. Route A is the simplest.

### Route A: Claude Code on the web (recommended)

1. Go to **https://claude.ai/code** and sign in.
2. Connect your GitHub account when asked, and choose the repository **petetrainor80/loops-dda**.
   - If the repo isn't in the list, ask Pete to install the Claude GitHub app on it. Repo owners have to approve that.
3. Start a session on that repo and paste:
   > Read COLLABORATOR-SETUP.md and get me set up.
4. From then on, just describe the changes you want in plain English.

### Route B: GitHub Codespaces (a full editor in your browser)

1. Open https://github.com/petetrainor80/loops-dda.
2. Click the green **Code** button, then the **Codespaces** tab, then **Create codespace on main**.
3. Wait a couple of minutes while it sets itself up. Everything is pre-installed, including Claude Code.
4. In the terminal panel at the bottom, type `claude`, press Enter and sign in to Claude. Then paste:
   > Read COLLABORATOR-SETUP.md and get me set up.

GitHub gives personal accounts about 60 free hours of Codespaces a month. Close the browser tab when you're done; it stops on its own after 30 minutes idle.

You do **not** need a Vercel account for either route.

---

## Instructions for Claude

You're helping a non-technical collaborator work on an existing project. Run the commands yourself, keep explanations short and plain, and ask the user only when a step says so.

### Project facts

| | |
|---|---|
| GitHub repo | https://github.com/petetrainor80/loops-dda (public) |
| Owner | Pete Trainor (GitHub: `petetrainor80`) |
| Stack | Next.js 16 (App Router), TypeScript, Tailwind CSS, npm |
| Hosting | Vercel project `loops-dda`, connected to this GitHub repo |
| Live site | https://ddaagents.app (currently a "Coming soon" holding page) |
| Deploys | A push or merge to `main` deploys to production automatically. A push to any other branch gets its own preview URL. |

**Rules:**
- **No Vercel commands.** Don't install or run the Vercel CLI (`vercel`, `vercel link`, `vercel deploy`). Deployment happens only through GitHub, and Vercel commands would create a separate, disconnected project.
- **Never force-push `main`,** and never rewrite history that's already on GitHub.
- **Never commit secrets or `.env` files.** If production needs a key, tell the user to ask Pete to add it in Vercel.

### Which environment am I in?

- **Codespace:** the `CODESPACES` environment variable is `true`. The repo is already cloned, dependencies are installed, and `git` and `gh` are already signed in as the user.
- **Claude Code on the web:** the repo is checked out in a cloud sandbox. Dependencies may not be installed yet, and `gh` may not be available or signed in.
- **Anything else:** the user is on their own machine. They'll need `git`, Node 20.9 or newer, and `gh` installed. If they can't install software (no admin rights), point them to Route A or B above instead of trying workarounds.

### First-time setup

1. **Check access.** Confirm the user can push. In a Codespace, run:
   ```bash
   gh api repos/petetrainor80/loops-dda --jq .permissions.push
   ```
   If the result isn't `true`, or the push in step 3 is rejected, the user hasn't accepted Pete's invite yet. Send them to https://github.com/petetrainor80/loops-dda/invitations, or tell them to give Pete their GitHub username.
2. **Install and check the build:**
   ```bash
   npm ci && npm run build
   ```
   In a Codespace, also start `npm run dev` in the background. Port 3000 opens as a preview tab, so show them the site running.
3. **Make a test push.** This proves the path from here to GitHub to Vercel works:
   ```bash
   git switch -c setup/test-push
   git commit --allow-empty -m "Test push"
   git push -u origin HEAD
   ```
   If your environment requires a particular branch name (Claude Code on the web often does), use that instead. After about a minute, check the deploy, using `gh` if available:
   ```bash
   gh api repos/petetrainor80/loops-dda/commits/$(git rev-parse HEAD)/status --jq '.statuses[] | "\(.context): \(.state) \(.target_url)"'
   ```
   Otherwise, tell the user to look for the green tick next to the branch's latest commit on GitHub; clicking it shows the Vercel preview link. A `Vercel: success` result means everything works. Then delete the test branch:
   ```bash
   git push origin --delete setup/test-push
   ```
4. **Explain the routine below** in two or three plain sentences.

### Day-to-day routine (follow this in every session)

1. **Start from the latest code.** Run `git fetch origin && git switch main && git pull --rebase`, or start a new branch from an up-to-date `origin/main`. Two people work on this repo.
2. **Work on a branch,** not directly on `main`. Give it a short descriptive name such as `add-contact-page`.
3. **Run `npm run build` before pushing.** If it fails here, it fails on Vercel too.
4. **Push the branch** and share the Vercel preview link with the user. It's in the commit status, or under the green tick on GitHub.
5. **Open a pull request** when the user is happy: `gh pr create --fill`, or the "Create PR" button in Claude Code on the web. Then tell the user that merging it on GitHub puts the change live on ddaagents.app within a minute or two.
6. **If a push or merge conflicts** with Pete's changes, rebase onto the latest `origin/main`, resolve the conflict carefully, and explain to the user in plain words what changed.
