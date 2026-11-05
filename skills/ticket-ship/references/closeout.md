# Close out

Start only when the combined score is 100, `<KEY>-live-test.md` shows PASS for the current commit with an evidence file for every step, and the `aqa` verdict is Done or Done with notes.

Each step below is on the conventions file's "Never ask" list, so none of them needs a card.

## 0. Publish the evidence

Evidence is in `.scratch/<KEY>-evidence/`. Check that every live test step has a screenshot or an API transcript. If one is missing, go back to step 4 of the skill. Never open a PR without proof.

Publish the images on an evidence branch. It shares no history with the code, so it can never be merged by accident.

1. `git -C <repo> worktree add --orphan -b evidence/<KEY> <root>/<repo>-worktrees/evidence-<KEY>`
2. In that folder: `mkdir <KEY>`, copy the images in, then `git add -A` and commit with the identity from `setup.md`.
3. `git push -u origin evidence/<KEY>`, then `git worktree remove` the folder.
4. Image links look like `https://github.com/<org>/<repo>/blob/evidence/<KEY>/<KEY>/<file>?raw=true`.

If the push is rejected, keep the images in `.scratch/<KEY>-evidence/`, paste the transcripts into the PR, and tell the user which folder holds the images to drag in. Remove secrets, tokens, emails and personal data from every image before it leaves the machine.

## 1. Push and open the PR

1. `gh auth status`. Switch to the GitHub account in `setup.md` if needed.
2. `git -C <wt> push -u origin <branch>`. Never force.
3. Write the PR body to `.scratch/<KEY>-pr.md`, then `gh pr create -R <org>/<repo> --base <default branch> --head <branch> --title "[<KEY>]: <lowercase imperative summary>" --body-file .scratch/<KEY>-pr.md`. The PR is ready for review, not a draft. One PR per repo.
4. Call `link_pull_request` with the full URL.
5. Write `pr-opened` to the timeline.

PR body headings:

```
## What this fixes
<2 to 4 plain sentences. Start with the Jira link: <jira site>/browse/<KEY>>

## What changed
<One bullet per subpart.>

## Proof it works locally
<One image or transcript per live test step, each with a one-line caption saying what it shows. Screenshots for UI changes, a screenshot of the browser inspector or response for things only visible there, a request and response transcript for APIs. For bugs, a before image next to the after image.>

## Checked
<Baseline against result. The live test table from <KEY>-live-test.md. Review scores from <KEY>-reviews.md. The aqa verdict and TL;DR from <KEY>-reviews/aqa-r<N>.md.>

## Known limits
<Risks, the Follow-ups list from the reviewers, and any bug drafts from aqa that are outside this ticket. Note that the evidence branch evidence/<KEY> can be deleted after merge.>

## Earlier work              (bug-reproduced tickets only)
<Old ticket key, old PR number, and why it came back.>
```

If the repo has a `.github/pull_request_template.md`, fill that instead and keep the Jira link, the proof section and the review scores.

## 2. Comment in Jira

Post one comment with `addCommentToJiraIssue` (markdown), right after the PR opens. Four plain lines, plus a fifth for repeat bugs:

```
Fixed: <one sentence on what was wrong and what changed>
PR: <link for each repo>
Tested: <one line that starts with the live test result>
Proof: screenshots and transcripts are in the PR description
Earlier work: <old key>, PR #<number>
```

## 3. Archive, then delete the worktree

1. Create `<root>/archived-worktrees/<KEY>/`.
2. Write `archive.md` there with: the repos, branches and PR links, the last commit SHA per repo, the commands to recreate each checkout (`git worktree add <path> <branch>`), the repro test path, the review scores, active hours and the Jira estimate.
3. Copy `<KEY>-ticket.md`, `-plan.md`, `-reviews.md`, `-timeline.md`, `-live-test.md`, `-history.md` (if it exists), `<KEY>-reviews/` and `<KEY>-evidence/` into the archive folder.
4. `git -C <repo> worktree remove <path>`. If it refuses, run `git -C <path> status --short`. Use `--force` only when the output lists nothing but ignored files such as `node_modules`.
5. Keep the branch. Never delete a remote branch.

