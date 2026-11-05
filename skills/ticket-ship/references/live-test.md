# Live test

Nothing passes without one. Unit tests, lint and builds do not count. A live test runs the changed behavior for real and checks what a user or caller would see.

## Pick the mode

1. **The agent runs it (preferred).**
   - Web UI: open the running app in a browser the agent controls and click through the steps. In T3, use the preview tools (`preview_open`, `preview_navigate`, `preview_act`, `preview_snapshot`). Load them with `ToolSearch`, and take screenshots as evidence. In other agents, use the agent's own browser tool.
   - API, worker or CLI: start the service from the worktree with the command in its README, against local or test data. Call it with `curl` or the real client. Save status codes and response bodies.
   - Docs or config change: follow the written steps literally and check the result.
2. **The user runs it.** Use this when the agent cannot run the change: it needs credentials, a real account, a device, shared or staging data, or a third-party service. Give numbered steps and the exact result to expect at each step. Wait for the answer. The user's "pass" or "fail", with their notes, is the result.

Never test against production. Shared or staging systems need the user's OK first.

## Steps

1. Start from the plan's Live test section. If it is thin, write it now: numbered steps, and for each step the action and the expected result. Always include the bug's original failing action and one nearby flow that must still work.
2. Run on the final commit. Record `git -C <wt> rev-parse HEAD`. Start services in the background. Stop them afterward and check that the ports are free.
3. Compare each step with its expected result. Take a screenshot or save the output for each step.
4. Write `.scratch/<KEY>-live-test.md`:

   ```
   Commit: <sha>   Mode: agent browser | agent run | user run   Date: <UTC>
   Setup: <commands, data used>
   | Step | Expected | Actual | Result |
   Evidence: <screenshot and log paths>
   Verdict: PASS | FAIL   Decided by: agent | user
   ```

5. The verdict is PASS only if every step matches. Any mismatch is FAIL.

A FAIL is handled like a verified Critical finding. It goes into the phase 2 plan, it counts as a failed review round, and the round limits in `review-loop.md` apply.

## User-run handoff

Post this in chat, then write `pause` to the timeline:

```
Live test for <KEY>
Where: <worktree path>   Start it with: <command>
1. <action>. Expect: <result>
2. <action>. Expect: <result>
```

Then show a card: "Passed", "Failed (I'll describe what I saw)", "I can't run it". "I can't run it" keeps the ticket blocked. Stop and ask what to do. Write `resume` after the answer.

## Staleness

A verdict is valid for one commit. Any later commit that changes code makes it stale, so run the full steps again on the new commit.
