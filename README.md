# skills

Agent skills that take a Jira ticket from To Do to a reviewed pull request.

## Install

```
npx skills@latest add talibilat/skills -g
```

Add `--all` to skip the prompts, then restart your agent.

## Skills

| Skill | Short name | What it does |
|---|---|---|
| `talib-ticket-plan` | `ttp` | With no ticket key, asks for epic, work type, priority and order, then picks the best To Do ticket. With a key (`ttp PROJ-123`) it asks nothing and starts the work. It claims the ticket, renames the chat to the ticket key, makes its own git worktrees, reproduces bugs and writes a checked plan with a plain-language brief. |
| `talib-ticket-ship` | `tts` | Builds the plan with parallel sub-agents, then needs a passing live test (browser or real run, or steps for you). Four reviewers give a score out of 100, with fix rounds that stop and diagnose after two failures. Then it opens the PR, comments in Jira, archives the worktree and, once the PR is good, moves the ticket to Code Review and logs time. |
| `talib-bug-reproduced` | | For a bug that came back. Searches Jira, PR and git history, reproduces it on the default branch, says why it returned, and plans from the earlier work. |

Nothing passes without a live test. Whatever you type after a skill name outranks the skill's own rules.

## Setup

On first run the skill creates `setup.md` next to its conventions file. It fills in your Jira site, account id and repos from your tools, and asks for the rest. That file stays on your machine.

You need an Atlassian Jira connector in your agent, the `gh` CLI logged in, and git.

## Optional companion skills

The reviewers and checks call these skills if you have them: `interrogate` (pstack), `postwork`, `ponytail-review`, `thermo-nuclear-code-quality-review`, `ticket-prework-review`. A missing reviewer shows as NOT RUN and blocks a score of 100 until you waive it.

Built for Claude Code. Codex, Cursor and other agents work too, with the tool swaps listed in `skills/talib-ticket-plan/references/conventions.md`.

## License

MIT
