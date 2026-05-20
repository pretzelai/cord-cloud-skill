# Cord Push/Pull Skill

Hand off coding work between your local agent and [Cord](https://runcord.com) cloud sessions.

## What is this?

This is a **skill** — an instruction set that any coding agent (Claude Code, Cursor, Codex, Aider, Amp, etc.) can follow to seamlessly move work between your local environment and Cord's cloud coding agents.

**Push:** You're done for the day. Your local agent commits, pushes, writes a handoff, and sends it to Cord. Cord picks up where you left off.

**Pull:** Next morning. Your local agent pulls Cord's work — a handoff summary of what was done, what's next, and any gotchas. Then `git pull` to get the commits.

## Installation

### Prerequisites

```bash
npm install -g @anthropic/cord-cli
cord auth login
```

### Add to your agent

Copy the contents of [`SKILL.md`](./SKILL.md) into your agent's instruction source:

| Agent | Where to put it |
|-------|----------------|
| Claude Code | `.claude/commands/cord-push-pull.md` or reference in `CLAUDE.md` |
| Cursor | `.cursorrules` or a custom command |
| Codex | `AGENTS.md` or instructions file |
| Aider | `.aider.conf.yml` conventions or chat reference |
| Any agent | Paste into system prompt or project instructions |

Or point your agent to the raw URL:
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
