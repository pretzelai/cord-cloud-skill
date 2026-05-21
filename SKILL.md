---
name: cord-push-pull
description: Hand off coding work between local agent and Cord cloud sessions using the cord CLI. Use when the user asks to push work to Cord, pull work from Cord, check Cord session status, or hand off to cloud.
---

# Cord Push/Pull Skill

You are a coding agent that can hand off work to and from Cord cloud sessions using the `cord` CLI. Follow these instructions when the user asks you to push work to Cord or pull work from Cord.

---

## When to use this skill

- User says: "push this to Cord", "let Cord continue", "hand off to cloud", "send to Cord"
- User says: "pull from Cord", "what did Cord do", "get the cloud work", "pick up from Cord"
- User says: "check Cord status", "is Cord done", "what's Cord doing"

---

## Prerequisites

Before running any cord commands, verify the setup:

### 1. Check if cord CLI is installed

```bash
cord --version
```

If not installed, tell the user:
> The cord CLI is not installed. Install it with: `npm install -g cord-cli` (or run commands via `npx cord-cli`)

Do NOT install it yourself without user permission.

### 2. Check if authenticated

```bash
cord auth status
```

If not authenticated, run:
```bash
cord auth login
```

This opens a browser for the user to log in (or sign up if they don't have an account). Wait for it to complete before proceeding.

---

## Push: Send work to Cord

Use this when the user wants Cord to continue their work in the cloud.

### Step 1: Ensure the branch is safe to hand off and git state is clean

Check that the working tree is ready to hand off:

```bash
git status
```

**Never push the base branch.** Do not assume it is named `main` or `master`; determine it first:

```bash
git symbolic-ref --quiet --short refs/remotes/origin/HEAD
```

This prints something like `origin/main` or `origin/staging`. If that fails, inspect the repository's default/base branch with `git remote show origin` or the hosting provider. If the current branch is the base branch, create a descriptive feature branch first:

```bash
git checkout -b <descriptive-branch-name>
```

**If there are uncommitted changes:** Stage and commit them.
```bash
git add -A
git commit -m "<descriptive message of current state>"
```

**Push to remote:**
```bash
git push -u origin HEAD
```

This is critical — Cord needs the branch to exist on GitHub to clone it.

### Step 2: Prepare the handoff context

Prepare structured handoff context. This is the most important part — the quality of this context determines how well Cord can continue your work.

Before pushing, make sure there is obvious work left for Cord to do. If the task looks complete or the next step is ambiguous, stop and ask the user what Cord should do next instead of pushing a vague handoff.

Use this structure:

```markdown
## What has been accomplished

<Concise summary of work done so far. Be specific — mention file paths, function names, architectural decisions.>

## Current state

<What state is the code in right now? Does it compile? Are tests passing? Any known broken things?>

## What to do next

<Specific, actionable instructions. NOT vague like "continue working". Instead: "Implement the pagination logic in src/components/UserList.tsx — the API already returns cursor tokens in the response, wire them up to the existing useInfiniteQuery hook.">

## Key decisions and context

<Important architectural decisions, constraints, or context that aren't obvious from the code alone. E.g., "We chose server components here because the data is user-specific and can't be cached at the edge.">

## Gotchas and things to be careful about

<Anything that might trip up the next agent. E.g., "The test suite uses a custom matcher in test/helpers.ts — don't use standard jest matchers for API responses.">
```

**Guidelines for good handoff context:**
- Be specific, not vague. File paths > descriptions.
- Include the "why" behind decisions, not just the "what".
- Mention things that aren't obvious from reading the code.
- If there's a plan document or spec, reference it.
- Keep it under 5000 words — concise beats comprehensive.
- The "What to do next" section must contain concrete unfinished work. If you cannot write it clearly, ask the user.

### Step 3: Push to Cord

Prefer piping the handoff directly so you do not create extra files:

```bash
cat <<'EOF' | cord push --message "<short concrete next step>"
<handoff context>
EOF
```

The CLI automatically detects your current branch and repository from git.

**Optional flags:**
- `--message "Short instruction"` — Add a brief directive on top of the full context
- `--branch <name>` — Override branch detection
- `--repository <owner/repo>` — Override repository detection
- `--session <id>` — Resume a previously suspended session instead of creating a new one
- `--model <provider/model>` — Request a specific model (e.g., `anthropic/claude-sonnet-4`)

**Alternative: pipe context directly:**
```bash
echo "Implement the auth flow as described in PLAN.md" | cord push
```

**Alternative: inline context:**
```bash
cord push --context "Finish implementing the test suite for the API routes. All route handlers are done, just need tests."
```

**Use a temp context file only if the handoff is too large or awkward to pipe.** If you do, delete it after the push.

```bash
cord push --context-file /tmp/cord-handoff.md --message "<short concrete next step>"
```

### Step 4: Confirm success

The CLI outputs:
- Session URL (for the user to monitor in browser)
- Session ID (for later pull/status)
- Branch and repository info

Tell the user the push was successful and give them the session URL.

### After pushing

If you created a temp handoff file, clean it up:
```bash
rm /tmp/cord-handoff.md
```

---

## Pull: Get work from Cord

Use this when the user wants to retrieve what Cord has done and continue locally.

### Step 1: Pull from Cord

From the same repository directory:

```bash
cord pull
```

The CLI automatically finds the Cord session linked to your current git branch.

**Optional flags:**
- `--session <id>` — Pull from a specific session instead of auto-detecting
- `--branch <name>` — Look up session by a different branch name
- `--output <path>` — Write handoff context to a file instead of stdout

### Step 2: Handle the response

**If the agent is still working:**

The CLI will output something like:
```
Agent is still working
The Cord agent is currently active on this session.
Check progress: https://app.runcord.com/<repo-id>/<session-id>
Run `cord pull` again later or use `cord status` to check.
```

Tell the user that Cord is still working and give them the URL. Do not proceed with git pull.

**If the pull succeeds (agent is idle/done):**

The CLI outputs the handoff context — read it carefully. Then sync git:

```bash
git pull origin HEAD
```

This gets all the commits Cord made. Review what changed:

```bash
git log --oneline -10
```

### Step 3: Continue working

Now you have:
1. The handoff context from Cord (what was done, what's next, gotchas)
2. The actual code changes (from git pull)

Read the handoff context carefully and continue working from where Cord left off. Pay special attention to:
- The "What to do next" section
- The "Gotchas" section
- Any mentioned files or architectural decisions

---

## Status: Check what Cord is doing

Quick check without pulling:

```bash
cord status
```

This shows:
- Whether the session is running, idle, or suspended
- The session URL
- The linked branch

Use this when the user just wants to peek at progress without pulling.

---

## Error handling

### "No Cord session found for this branch"

The current branch has no linked Cord session. Either:
- Push hasn't been done yet from this branch
- The session was on a different branch

Solutions:
- Use `cord pull --session <id>` with a known session ID
- Use `cord push` to create a new session first

### "Repository not accessible"

Cord's GitHub App isn't installed on this repo. Tell the user:
> Cord needs access to this repository. Install the GitHub App at: https://github.com/apps/cord-ai/installations/new

### "Could not infer a GitHub repository from this directory"

Not in a git repo, or the remote doesn't point to GitHub. Use explicit flags:
```bash
cord push --repository owner/repo-name
```

### Authentication failures

Re-authenticate:
```bash
cord auth login
```

---

## Round-trip example

A typical multi-day workflow:

```
Day 1 (local, evening):
  User: "Push this to Cord, let it finish the API routes overnight"
  Agent: commits, pushes branch, writes handoff, runs `cord push`

Day 2 (morning):
  User: "Pull from Cord, what did it do?"
  Agent: runs `cord pull`, reads context, runs `git pull`, continues

Day 2 (evening):
  User: "Push again, have Cord write the tests"
  Agent: commits, pushes, writes new handoff, runs `cord push --session <id>`
```

---

## Important notes

- **Always push your branch before calling `cord push`.** Cord clones from GitHub — uncommitted or unpushed work will be lost.
- **Always `git pull` after a successful `cord pull`.** The handoff context tells you what changed, but you need the actual commits too.
- **Don't interrupt a running session.** If `cord pull` says the agent is working, let it finish or check the web UI.
- **Branch is the link.** Cord finds your session by matching the current git branch. Stay on the same branch for push/pull continuity.
- **Context quality matters.** A vague handoff like "finish the feature" produces worse results than specific instructions with file paths and architectural context.
