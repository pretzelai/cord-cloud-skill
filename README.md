# Cord Push/Pull Skill

Hand off coding work between your local agent and [Cord](https://runcord.com) cloud sessions.

## What is this?

This is a **skill** — an instruction set that any coding agent (Claude Code, Cursor, Codex, Aider, Amp, etc.) can follow to seamlessly move work between your local environment and Cord's cloud coding agents.

**Push:** You're done for the day. Your local agent commits, pushes, writes a handoff, and sends it to Cord. Cord picks up where you left off.

**Pull:** Next morning. Your local agent pulls Cord's work — a handoff summary of what was done, what's next, and any gotchas. Then `git pull` to get the commits.

## Installation

### Prerequisites

```bash
npm install -g cord-cli
cord auth login
```

Or use without installing: `npx cord-cli auth login`

### Install the skill

**Recommended — via the skills CLI:**

```bash
npx skills add pretzelai/cord-cloud-skill -y
```

This installs to your agent's skills directory (e.g. `.agents/skills/` for Cursor and OpenCode).

**Manual — copy into your project:**

| Agent | Where to put it |
|-------|----------------|
| Cursor / OpenCode | `.agents/skills/cord-push-pull/SKILL.md` |
| Claude Code | `.claude/skills/cord-push-pull/SKILL.md` |
| Any agent | Reference the raw URL below in project instructions |

Raw skill URL:
```
https://raw.githubusercontent.com/pretzelai/cord-cloud-skill/main/SKILL.md
```

## Quick Usage

Tell your local agent:

> "Push this work to Cord and let it continue overnight"

or

> "Pull the latest from Cord — what did it do?"

The agent follows the skill instructions to handle git, write context, and call the CLI.

## How it works

```
Local Agent                    CLI (transport)              Cord Cloud
───────────                    ─────────────               ──────────
Commit + push branch    →
Write handoff context   →      cord push          →       Creates session
                                                          Checks out your branch
                                                          Starts working

                               cord pull          ←       Commits + pushes
Read handoff context    ←                         ←       Writes handoff
git pull               ←
Continue working
```

The CLI is a thin pipe. Your agent does all the intelligent work (preparing context, managing git, deciding what to hand off). Cord's agent does the same on the other side.

## License

MIT
