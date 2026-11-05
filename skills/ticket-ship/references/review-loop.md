# Review loop

## The four reviewers

| Reviewer | What the child reads and does (find each skill as the conventions file's Companion skills section says) |
|---|---|
| pstack interrogate | the `interrogate` skill from the pstack plugin. Adversarial review where every finding is proven by a command and its output. Review only. Do not fix. |
| postwork | the `postwork` skill. Reviews the change against `<KEY>-ticket.md`. Use the plain `postwork`, not `ticket-postwork-review`. |
| ponytail-review | the `ponytail-review` skill. Correctness, safety, load, tests, speed and lean code. |
| thermo-nuclear | the `thermo-nuclear-code-quality-review` skill. Strict maintainability review. |

## Start a round

Start the reviewers of the round together with `delegate_task`, role `review`. Round 1 runs all four. Later rounds run only the reviewers that still have open findings. When those pass, run one last pass with all four.

Target: Cursor `auto-smart` on `cost` first. A Cursor error or limit means run that reviewer again on Sonnet medium. See the conventions file.

Brief for every reviewer, with paths and no pasted text:

```
Review the change in <worktree path> against origin/dev. Ticket: <path>. Plan: <path>.
Read <the reviewer's SKILL.md path, found as above> and follow it, with these overrides:
- Review only. Do not edit, stage, commit or push anything.
- Do not rename the chat. Do not ask the user questions.
- Do not start sub-agents. Read only the diff, the plan and the files the diff touches.
- Write your findings to <.scratch/<KEY>-reviews/<name>-r<N>.md> in this exact format, one row per finding:
  F<n> | Critical/High/Medium/Low | file:line | in-scope yes/no | claim | proof (command and output, or the exact lines) | smallest fix
- A finding with no concrete failing case or proof is not a finding. Leave it out.
- "in-scope yes" means the finding is on a changed line, or on code this change breaks.
- Reply with the file path, the number of findings, and one line for the verdict. Nothing else.
For postwork also include its score out of 100 and its PASS, PASS WITH MANUAL CHECKS or FAIL gate.
```

After the reviewers return, run `git -C <wt> status --porcelain` and compare it with the state before the round. A reviewer that edited anything is a failed reviewer. Revert its edits and run it again on Sonnet.

## Combine and score

1. Read each findings file. Merge duplicates (same file and line, or same claim) into one row and keep the strongest proof.
2. Check every merged finding yourself. Read the cited lines or re-run the proof command. Mark it verified or dropped (with the reason). Dropped findings do not cost points.
3. Findings with "in-scope no" go to a Follow-ups list. They go in the PR body, not into the fix plan, and do not cost points.
4. Score starts at 100. For each verified in-scope finding subtract 25 (Critical), 10 (High), 5 (Medium) or 2 (Low). Never go below 0.
5. 100 means no verified in-scope findings and postwork says PASS. PASS WITH MANUAL CHECKS counts as not passed until the manual checks are done and recorded. The live test is a gate, not points. Without a PASS in `<KEY>-live-test.md` for the current commit, the result is NOT PASSED, whatever the score.

Show this table in chat and append it to `.scratch/<KEY>-reviews.md`:

| Reviewer | Findings | Verified in scope | Dropped | Follow-ups | Gate |
|---|---|---|---|---|---|

Then: `Combined score: <n>/100`, the postwork score and gate on one line, and `Live test: PASS | FAIL | NOT RUN (commit <sha>)`.

## QA review (after the code reviewers pass)

Start it only when the four code reviewers have no verified in-scope findings, postwork says PASS, and the live test passed on the current commit.

Start one `delegate_task` child (role `review`, Claude Sonnet medium). Do not send it to Cursor: it runs the app and creates test data. Brief, with paths and no pasted text:

```
Run the aqa skill in pipeline mode for <KEY>. Read ~/.agents/skills/aqa/SKILL.md and follow it.
Worktree: <path>. Ticket: <path>. Plan: <path>. Live test: <path>. Evidence folder: <path>.
Write the full report to <.scratch/<KEY>-reviews/aqa-r<N>.md>. Reply with the TL;DR, the verdict and the failing items in 25 lines or fewer.
```

Afterward, run the same `git status --porcelain` check as for the code reviewers.

Handle the verdict:

- **Done, or Done with notes.** The QA review passes. Notes and Low-only items go to the Follow-ups list. Add an `aqa` row to the table with the verdict as its gate.
- **Stays open / back to dev.** Check each failing item yourself, like any finding. An AC marked Fail counts as High. A Partial AC counts as Medium. A drafted bug counts by its priority: Highest 25, High 10, Medium 5, Low 2. Verified items go into the phase 2 plan. This is a failed round. After the fix, run steps 2 to 4 of the skill, re-run the code reviewers that had findings, then run the QA review again.
- **Needs BA input.** Stop. Show the ambiguous or conflicting requirement and ask the user, even in autopilot.

Bug drafts for problems outside this ticket go to the Follow-ups list and the final message. Never file them in Jira from the loop. Offer to file them, and file only after the user confirms.

## Phase 2 plan (every round below 100)

Show a short plan in chat before touching code:

| Finding | Fix | Files | Owner subpart |
|---|---|---|---|

Start a fresh builder per independent group (Sonnet medium, at most 3 at once). The brief is the finding rows plus the worktree path. Then run steps 2, 3 and 4 of the skill again (re-check the bug, baseline and scope, live test), commit the fixes as `<KEY>: <what changed>`, and start the next round.

## Round limits

- Round 1 fails: phase 2 plan, fix, round 2.
- Round 2 fails: stop fixing. Write the diagnosis, then correct the plan.
  - Do the same findings come back, or new ones each time?
  - Is the plan wrong, is the ticket unclear, or is a fix aimed at a symptom?
  - Is a reviewer demanding something outside the ticket?
  - Which single change removes the cause?

  Show the diagnosis in 6 lines or fewer, then run round 3 with the corrected plan.
- Round 3 fails: stop. Show the diagnosis and a card: "Re-plan", "Ship with follow-ups", "I'll take over". Wait for the answer.

Append each round's diagnosis and result to `<KEY>-reviews.md`.
