# Plan template and handoff brief

The plan lives in `.scratch/<KEY>-plan.md`. The same text is shown in chat. Blockers go first, before everything else.

## Plan

```
# <KEY> plan: <ticket summary>

Repeat bug: no            (bug-reproduced sets this to yes and adds the two sections below)
Budget: <estimate or 4h>   Plan estimate: <hours>   Jira estimate: <hours or none>
Repos: <repo list>

## Blockers and open questions
<None, or a numbered list. Each item says who can unblock it.>

## Problem
<2 to 4 sentences in plain words.>

## Acceptance criteria
1. <each one written so a test or a command can check it>

## Reproduction (bugs only)
<Steps. Test path or script path. The failing output. Root cause, once known.>

## Previous attempt          (bug-reproduced only)
## Why it came back          (bug-reproduced only: one of "old fix incomplete", "later change broke it", "different root cause", "never really fixed")

## Approach
<What changes and why. Name the files.>

## Subparts
| Id | What | Files it owns | Depends on | Test |
|---|---|---|---|---|
No two subparts own the same file. Independent subparts run in parallel.

## Do not touch
<Files and behaviors that must stay the same. Always includes lockfiles, CI config and dependency files unless a subpart lists them.>

## Baseline (before any change)
| Repo | Test | Lint | Typecheck | Build |
|---|---|---|---|---|
<pass/fail and counts. Pre-existing failures are listed by name so they are not blamed later.>

## Test plan
<New tests, the repro test, and the commands that must pass.>

## Live test
<Mode: agent browser, agent run, or user run, and why. Setup commands and data. Numbered steps, each with the action and the expected result. Include the bug's failing action and one nearby flow that must still work. Never production.>

## Risks
<From blast-radius: what the change could break, with callers checked.>
```

## Plan check

Before showing the plan, run one Sonnet child (medium effort) that reads the `ticket-prework-review` skill and the pstack `blast-radius` skill (see Companion skills in the conventions file), then reviews `<KEY>-ticket.md` and `<KEY>-plan.md`. Tell it:

- Do not rename the chat.
- Do not ask questions.
- Do not edit any file except to write its report to `.scratch/<KEY>-reviews/plan-check.md`.
- Return at most 40 lines: a score out of 100, the gaps, and the risks.

Fix every gap that is a plan problem. List the rest under Blockers.

## Handoff brief

Show it in chat after the plan and before the handoff card. Use everyday words. Keep the real term when it is the point, with a short explanation in brackets. Five short parts:

1. **What is wrong.** What a person sees or loses.
2. **A real-life picture.** One everyday comparison.
3. **What we change.** The fix, and what it looks like in practice.
4. **What you will notice.** The visible result once it ships.
5. **What could still go wrong.** One or two honest lines.

Example for a caching bug:

> The cache (a saved copy the app keeps so it does not have to look things up again) is holding old answers. Chats are being saved, but the screen reads the stale copy, so they look empty. It is like a restaurant menu board nobody updated: the kitchen has the new dishes, but customers read the old board. The fix makes the board refresh whenever a chat is saved. After it ships, a new chat shows up straight away. The risk is that other screens also read this copy, so we check each one.

When the brief and the plan disagree, fix the plan.
