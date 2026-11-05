---
name: talib-ticket-ship
description: Build a planned Jira ticket with parallel Sonnet sub-agents in its worktree, re-check the bug, run four reviewers (pstack interrogate, postwork, ponytail-review, thermo-nuclear) and score the result out of 100, require a passing live test (browser or real run, or steps for the user), loop on fixes, then raise the PR, comment in Jira, archive the worktree, wait for the PR to be good and move the ticket to Code Review with a worklog. Use after talib-ticket-plan, or when the user asks to implement or ship a ticket that already has a plan in .scratch.
---

# talib-ticket-ship

Turn an approved plan into a merged-quality PR. Work only inside the ticket's worktree. The user's message with the call outranks everything here (see "Your message comes first" in the conventions file).

First read `~/.agents/skills/talib-ticket-plan/references/conventions.md`. Read `references/live-test.md` before step 4, `references/review-loop.md` before step 5 and `references/closeout.md` before step 6.

If you are not Claude Code inside T3, read the "Using these skills from another agent" table in that file first and swap the tools as it says.

## 0. Recap and go

Input is a ticket key. With no key, list `.scratch/*-plan.md` and ask which one.

Read `<KEY>-ticket.md`, `<KEY>-plan.md` and `<KEY>-timeline.md`. Confirm the worktree exists. If it was archived, recreate it from the branch with `git worktree add <path> <branch>`.

- Run straight after `talib-ticket-plan`: give a recap of about 5 lines.
- Run on its own: explain the plan in plain words (what changes, why, how we check it) and add one Mermaid diagram.

Show a "Go (Recommended)", "Adjust the plan", "Stop" card only when this skill runs on its own with no key and no instructions. Write `pause` before it and `resume` after it. Run from `talib-ticket-plan`, or with a key, start building at once.

## 1. Build

1. Read the Subparts table. Order the subparts by dependency into batches. Subparts in one batch run in parallel, at most 3 at a time.
2. Start one builder per subpart with `delegate_task` (role `implementation`, Sonnet medium, shared checkout). The brief is short and uses paths:
   - the worktree path and the plan path
   - the subpart id, the files it owns, and the test to pass
   - the do-not-touch list
   - "Edit only your files. Do not commit or push. Do not run installs. Report the files you changed and the test result in under 15 lines."
3. End the turn. The completions wake the chat.
4. After each batch, in the main chat:
   - `git -C <wt> status --porcelain`. Every changed file must belong to a subpart in this batch. Revert or ask about anything else.
   - Run the tests the subparts name.
   - Commit one subpart at a time with the identity from the conventions file. Message: `<KEY>: <what changed>`.
5. Check the overrun rule at every batch boundary. Replace a failed builder once, then ask.

## 2. Re-check the bug

For a Bug, run the repro test or script again. It must now pass. If it still fails, the build is not done. Go back to the plan, not to a guess.

## 3. Prove nothing else broke

Run the baseline commands again. Every command must do at least as well as the baseline, and no new failures may appear. Then check scope:

- `git -C <wt> diff --name-only origin/dev...HEAD` must list only files the plan lists, plus the repro test.
- Any extra file needs a written reason in the plan or gets reverted.
- Lockfiles, dependency files, CI config and anything on the do-not-touch list stay unchanged unless a subpart lists them.

## 4. Live test

Nothing passes without a live test. Follow `references/live-test.md`. Run the changed behavior for real in a browser or a running service and record steps, expected results and actual results in `.scratch/<KEY>-live-test.md`. When the agent cannot run it, give the user numbered steps with the expected result and wait. Unit tests, lint and builds never count. A FAIL goes back into the phase 2 plan. The verdict is valid for one commit only.

## 5. Review loop

Follow `references/review-loop.md`. Four reviewers, one combined score out of 100, a phase 2 plan for every failed round, and these hard stops:

- Round 2 still failing: write the diagnosis, correct the plan, then run round 3 without asking.
- Round 3 still failing: stop and show the diagnosis with a card. Do not start round 4.
- The user's overrun flag from the conventions file applies throughout.

The loop ends when the combined score is 100, postwork says PASS, and the live test passed on the final commit. Every fix round changes the code, so run step 4 again after it.

## 6. Close out

Follow `references/closeout.md`. Do not start it without a PASS in `<KEY>-live-test.md` for the current commit. In order: push, PR, Jira comment, archive, delete the worktree, wait for the PR to be good, then Code Review and the worklog.

## Never

- Never let a sub-agent commit or push.
- Never edit outside the plan's files without a written reason.
- Never run a bigger model without a card.
- Never call the work done below 100.
- Never pass, push or open a PR without a live test PASS on the current commit.
- Never poll an agent or CI.
