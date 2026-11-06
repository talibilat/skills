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
| `ticket-plan` | `ttp` | With no ticket key, asks for epic, work type, priority and order, then picks the best To Do ticket. With a key or Jira link (`ttp PROJ-123`) it asks nothing, starts the work and carries on into `ticket-ship`. It traces the ticket to its original requirement documents and keeps a project knowledge file up to date. It claims the ticket, renames the chat to the ticket key, makes its own git worktrees, reproduces bugs and writes a checked plan with a plain-language brief. |
| `ticket-ship` | `tts` | Works alone: give it a key, or plain instructions, and it plans first if no plan exists. Builds the plan with parallel sub-agents, then needs a passing live test (browser or real run, or steps for you). Four code reviewers and then an `aqa` QA review give a score out of 100, with up to 5 fix rounds on its own, stopping when the loop is stuck. Then it opens the PR with screenshot or API proof, comments in Jira, archives the worktree and, once the PR is good, moves the ticket to Code Review and logs time. It keeps watching until the PR is merged, fixing CI and resolving every review comment. |
| `bug-reproduced` | | For a bug that came back. Searches Jira, PR and git history, reproduces it on the default branch, says why it returned, and plans from the earlier work. |
| `aqa` | | Senior QA validation of a ticket or bug fix. Finds the parent chain, checks every acceptance criterion, reviews code and logs, runs positive, negative and edge scenarios live, and gives a verdict: Done, Done with notes, Stays open, or Needs BA input. `ticket-ship` runs it after code review. |
| `plain-talk` | | Explains any ticket, bug, PR or result in everyday language, as a story that opens with the name of the problem, told over a few turns with a check-in between each so you can follow along. Several tickets become one story. Name a skill or concept after it (`plain-talk ticket-ship`) and it explains it with a real-life example. |

Nothing passes without a live test, and every PR carries proof that it works locally. Whatever you type after a skill name outranks the skill's own rules.

## Setup

On first run the skill creates `setup.md` next to its conventions file. It fills in your Jira site, account id and repos from your tools, and asks for the rest. That file stays on your machine.

You need an Atlassian Jira connector in your agent, the `gh` CLI logged in, and git.

## Optional companion skills

The reviewers and checks call these skills if you have them: `interrogate` (pstack), `postwork`, `ponytail-review`, `thermo-nuclear-code-quality-review`, `ticket-prework-review`. A missing reviewer shows as NOT RUN and blocks a score of 100 until you waive it.

Built for Claude Code. Codex, Cursor and other agents work too, with the tool swaps listed in `skills/ticket-plan/references/conventions.md`.

## License

MIT