## 4. Watch the PR without wasting tokens

Run this after the PR opens, and again on every wake:

```
python3 -I ~/.agents/skills/ticket-ship/scripts/pr-status.py <org>/<repo> <number> --state .scratch/<KEY>-pr-<repo>.json
```

It makes one call and prints about 1 KB of JSON: PR state, CI result, review decision, open threads, new comments since the last check, what to do next, and when to check again. With several repos, run it once per PR. Act on `next`.

**Cadence.** `check_again_in_minutes` is 10 while CI is running or failing or anything is open, and 60 once CI is green and nothing is open.

- In T3, call `watch_pull_request` for each PR and end the turn. It wakes the chat on CI results, reviews, comments and conflicts. It replaces timers and costs no tokens between wakes. In T3, never use `sleep`, loops, `gh --watch`, Monitor or scheduled wake-ups for a PR.
- In another agent with a scheduler, schedule the script at `check_again_in_minutes`. With no scheduler, tell the user to say "check the PR" when they want a check. Never loop.

**Token rules.**

- A wake where `next` starts with "wait" needs no chat message. End the turn.
- Do not open PR pages, full logs or comment threads unless `next` points to them.
- For a failed check, read only that job: `gh run view <id> --log-failed | tail -80`. Fix it with the `fix-ci` skill, or by hand without it.

## 5. Resolve every comment

For every item in `unresolved_threads` and `new_activity`, whether it comes from a bot or a person:

1. Read the comment and the lines around it.
2. Decide. Fix it. Or answer with a reason when the comment is wrong or out of scope. Ask the user only when the comment asks for something outside the ticket, contradicts the plan, or needs a product decision.
3. To fix: recreate the worktree from the branch if it is gone, make the change, run the baseline commands, and run the live test steps the change touches. Capture fresh proof and add it to the evidence branch. Commit `<KEY>: address review comment (<short>)` and push. If the fix changes more than 50 lines or adds files, run the review loop again. Otherwise run only the scope check.
4. Reply on the thread with the commit and what changed, or with the reason. Then resolve the thread:

   ```
   gh api graphql -f query='mutation($id:ID!){resolveReviewThread(input:{threadId:$id}){thread{isResolved}}}' -f id=<thread_id>
   ```

   Plain PR comments and review bodies get a reply with `gh pr comment`.
5. Run the script again. This step is done when `unresolved_count` is 0 and nothing new is waiting.

## 6. Code Review and worklog

The PR is good when CI passed, there are no conflicts, `unresolved_count` is 0 and no new comment is unhandled. Human approval is not required for this step.

1. Write `done` to the timeline and compute active hours.
2. Look up the issue's transitions and move it to "Code Review" by name.
3. Log work with `addWorklogToJiraIssue`:
   - `timeSpent` is the larger of active time and the Jira original estimate. Round active time up to the next 15 minutes. With no estimate, use active time.
   - `started` is the `claim` timestamp.
   - `commentBody` is 2 to 4 lines: what changed, the PR links and how it was tested.
4. Tell the user in chat: ticket key, PR links, combined score, hours logged and the archive path. Keep watching.

## 7. Keep watching until merged

Keep the watch or schedule running. Once CI is green and nothing is open, it checks every 60 minutes for approval and new comments.

- A new comment: go to step 5.
- CI fails again or a conflict appears: go to step 4 and fix it.
- Approved but not merged: wait. Never merge the PR yourself.
- Time spent on review fixes after step 6: log it as a second worklog when it adds up to more than 15 minutes.
- `state` is MERGED: call `unwatch_pull_request`, then tell the user the PR is merged.
- `state` is CLOSED without a merge: report it and ask what to do next.

## Failure notes

- If a Jira call fails, say which one failed and what is left to do by hand. Do not retry in a loop.
- If the PR cannot be opened, leave the worktree in place and tell the user why.
