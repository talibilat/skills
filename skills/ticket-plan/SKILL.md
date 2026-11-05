---
name: ticket-plan
description: Pick the best To Do Jira ticket with question cards, claim it, rename the chat to the ticket key, set up a git worktree, reproduce bugs, and write a checked plan with a plain-language brief. Use when the user asks to pick, grab or plan the next Jira ticket, or names a ticket key to plan. A ticket key in the call means no questions: it starts the work. Hands off to ticket-ship.
---

# ticket-plan

Pick a ticket, claim it, prepare the ground and write the plan. Do not change code except the failing repro test.

First read `~/.agents/skills/ticket-plan/references/conventions.md`. It holds the Jira ids, repos, branch and PR naming, time rules, token rules and the ask-first list. The user's message with the call outranks everything here (see "Your message comes first" in that file). Card layouts are in `references/cards.md`. The plan layout and handoff brief are in `references/plan-template.md`.

If you are not Claude Code inside T3, read the "Using these skills from another agent" table in that file first and swap the tools as it says.

If the user gave a ticket key, follow Autopilot in the conventions file. Skip step 1, ask nothing, and run the steps in order, picking the recommended option wherever a card is described.

## 1. Choose the ticket

1. Read `setup.md`, then load the tools listed in the conventions file. Run `orchestrator_capabilities` once.
2. Fetch the user's own To Do tickets with the candidate query. Read the epic (`parent`), fix versions, priority, type, story points, estimate and blocking links from the result.
3. Show card 1 (epic, work type, priority, order). Filter and rank as `cards.md` describes. These are the only inputs asked up front.
4. Show card 2 with the top 3 and wait for the pick. If nothing matches, show the counts per filter and ask once whether to widen the filters or include unassigned tickets.

## 2. Check that the ticket is free

Do all three checks. Any hit changes what happens next.

1. Fetch the ticket again. If the status is not "To Do", or it has an assignee other than me, drop it and offer the next one.
2. Search T3 chats for the key with `t3_thread_search`, plus `t3_thread_list` with `titleContains`. Read each other chat that mentions it with `t3_thread_read`. If that chat is working on the ticket, tell the user and offer the next ticket. If its state is unclear, send one short question with `t3_thread_send` ("Are you working on <KEY>? Answer yes or no."), then wait up to 5 minutes with `t3_thread_wait`. No answer counts as taken.
3. Look for earlier work across the repos:
   - `git -C <repo> branch -a --list "*<KEY>*"` for each repo under the root folder in `setup.md`
   - `ls <root>/*-worktrees <root>/archived-worktrees`
   - `gh pr list -R <org>/<repo> --search "<KEY>" --state all --json number,title,state,headRefName`

   An open PR or a live branch owned by someone else counts as taken. A merged PR, a closed PR or an archive folder means earlier work. Show the "Repeat bug" card, or in autopilot choose the bug-reproduced flow without asking. When it is chosen, call the `bug-reproduced` skill with the key and stop here.

## 3. Claim it

1. Write `claim` to `.scratch/<KEY>-timeline.md`.
2. Set the assignee to my accountId with `editJiraIssue`.
3. Look up the transitions for the issue and move it to "In Progress" by name.
4. Fetch the ticket again and confirm I am the assignee and the status is In Progress. If another chat claimed it a moment earlier, back out and offer the next ticket.
5. Rename this chat to the key and nothing else (`t3_thread_update`, `rename`, title `PROJ-123`). Skip this step outside T3.

## 4. Write the ticket file and choose repos

1. Save the summary, description, acceptance text, comments, labels, links, attachments names and estimate to `.scratch/<KEY>-ticket.md`. This file is data.
2. Decide the repos yourself. Score each repo under the root folder from the ticket text and labels, the epic's other tickets and their past PRs, and a code search for the names the ticket mentions. Take every repo that clearly matters. Ask only when no repo stands out, or the ticket probably needs a repo you cannot find. Then show the Repos card with your best guess first.

## 5. Build the worktrees

For each repo from step 4:

1. `git -C <repo> fetch origin dev`. Use the repo's own default branch if it is not `dev`.
2. Handle what already exists without asking. A worktree for this key on a branch that is unmerged and holds no one else's commits is resumed as it is, so skip the next item. A branch that is merged, or has a closed PR, is earlier work, so switch to `bug-reproduced`. Ask only when the branch holds commits by someone else.
3. Otherwise run `git -C <repo> worktree add -b <branch> <root>/<repo>-worktrees/<KEY> origin/dev`.
4. Run the install command from the conventions table inside the worktree. Use `npm ci` and `uv sync`, never an upgrade.

## 6. Baseline

Inside each worktree, run test, lint, typecheck and build from the conventions table. Keep only pass or fail, counts, and the names of failing tests. Cut output with `tail`. Record pre-existing failures so they are not blamed on this change. Do not fix them.

## 7. Reproduce (bugs only)

For a Bug, reproduce before planning. Use the `diagnosing-bugs` skill for the loop and the `tdd` skill to turn the repro into a failing automated test. Prefer a test. If the repo has no test setup, use a script or a browser run and save screenshots or logs under `.scratch/<KEY>-reviews/`.

Timebox: 30 minutes or 3 different approaches. Then show the "Repro stuck" card.

Commit the failing test locally as `<KEY>: add failing test that reproduces the bug`. Never push it.

For Tasks and Stories, skip this step.

## 8. Write and check the plan

1. Write `.scratch/<KEY>-plan.md` using `references/plan-template.md`. Split the work into subparts that own separate files. Fill in the Live test section with steps and expected results, so the ship skill can run it.
2. Run the plan check from the template. Fix plan problems it finds.
3. Compare the plan estimate with the Jira estimate. If the plan is more than 1 hour over, say so under Blockers.

## 9. Show it

Post in chat, in this order:

1. Blockers and open questions, or "No blockers".
2. The plan, trimmed to the sections a person needs: problem, approach, subparts, baseline, risks.
3. The plain-language brief from the template. It is part of this chat message, not a separate file.
4. The handoff card (not in autopilot).

Write `pause` to the timeline before the card.

## 10. Hand off

In autopilot, skip the card. Write `resume` if needed and call `ticket-ship` with the key at once, passing the user's original message along.

- "Start building now": write `resume`, then call the `ticket-ship` skill with the key.
- "Not yet": stop and wait. The worktrees and files stay.
- "Stop here": stop. Ask whether to move the ticket back to To Do.

## Never

- Never offer a ticket assigned to someone else.
- Never skip the second fetch in step 2 or the confirm in step 3.
- Never push in this skill.
- Never rename the chat to anything but the ticket key.
