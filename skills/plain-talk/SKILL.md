---
name: plain-talk
description: Explain any situation, ticket, bug, PR, skill or technical result in everyday language, like talking to a friend who does not work in tech. Gives 2-3 short lines for one thing, or 2-3 lines covering several tickets or issues together. Use when the user types plain-talk, asks to "explain simply", "in layman terms", "like I am not technical", "talk like a human", or names a skill or concept and wants a real-life example.
---

# plain-talk

Talk like a person, not a report. The reader does not work in tech. They want to know what is going on, and what happens next, in the time it takes to read three lines.

Follow the `unslop` rules for wording. This skill adds the length and shape rules below.

## What to do

1. Work out what the user is asking about. It can be a ticket, a bug, a PR, a test result, an error, a plan, a skill, or a concept. Read the real material first if you have not. Do not guess.
2. Write the explanation using the shape that fits.

## Shape

**One thing (one ticket, bug, PR or topic).** Write 2 or 3 short lines in total. Say:

- what the problem or situation is, in plain words
- where it stands now
- what happens next, or what the person needs to do

**Several things (many tickets, issues or PRs).** Still write only 2 or 3 lines, covering all of them together. Group them by what they have in common, give counts ("four of the five are done"), and call out only the one that needs attention. Do not list each item. If the user wants the per-item detail, they will ask.

No headings, no bullet lists, no tables, no diagrams. Short sentences. A reply is at most 3 lines unless the user asks for more.

## Language rules

- Use everyday words. If a technical word is unavoidable, say what it means right there, in a few words.
- Never use ticket keys, branch names, file names, status codes or acronyms alone. Say what they are about ("the login page bug"), and mention the key once at the end only if the user needs to find it.
- No code, no stack traces, no commands, unless the user asks.
- Say who does what. "We fixed it", "it is waiting on you".
- Say plainly if something is broken, late or unknown. Do not soften it and do not overstate it.
- No openers like "Sure" or "Great question", and no closing offers.

## When the user names a skill, tool or concept

If the user types the skill name plus a topic or example, for example `plain-talk ticket-ship` or `plain-talk what is a git worktree`, explain that thing with a real-life example from everyday life. Pick a situation almost anyone knows, such as a restaurant kitchen, a post office, a builder checking a house, or a library. Say what the thing does in that picture, then say in one line what it means for them. Keep to 2 or 3 lines, and do not stretch the comparison past the point where it stays true.

## Examples

One ticket:

> The checkout page was crashing when someone used a discount code. We found the cause and fixed it, and it passed our test. It is now waiting for a teammate to look it over before it goes live.

Several tickets:

> Of the six jobs this week, four are finished and two are still being built. One of the finished ones, the broken password reset, had come back twice before, so we are keeping an eye on it. Nothing needs your input right now.

Named skill with a real-life example (`plain-talk ticket-ship`):

> Think of a builder finishing a house extension. They build it, then a few inspectors each check different things, and only when everyone signs off do they hand over the keys. `ticket-ship` does that for a piece of software work: it builds the change, has several reviewers check it, tests it for real, and then hands it over for approval.
