---
name: talib-bug-reproduced
description: Plan a Jira bug that came back after earlier work (a reopened ticket, a new ticket for the same problem, or a regression of a past fix). Searches Jira, PR, git and archive history, reproduces the bug on current dev, says why it came back, and writes a plan that builds on the old work. Use when the user says a bug is back, reproduced, reopened or a repeat, or when talib-ticket-plan finds earlier work on a ticket. Hands off to talib-ticket-ship.
---

# talib-bug-reproduced

Same flow as `talib-ticket-plan`, plus a history check and a different plan. Read these first:

1. `~/.agents/skills/talib-ticket-plan/references/conventions.md`
2. `~/.agents/skills/talib-ticket-plan/SKILL.md`, the steps you follow below
3. `~/.agents/skills/talib-ticket-plan/references/plan-template.md`

If you are not Claude Code inside T3, read the "Using these skills from another agent" table in that file first and swap the tools as it says.

Input: the new ticket key, and optionally the old key. With a key, run in autopilot: ask nothing and follow the Autopilot section of the conventions file. The user's message with the call outranks everything here. With no key, start at step 1 of `talib-ticket-plan` and let the user pick.

## Steps

Run these in order. Steps marked "as in plan" are the same steps as in `talib-ticket-plan`.

1. **Free check and claim.** As in plan, steps 2 and 3 (second fetch, T3 chat search, claim, confirm, rename the chat to the key). The "earlier work" hit from step 2.3 is expected here, so skip the Repeat bug card.
2. **Ticket file.** As in plan, step 4.1.
3. **History search.** See below. Write `.scratch/<KEY>-history.md`.
4. **Verdict.** Pick one and say why in 3 lines or fewer:
   - old fix incomplete
   - a later change broke it
   - different root cause
   - never really fixed
5. **Repos and worktrees.** As in plan, steps 4.2 and 5, with this rule:
   - If the old PR is still open, reuse the old branch. Recreate its worktree from the archive with `git worktree add <path> <old branch>`, then `git fetch` and rebase onto `origin/dev` only after asking.
   - If the old PR is merged or closed, make a new branch from fresh `origin/dev`, named for the new key.
6. **Baseline.** As in plan, step 6.
7. **Reproduce on current `dev`.** As in plan, step 7, with these changes. The bug must reproduce on current `dev` before any plan is written. If an old regression test exists, run it first and say why it did not catch this case. The new repro test extends the old one where it can. If the bug does not reproduce, show the Repro stuck card.
8. **Plan.** As in plan, step 8, with `Repeat bug: yes` and these two sections filled in:
   - **Previous attempt.** Old ticket, PR, files changed, what the fix did, and how it was tested.
   - **Why it came back.** The verdict, and the evidence.
9. **Jira link.** If the new and old tickets have no link, add "Relates" with `createIssueLink`. Use `getIssueLinkTypes` for the exact name.
10. **Show it.** As in plan, step 9, with one extra line at the top of the brief: "This is a repeat of <old key>."
11. **Hand off.** As in plan, step 10. When calling `talib-ticket-ship`, say it is a repeat bug and give the old key and PR number. Ship then adds the "Earlier work" line to the Jira comment and the PR body.

## History search

Jira, done in the main chat:

- The old ticket: description, comments, fix version, worklogs, who worked on it.
- Issue links on both tickets (relates, clones, blocks, duplicates).
- Tickets in the same epic, with the same labels, or with a similar summary (`text ~ "<key words>"`), resolved in the last 12 months.

GitHub, git and local files, done by one read-only child (Cursor first, Sonnet fallback, see the conventions file). Tell it to read only, write `.scratch/<KEY>-history-repos.md` and reply in 40 lines or fewer:

- PRs that mention the old key, from `gh pr list -R <org>/<repo> --search "<old key>" --state all`
- For each, the changed files, the review comments and any failed checks
- Other PRs that changed the same files since the old fix
- `git log --oneline` on the files the new ticket touches, since the old fix merged, to spot commits that could have undone it
- `<root>/archived-worktrees/<old key>/archive.md` and any `.scratch/<old key>-*.md` files

Merge both parts into `.scratch/<KEY>-history.md`. The "Previous attempt" section of the plan summarizes it: old ticket, what was fixed, which PR, which files, and why it may have come back.

## Never

- Never write the plan before the bug reproduces on current `dev`.
- Never reuse an old branch whose PR is merged.
- Never push in this skill.
