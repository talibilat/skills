# Close out

Start only when the combined score is 100 and `<KEY>-live-test.md` shows PASS for the current commit.

Do these in order. Each step is a rule from the conventions file's "Never ask" list, so none of them needs a card.

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

## Checked
<Baseline against result. For bugs, the repro before and after. The live test steps, expected and actual results, and screenshots or logs from <KEY>-live-test.md. Review scores from <KEY>-reviews.md.>

## Known limits
<Risks, and the Follow-ups list from the reviewers.>

## Earlier work              (bug-reproduced tickets only)
<Old ticket key, old PR number, and why it came back.>
```

If the repo has a `.github/pull_request_template.md`, fill that instead and keep the Jira link and review scores.

## 2. Comment in Jira

Post one comment with `addCommentToJiraIssue` (markdown), right after the PR opens. Three plain lines, plus a fourth for repeat bugs:

```
Fixed: <one sentence on what was wrong and what changed>
PR: <link for each repo>
Tested: <one line that starts with the live test result>
Earlier work: <old key>, PR #<number>
```

## 3. Archive, then delete the worktree

1. Create `<root>/archived-worktrees/<KEY>/`.
2. Write `archive.md` there with: the repos, branches and PR links, the last commit SHA per repo, the commands to recreate each checkout (`git worktree add <path> <branch>`), the repro test path, the review scores, active hours and the Jira estimate.
3. Copy `<KEY>-ticket.md`, `-plan.md`, `-reviews.md`, `-timeline.md`, `-history.md` (if it exists) and `<KEY>-reviews/` into the archive folder.
4. `git -C <repo> worktree remove <path>`. If it refuses, run `git -C <path> status --short`. Use `--force` only when the output lists nothing but ignored files such as `node_modules`.
5. Keep the branch. Never delete a remote branch.

## 4. Wait until the PR is good

Call `watch_pull_request` and end the turn. Write `pause` first. Do not poll.

On each wake, write `resume`, then:

- **Failed checks.** Recreate the worktree from the branch. Diagnose with the `pr-review-and-recovery` skill. Fix with the `fix-ci` skill. Commit, push, archive and delete again.
- **Bot and `github-actions` comments.** Fix them, or answer with a reason. Do not ask.
- **Human comments.** Summarize them for the user and ask before changing code.
- **Conflicts.** Use the `fix-merge-conflicts` skill. Re-run the baseline commands afterward.

The PR is good when all checks pass, there are no conflicts, and every bot comment is handled. Human approval is not required.

## 5. Move to Code Review and log the time

1. Write `done` to the timeline and compute active hours.
2. Look up the issue's transitions and move it to "Code Review" by name.
3. Log work with `addWorklogToJiraIssue`:
   - `timeSpent` is the larger of active time and the Jira original estimate. Round active time up to the next 15 minutes. With no estimate, use active time.
   - `started` is the `claim` timestamp.
   - `commentBody` is 2 to 4 lines: what changed, the PR links and how it was tested.
4. Call `unwatch_pull_request`.
5. Tell the user in chat: ticket key, PR links, combined score, hours logged and the archive path.

## Failure notes

- If a Jira call fails, say which one failed and what is left to do by hand. Do not retry in a loop.
- If the PR cannot be opened, leave the worktree in place and tell the user why.
