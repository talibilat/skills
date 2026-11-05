# Cards for talib-ticket-plan

A card is one `AskUserQuestion` call. Each card holds up to 4 questions, each with 2 to 4 options. The tool adds a free-text "Other" box. Headers are 12 characters or fewer.

Only two cards are shown in a normal run. Everything else is decided by the skill. Show another card only when a real decision is missing (see the end of this file).

Build every option from the data fetched in step 1. Cut long lists to the top few. Counts come from the user's own To Do tickets.

## Card 1: what to work on

One card, four questions.

| Question | Type | Options |
|---|---|---|
| Epic | multi | The 3 epics with the most To Do tickets (key, summary, count), plus "No epic preference". Any other epic key goes in "Other". |
| Work type | multi | The types present among the tickets (Bug, Task, Story, Sub-task), with counts. |
| Priority | multi | Highest, High, Medium, "Low or Lowest", with counts. |
| Order | single | "Easiest first (Recommended)", "Hardest first". Add the measure to the label: "by story points", or "by estimate" when the project has no story points field. |

Fix versions are not asked. They only break ties in the ranking.

## Size measure

Size is story points when the project has a story points field (see the Story points section of the conventions file). With no such field, size is the original estimate in hours. State which one you used in the pick card.

## Ranking

1. Drop tickets that fail the card 1 filters.
2. Drop blocked tickets and list them as "Blocked, not offered" with the blocking key.
3. Sort by priority (Highest first).
4. Then by size, in the direction chosen in the Order question. Tickets with no size go after sized tickets of the same priority and carry the tag "no size".
5. Then by nearest fix version release date, then by Jira rank.

## Card 2: the pick

One question, single choice, because claiming a ticket assigns it to the user in Jira. Options are the top 3 ranked tickets, the first marked "(Recommended)". Label: `PROJ-123 (Bug, High, 2 points)`. Description: one line on why it ranks there and which size measure was used. "Other" takes a ticket key.

## Only when a real decision is missing

- **Nothing matches.** Show the count per filter. Ask once: widen the filters, or include unassigned tickets. Never include unassigned tickets without this answer.
- **Repos unclear.** Shown only when no repo clearly fits the ticket, or the ticket probably needs a repo that cannot be found. Multi-select, with the skill's best guess first and marked "(Recommended)".
- **Repro stuck.** Shown after 30 minutes or 3 failed approaches. Options: "Keep trying for 30 more minutes", "Move ticket back to To Do and stop", "Post a Jira comment asking for repro details".
- **Repeat bug.** Shown when step 2 finds earlier work on this ticket: "Switch to the bug-reproduced flow (Recommended)", "Treat as a new ticket".
- **Handoff.** Shown last: "Start building now (Recommended)", "Not yet, I want to change the plan", "Stop here".
